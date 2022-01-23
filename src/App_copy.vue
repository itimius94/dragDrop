<template>
  <div id="app">
    <div class="content_wrap">
      <div class="content_left">
        <div
          @drag="drag($event, 'TestElement')"
          @dragend="dragend($event, 'TestElement')"
          class="droppable-element"
          draggable="true"
          unselectable="on"
        >
          Droppable Element (Drag me!)
        </div>
        <div
          @drag="drag($event, 'TestElement2')"
          @dragend="dragend($event, 'TestElement2')"
          class="droppable-element"
          draggable="true"
          unselectable="on"
        >
          <p>test</p>

          <img
            src="https://images.fpt.shop/unsafe/filters:quality(90)/fptshop.com.vn/uploads/images/tin-tuc/82409/Originals/select-default-operating-system-with-boot-camp-on-mac.png"
            width="50"
            height="50"
          />
          <div class="overlap"></div>
        </div>
      </div>
      <div class="content_right">
        <div id="content">
          <!-- Add to show rtl support -->
          <input type="checkbox" v-model="draggable" /> Draggable
          <input type="checkbox" v-model="resizable" /> Resizable
          <input type="checkbox" v-model="mirrored" /> Mirrored
          <input type="checkbox" v-model="responsive" /> Responsive
          <input type="checkbox" v-model="preventCollision" /> Prevent Collision
          <div style="margin-top: 10px; margin-bottom: 10px">
            Row Height: <input type="number" v-model="rowHeight" /> Col nums:
            <input type="number" v-model="colNum" /> Margin x:
            <input type="number" v-model="marginX" /> Margin y:
            <input type="number" v-model="marginY" />
          </div>

          <grid-layout
            ref="gridlayout"
            :margin="[parseInt(marginX), parseInt(marginY)]"
            :layout.sync="layout"
            :col-num="parseInt(colNum)"
            :row-height="rowHeight"
            :is-draggable="draggable"
            :is-resizable="resizable"
            :is-mirrored="mirrored"
            :prevent-collision="preventCollision"
            :vertical-compact="compact"
            :use-css-transforms="true"
            :responsive="responsive"
          >
            <grid-item
              v-for="item in layout"
              :key="item.i"
              :x="item.x"
              :y="item.y"
              :w="item.w"
              :h="item.h"
              :i="item.i"
              :min-w="item.minW"
              :max-w="item.maxW"
              :max-h="item.maxH"
              :min-x="item.minX"
              :max-x="item.maxX"
              :min-y="item.minY"
              :max-y="item.maxY"
              :static="item.static"
              :preserve-aspect-ratio="item.preserveAspectRatio"
              @resize="resize"
              @removeItem="removeItem($event)"
            >
              <component
                v-bind:is="item.componentName || 'TestElement'"
                :text="item.i"
                @removeItem="removeItem($event)"
              ></component>
            </grid-item>

            <div class="grid_wrap">
              <div class="grid_inner">
                <div class="test" v-for="item in 40" :key="item"></div>
              </div>
            </div>
          </grid-layout>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import GridItem from "./customs/components/GridItem.vue";
import GridLayout from "./customs/components/GridLayout.vue";
import TestElement from "./customs/components/TestElement.vue";
import TestElement2 from "./customs/components/TestElement2.vue";

let mouseXY = { x: null, y: null };
let DragPos = { x: null, y: null, w: 1, h: 1, i: null };

let testLayout = [
  {
    x: 0,
    y: 0,
    w: 1,
    h: 1,
    i: "0",
    resizable: true,
    draggable: true,
    static: false,
    maxH: 1,
    componentName: "TestElement2",
  },
  {
    x: 2,
    y: 0,
    w: 1,
    h: 1,
    i: "2",
    resizable: false,
    draggable: false,
    static: false,
    maxH: 1,
    maxW: 1,
  },
  {
    x: 0,
    y: 1,
    w: 2,
    h: 1,
    i: "3",
    resizable: false,
    draggable: false,
    static: false,
  },
  {
    x: 2,
    y: 2,
    w: 2,
    h: 1,
    i: "4",
    resizable: false,
    draggable: false,
    static: false,
  },
];

