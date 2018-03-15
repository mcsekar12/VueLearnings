<template>
<div>
  Home
  <div class="js-container1" id="js-container1">
    <div id="jsplumb1" class="cont"></div>
   <div id="jsplumb2" class="cont"></div>
  </div>
  
   <div class="jtk-demo-main">
			<!-- demo -->
            <div class="jtk-demo-canvas canvas-wide source-target-demo jtk-surface jtk-surface-nopan" id="canvas">
                <div class="window" id="sourceWindow1">
                    <strong>Window 1</strong>
                    <a href="#" id="enableDisableSource">disable</a>
                </div>
                <div class="window smallWindow" id="targetWindow2"><strong>Window 2</strong><a href="#" class="enableDisableTarget">disable</a><br/><br/></div>
                <div class="window smallWindow" id="targetWindow3"><strong>Window 3</strong><a href="#" class="enableDisableTarget">disable</a><br/><br/></div>
                <div class="window smallWindow" id="targetWindow4"><strong>Window 4</strong><a href="#" class="enableDisableTarget">disable</a><br/><br/></div>
                <div class="window smallWindow" id="targetWindow5"><strong>Window 5</strong><a href="#" class="enableDisableTarget">disable</a><br/><br/></div>
                <div class="window smallWindow" id="targetWindow6"><strong>Window 6</strong><a href="#" class="enableDisableTarget">disable</a><br/><br/></div>
            </div>
			<!-- /demo -->
   
        </div>
</div>
</template>

<script>
import jsplumb from "../assets/jsplumb.js";
export default {
  name: "HelloWorld",
  data() {
    return {
      firstInstance: null,
      instance: null
    };
  },
  props: {
    msg: String
  },
  mounted() {
    this.firstInstance = jsPlumb.getInstance({
      // drag options
      DragOptions: { cursor: "pointer", zIndex: 2000 },
      // default to a gradient stroke from blue to green.
      PaintStyle: {
        gradient: {
          stops: [[0, "#0d78bc"], [1, "#558822"]]
        },
        stroke: "#558822",
        strokeWidth: 10
      },
      Container: "js-container1"
    });

    this.firstInstance.connect({
      source: "jsplumb1",
      target: "jsplumb2",
      scope: "someScope"
    });
    this.firstInstance.draggable("jsplumb1");
    this.firstInstance.draggable("jsplumb2");

    //second

    // list of possible anchor locations for the blue source element
    var sourceAnchors = [
      [0, 1, 0, 1],
      [0.25, 1, 0, 1],
      [0.5, 1, 0, 1],
      [0.75, 1, 0, 1],
      [1, 1, 0, 1]
    ];

    this.instance = jsPlumb.getInstance({
      // drag options
      DragOptions: { cursor: "pointer", zIndex: 2000 },
      // default to a gradient stroke from blue to green.
      PaintStyle: {
        gradient: {
          stops: [[0, "#0d78bc"], [1, "#558822"]]
        },
        stroke: "#558822",
        strokeWidth: 10
      },
      Container: "canvas"
    });

    // bind to a connection event, just for the purposes of pointing out that it can be done.
    this.instance.bind("connection", (i, c) => {
      if (typeof console !== "undefined")
        console.log("connection", i.connection);
    });

    // get the list of ".smallWindow" elements.
    var smallWindows = jsPlumb.getSelector(".smallWindow");
    // make them draggable
    this.instance.draggable(smallWindows, {
      filter: ".enableDisableTarget"
    });

    // suspend drawing and initialise.
    this.instance.batch(() => {
      // make 'window1' a connection source. notice the filter and filterExclude parameters: they tell jsPlumb to ignore drags
      // that started on the 'enable/disable' link on the blue window.
      this.instance.makeSource("sourceWindow1", {
        filter: "a",
        filterExclude: true,
        maxConnections: -1,
        endpoint: ["Dot", { radius: 7, cssClass: "small-blue" }],
        anchor: sourceAnchors
      });

      // configure the .smallWindows as targets.
      this.instance.makeTarget(smallWindows, {
        dropOptions: { hoverClass: "hover" },
        anchor: "Top",
        endpoint: ["Dot", { radius: 11, cssClass: "large-green" }]
      });

      // and finally connect a couple of small windows, just so its obvious what's going on when this demo loads.
      this.instance.connect({
        source: "sourceWindow1",
        target: "targetWindow5"
      });
      this.instance.connect({
        source: "sourceWindow1",
        target: "targetWindow2"
      });
    });

    jsPlumb.fire("jsPlumbDemoLoaded", this.instance);
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.js-container1 {
  width: 100%;
  height: 400px;
  background: lightgrey;
  position: relative;
  display: flex;
}
.cont {
  width: 100px;
  height: 100px;
  margin: 50px;
  background: lightcoral;
}
#jsplumb2 {
  background: lightcyan;
}

.demo {
  /* for IE10+ touch devices */
  touch-action: none;
}

/** ELEMENTS **/
.window {
  position: absolute;
}

.window:hover {
  box-shadow: 2px 2px 19px #444;
  -o-box-shadow: 2px 2px 19px #444;
  -webkit-box-shadow: 2px 2px 19px #444;
  -moz-box-shadow: 2px 2px 19px #fff;
}

.source-target-demo .window {
  border: 2px dotted #346789;
  width: 14em;
  height: 4em;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0.5em;
}

.smallWindow {
  background-color: #558822;
  cursor: move;
}

/** ENABLE/DISABLE LINK **/
#enableDisableSource {
  position: absolute;
  top: 1em;
  right: 5px;
  color: #ddd;
  font-size: 12px;
}

#enableDisableSource:hover {
  color: orange;
}

.enableDisableTarget {
  color: white;
  right: 5px;
  top: 5px;
  position: absolute;
  font-size: 12px;
}

.enableDisableTarget:visited,
.enableDisableTarget:focus {
  color: white;
}

.enableDisableTarget:hover {
  color: orange;
}

.element-disabled {
  outline: 2px solid lightcoral;
}

/** LARGE WINDOW STYLE AND POSITION **/
#sourceWindow1 {
  background-color: #0d78bc;
  top: 9em;
  left: 14em;
  width: 30em;
  height: 6em;
  text-align: left;
}

/** SMALL WINDOW POSITIONS **/
#targetWindow2 {
  top: 29em;
  left: 44em;
}

#targetWindow3 {
  top: 23em;
  left: 2em;
}

#targetWindow4 {
  top: 31em;
  left: 26em;
}

#targetWindow5 {
  top: 39em;
  left: 13em;
}

#targetWindow6 {
  top: 21em;
  left: 56em;
}

/** JSPLUMB ARTEFACTS **/
.jtk-endpoint {
  z-index: 32;
}

path,
.jtk-endpoint {
  cursor: pointer;
}

/* ENDPOINT STYLES */
.large-green circle {
  fill: #558822;
}

.small-blue circle {
  fill: #0d78bc;
}
</style>
