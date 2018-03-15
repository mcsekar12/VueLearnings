<template>
<div class="flow" :style="{'height':autoHeight+'px'}">
  <div class="action__cont">


      <button @click="addEnvironment">
        <img src="../assets/images/plus-button.svg" alt="icon">
        <img src="../assets/images/cloud.svg" alt="icon">
        Add Environment
        </button>


   <button @click="addCluster">
      <img src="../assets/images/plus-button.svg" alt="icon">
        <img src="../assets/images/cloud.svg" alt="icon">Add Cluster</button>
    <button @click="addVm">
       <img src="../assets/images/plus-button.svg" alt="icon">
        <img src="../assets/images/cloud.svg" alt="icon">Add VM</button>
  </div>


        <div class="jtk-demo-main">

  <div class="jtk-demo-canvas canvas-wide statemachine-demo jtk-surface jtk-surface-nopan"  id="canvas">



                <div class="w" :class="node.type" :id="node.type+'#'+node.id" v-for="(node,envIndex) in envList" :key="'env'+envIndex" :data-item="node.type"
                v-bind:style="{  top: 100*envIndex + 'px' }" >
                <img data-v-36dfb84c="" src="../assets/images/cloud.svg" alt="icon">
                 <input type="text" v-model="node.environmentName">

                    <div class="ep" action="index" :class="node.type"></div>
                </div>

                <div class="w" :class="node.type" :id="node.type+'#'+node.id" v-for="(node,clusterIndex) in clusterList" :key="'cluster'+clusterIndex" :data-item="node.type"
                v-bind:style="{  top: 100*clusterIndex + 'px' }">
                <img data-v-36dfb84c="" src="../assets/images/environment.svg" alt="icon">

                 <input type="text" v-model="node.clusterName">

                    <div class="ep" action="clusterIndex" :class="node.type"></div>
                </div>

                <div class="w" :class="node.type" :id="node.type+'#'+node.id" v-for="(node,vmIndex) in vmList" :key="'vm'+vmIndex" :data-item="node.type"
                v-bind:style="{  top: 100*vmIndex + 'px' }">
                <img data-v-36dfb84c="" src="../assets/images/server.svg" alt="icon">

                <input type="text" v-model="node.serverName">

                    <div class="ep" action="vmIndex" :class="node.type"></div>
                </div>

            </div>




        </div>

           <div class="json__view">
 <vue-json-pretty
      :path="'res'"
      :data="envList"
      >
    </vue-json-pretty>
           </div>


</div>
</template>

<script>
import jsplumb from "../assets/jsplumb.js";
import VueJsonPretty from "vue-json-pretty";