export default {
  name: "app",
  components: {
    GridLayout,
    GridItem,
    TestElement,
    TestElement2,
  },
  data() {
    return {
      layout: JSON.parse(JSON.stringify(testLayout)),
      draggable: true,
      resizable: true,
      mirrored: false,
      responsive: false,
      preventCollision: false,
      compact: true,
      rowHeight: 155,
      colNum: 4,
      index: 0,
      marginX: 10,
      marginY: 10,
    };
  },
  mounted: function () {
    this.index = this.layout.length;
    document.addEventListener(
      "dragover",
      function (e) {
        mouseXY.x = e.clientX;
        mouseXY.y = e.clientY;
      },
      false
    );
  },
  methods: {
    removeItem: function (i) {
      console.log("### REMOVE " + i);
      const index = this.layout.map((item) => item.i).indexOf(i);
      this.layout.splice(index, 1);
    },
    drag: function () {
      let parentRect = document
        .getElementById("content")
        .getBoundingClientRect();
      let mouseInGrid = false;
      if (
        mouseXY.x > parentRect.left &&
        mouseXY.x < parentRect.right &&
        mouseXY.y > parentRect.top &&
        mouseXY.y < parentRect.bottom
      ) {
        mouseInGrid = true;
      }
      if (
        mouseInGrid === true &&
        this.layout.findIndex((item) => item.i === "drop") === -1
      ) {
        this.layout.push({
          x: (this.layout.length * 2) % (this.colNum || 12),
          y: this.layout.length + (this.colNum || 12), // puts it at the bottom
          w: 1,
          h: 1,
          i: "drop",
        });
      }
      let index = this.layout.findIndex((item) => item.i === "drop");
      if (index !== -1) {
        try {
          this.$refs.gridlayout.$children[
            this.layout.length
          ].$refs.item.style.display = "none";
        } catch (error) {
          console.log("");
        }
        let el = this.$refs.gridlayout.$children[index];
        el.dragging = {
          top: mouseXY.y - parentRect.top,
          left: mouseXY.x - parentRect.left,
        };
        let new_pos = el.calcXY(
          mouseXY.y - parentRect.top,
          mouseXY.x - parentRect.left
        );
        if (mouseInGrid === true) {
          this.$refs.gridlayout.dragEvent(
            "dragstart",
            "drop",
            new_pos.x,
            new_pos.y,
            1,
            1
          );
          DragPos.i = String(index);
          DragPos.x = this.layout[index].x;
          DragPos.y = this.layout[index].y;
        }
        if (mouseInGrid === false) {
          this.$refs.gridlayout.dragEvent(
            "dragend",
            "drop",
            new_pos.x,
            new_pos.y,
            1,
            1
          );
          this.layout = this.layout.filter((obj) => obj.i !== "drop");
        }
      }
    },
    dragend: function (_, componentName) {
      let parentRect = document
        .getElementById("content")
        .getBoundingClientRect();
      let mouseInGrid = false;
      if (
        mouseXY.x > parentRect.left &&
        mouseXY.x < parentRect.right &&
        mouseXY.y > parentRect.top &&
        mouseXY.y < parentRect.bottom
      ) {
        mouseInGrid = true;
      }
      if (mouseInGrid === true) {
        // alert(`ID ${id} Dropped element props:\n${JSON.stringify(DragPos, ['x', 'y', 'w', 'h'], 2)}`);
        this.$refs.gridlayout.dragEvent(
          "dragend",
          "drop",
          DragPos.x,
          DragPos.y,
          1,
          1
        );
        this.layout = this.layout.filter((obj) => obj.i !== "drop");
        // UNCOMMENT below if you want to add a grid-item
        this.layout.push({
          x: DragPos.x,
          y: DragPos.y,
          w: 1,
          h: 1,
          i: DragPos.i,
          componentName: componentName,
        });
        this.$refs.gridLayout.dragEvent(
          "dragend",
          DragPos.i,
          DragPos.x,
          DragPos.y,
          1,
          1
        );
        try {
          this.$refs.gridLayout.$children[
            this.layout.length
          ].$refs.item.style.display = "block";
        } catch (error) {
          console.log(error);
        }
      }
    },
    resize: function (i, newH, newW, newHPx, newWPx) {
      console.log(
        "RESIZE i=" +
          i +
          ", H=" +
          newH +
          ", W=" +
          newW +
          ", H(px)=" +
          newHPx +
          ", W(px)=" +
          newWPx
      );
    },
  },
};
</script>


<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

.grid_wrap {
  padding: 10px;
  position: absolute;
  width: calc(100% - 20px);
}

.grid_inner {
  display: grid;
  grid-template-columns: auto auto auto auto;
  grid-gap: 10px;
}

.test {
  border: 1px dashed;
  height: 155px;
}

.overlap {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  z-index: 1;
}

.vue-grid-item {
  z-index: 2;
}

.droppable-element {
  width: 150px;
  text-align: center;
  background: #fdd;
  border: 1px solid black;
  margin: 10px 0;
  padding: 10px;
  position: relative;
}
.vue-grid-layout {
  background: #eee;
}
// .vue-grid-item:not(.vue-grid-placeholder) {
//   background: #ccc;
//   border: 1px solid black;
// }
// .vue-grid-item .resizing {
//   opacity: 0.9;
// }
// .vue-grid-item .static {
//   background: #cce;
// }
// .vue-grid-item .text {
//   font-size: 24px;
//   text-align: center;
//   position: absolute;
//   top: 0;
//   bottom: 0;
//   left: 0;
//   right: 0;
//   margin: auto;
//   height: 100%;
//   width: 100%;
// }
// .vue-grid-item .no-drag {
//   height: 100%;
//   width: 100%;
// }
// .vue-grid-item .minMax {
//   font-size: 12px;
// }
// .vue-grid-item .add {
//   cursor: pointer;
// }

.layoutJSON {
  background: #ddd;
  border: 1px solid black;
  margin-top: 10px;
  padding: 10px;
}
.layoutJSON {
  background: #ddd;
  border: 1px solid black;
  margin-top: 10px;
  padding: 10px;
}
.columns {
  -moz-columns: 120px;
  -webkit-columns: 120px;
  columns: 120px;
}

.content_wrap {
  display: flex;
}

.content_flex {
  flex: 0 0 300px;
}

.content_right {
  flex: 0 0 1;
}
</style>