export default {
  name: "FlowChart",
  props: {
    applicationId: {
      type: [String, Number]
    },
    environment: {
      type: Object
    }
  },
  components: {
    VueJsonPretty
  },
  data() {
    return {
      instance: null,
      envList: [],
      clusterList: [],
      vmList: [],
      connectList: [],
      loading: false
    };
  },
  computed: {
    env() {
      return this.environment ? [this.environment] : [];
    },
    autoHeight() {
      let max = this.envList.length;
      if (this.clusterList.length > max) {
        max = this.clusterList.length;
      }
      if (this.vmList.length > max) {
        max = this.vmList.length;
      }
      return 100 + max * 100 || 150;
    }
  },
  methods: {
    addEnvironment() {
      let len = this.envList.length;
      this.envList.push({
        id: jsPlumbUtil.uuid(),
        type: "env",
        environmentName: "env" + len,
        clusterDetails: []
      });
      // make all the window divs draggable
    },
    addCluster() {
      let len = this.clusterList.length;
      this.clusterList.push({
        id: jsPlumbUtil.uuid(),
        type: "cluster",
        clusterName: "cluster" + len,
        serverDetails: []
      });

      // make all the window divs draggable
    },
    addVm() {
      let len = this.vmList.length;
      this.vmList.push({
        id: jsPlumbUtil.uuid(),
        type: "vm",
        serverName: "vm" + len
      });

      // make all the window divs draggable
    },
    initNode(el, type) {
      // initialise draggable elements.
      this.instance.draggable(el);
      let maxConnections = -1;

      if (type == "env") {
        this.instance.makeSource(el, {
          filter: ".ep",

          anchor: "Continuous",
          connectorStyle: {
            stroke: "#f8b686",
            strokeWidth: 2,
            outlineStroke: "transparent",
            outlineWidth: 4
          },

          connectionType: "basic",
          extract: {
            action: "the-action"
          },
          maxConnections: 3,
          onMaxConnections: function(info, e) {
            alert("Maximum connections (" + info.maxConnections + ") reached");
          }
        });
      } else if (type == "cluster") {
        this.instance.makeSource(el, {
          filter: ".ep",
          anchor: "Continuous",
          connectorStyle: {
            stroke: "#f8b686",
            strokeWidth: 2,
            outlineStroke: "transparent",
            outlineWidth: 4
          },

          connectionType: "basic",
          extract: {
            action: "the-action"
          },
          maxConnections: -1,
          onMaxConnections: function(info, e) {
            alert("Maximum connections (" + info.maxConnections + ") reached");
          }
        });

        this.instance.makeTarget(el, {
          dropOptions: { hoverClass: "dragHover" },
          anchor: "Continuous",
          deleteEndpointsOnDetach: false,
          maxConnections: 1
        });
      } else {
        this.instance.makeTarget(el, {
          dropOptions: { hoverClass: "dragHover" },
          maxConnections: 1
        });
      }

      // this is not part of the core demo functionality; it is a means for the Toolkit edition's wrapped
      // version of this demo to find out about new nodes being added.
      //
      this.instance.fire("jsPlumbDemoNodeAdded", el);
    },
    saveEnv() {
      this.clusterList.forEach(cl => {
        cl.serverDetails = [];
        cl.server.forEach(item => {
          let { serverName } = this.vmList[this.findIndex(this.vmList, item)];
          cl.serverDetails.push({
            serverName
          });
        });
        delete cl.server;
      });
      this.envList.forEach(env => {
        env.clusterDetails = [];
        env.cluster.forEach(item => {
          let { clusterName, serverDetails } = this.clusterList[
            this.findIndex(this.clusterList, item)
          ];

          env.clusterDetails.push({
            clusterName,
            serverDetails
          });
        });
        delete env.cluster;
        delete env.id;
        delete env.type;
      });
    },
    findIndex(arr, id) {
      let idIndex = -1;
      arr.forEach((cl, index) => {
        if (cl.id == id) {
          idIndex = index;
        }
      });
      return idIndex;
    },
    cancel() {
      let envId = this.environment.id;
      this.$emit("cancel", envId);
    }
  },
  created() {
    console.log("created");
    this.env.forEach((item, envIndex) => {
      this.envList.push({
        id: item.id,
        environmentName: item.environmentName,
        type: "env"
      });

      item.clusterDetails.forEach((cl, clusterIndex) => {
        this.clusterList.push({
          clusterName: cl.clusterName,
          id: cl.id,
          type: "cluster"
        });

        this.connectList.push({
          source: "env#" + item.id,
          target: "cluster#" + cl.id,
          type: "basic"
        });
        cl.serverDetails.forEach((svr, serverIndex) => {
          this.vmList.push({
            id: svr.id,
            serverName: svr.serverName,
            type: "vm"
          });

          this.connectList.push({
            source: "cluster#" + cl.id,
            target: "vm#" + svr.id,
            type: "basic"
          });
        });
      });
    });
  },
  updated() {
    console.log("updated");
    this.instance.batch(() => {
      let nodes = this.$el.querySelectorAll(
        ".statemachine-demo .w:not(.jtk-draggable)"
      );

      nodes.forEach(el => {
        this.initNode(el, el.dataset.item);
      });

      jsPlumb.fire("jsPlumbDemoLoaded", this.instance);
    });
  },
  mounted() {
    this.instance = jsPlumb.getInstance({
      Endpoint: [
        "Dot",
        { radius: 1, cssClass: "connector__circle", fill: "#f8b686" }
      ],
      Connector: "StateMachine",
      HoverPaintStyle: { stroke: "#1e8151", strokeWidth: 2 },
      ConnectionOverlays: [
        [
          "Arrow",
          {
            location: 1,
            id: "arrow",
            length: 14,
            foldback: 0.8
          }
        ],
        ["Label", { label: "", id: "label", cssClass: "aLabel" }]
      ],
      Container: "canvas"
    });

    this.instance.registerConnectionType("basic", {
      anchor: "Continuous",
      connector: "StateMachine"
    });
    this.instance.batch(() => {
      let nodes = this.$el.querySelectorAll(".statemachine-demo .w");

      nodes.forEach(el => {
        this.initNode(el, el.dataset.item);
      });
      jsPlumb.fire("jsPlumbDemoLoaded", this.instance);
    });

    jsPlumb.fire("jsPlumbDemoLoaded", this.instance);

    this.instance.bind("beforeDrop", connection => {
      let source = connection.sourceId.split("#")[0];
      let target = connection.targetId.split("#")[0];
      return source !== target || (source == "cluster" && taget !== "env");
    });

    this.instance.bind("connection", info => {
      let source = info.source.id.split("#");
      let target = info.target.id.split("#");

      let sourceType = source[0];
      let targetType = target[0];

      let sourceIndex = source[1];
      let targetIndex = target[1];

      if (sourceType == "env") {
        sourceIndex = this.findIndex(this.envList, sourceIndex);
        this.envList[sourceIndex].clusterDetails.push(
          this.clusterList[this.findIndex(this.clusterList, targetIndex)]
        );
      } else if (sourceType == "cluster") {
        sourceIndex = this.findIndex(this.clusterList, sourceIndex);
        let serverDetails = this.vmList[
          this.findIndex(this.vmList, targetIndex)
        ];
        this.clusterList[sourceIndex].serverDetails.push(serverDetails);
      }
    });

    this.connectList.forEach(conn => {
      this.instance.connect(conn);
    });

    this.instance.bind("click", c => {
      this.instance.deleteConnection(c);
    });
  }
};
</script>

<style lang="scss"  scoped>
// Colors
$bg-color: #fff;
$dot-color: lightgrey;

// Dimensions
$dot-size: 2px;
$dot-space: 22px;
.demo {
  /* for IE10+ touch devices */
  touch-action: none;
}
#canvas {
  position: relative;
  width: 100%;
  height: 100%;
}
.jtk-demo-main {
  margin-top: 20px;
  // min-height: 200px;
  height: 100%;
  width: 100%;
  position: relative;
}
.flow {
  // height: 100%;
  // min-height: 300px;
  width: 100%;
  z-index: 1;
  padding: 20px;
  border-radius: 4px;
  background-color: #ffffff;
  box-shadow: 2px 2px 8px 0 rgba(0, 0, 0, 0.14);
  border: solid 1px #e0e0e0;
  margin-top: 20px;

  background: linear-gradient(
        90deg,
        $bg-color ($dot-space - $dot-size),
        transparent 1%
      )
      center,
    linear-gradient($bg-color ($dot-space - $dot-size), transparent 1%) center,
    $dot-color;
  background-size: $dot-space $dot-space;
  position: relative;
  x: 100px;
}
.statemachine-demo {
  position: relative;
  height: 100%;
}

.w {
  left: 10px;
  top: 20px;
  position: absolute;
  min-width: 150px;
  z-index: 4;
  width: 100%;
  max-width: 250px;
  opacity: 0.8;
  cursor: move;
  -webkit-transition: background-color 0.25s ease-in;
  -moz-transition: background-color 0.25s ease-in;
  transition: background-color 0.25s ease-in;
  padding: 14px 25px;
  border-radius: 2px;
  background-color: #f3f3f3;
  border: solid 1px #e0e0e0;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 20px;
  margin-right: 20px;
  input {
    border: none;
    outline: none;
    font-size: 12px;
    width: 100%;
    background-color: transparent;
  }
  img {
    width: 22px;
    height: 22px;
    margin-right: 8px;
  }
}

.w.env {
  top: 120px;
  left: 10px;
}
.w.cluster {
  top: 120px;
  left: 350px;
}
.w.vm {
  top: 120px;
  left: 750px;
}
.ep.vm {
  display: none;
}

.aLabel.jtk-hover,
.jtk-source-hover,
.jtk-target-hover {
  background-color: #1e8151;
  color: white;
}

.ep {
  position: absolute;
  right: -5px;
  width: 8px;
  border-radius: 50%;
  height: 8px;
  background-color: #f8b686;
  cursor: pointer;
}

.ep:hover {
  box-shadow: 0 0 6px black;
}

.statemachine-demo .jtk-endpoint {
  z-index: 3;
}

.dragHover {
  border: 2px solid orange;
}

path,
.jtk-endpoint {
  cursor: pointer;
}

.action__cont {
  display: flex;

  button {
    border-radius: 25px;
    background-color: #ffffff;
    border: solid 1px #e0e0e0;
    padding: 8px;
    display: flex;
    font-family: Ubuntu;
    font-size: 14px;
    text-align: left;
    color: #393939;
    margin-right: 20px;
    cursor: pointer;
    outline: none;
    img {
      width: 16px;
      height: 16px;
      margin-right: 5px;
    }
  }
}
.footer__cont {
  position: absolute;
  bottom: 20px;
  right: 30px;
  button {
    font-family: Ubuntu;
    font-size: 12px;
    font-weight: 500;
    text-align: center;
    color: #38454f;
    border-radius: 4px;
    background-color: #ffffff;
    border: solid 2px #e0e0e0;
    padding: 6px 15px;
    margin-right: 10px;
    cursor: pointer;
    &.save {
      background: #38454f;
      color: white;
      border: solid 2px #38454f;
    }
  }
}
.json__view {
  text-align: left;
  margin: 20px;
}
</style>
