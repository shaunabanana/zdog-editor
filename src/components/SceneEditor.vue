<template>
  <canvas
    ref="canvas"
    id="zdog-canvas"
    :style="{ background: background }"
  ></canvas>
</template>

<script>
import Zdog from "zdog";

export default {
  name: "SceneEditor",
  props: {
    scene: Array,
    background: {
      type: String,
      default: "#FDB",
    },
  },

  data() {
    return {
      resizeObserver: null,
      width: 0,
      height: 0,
      illustration: null,
    };
  },

  methods: {
    construct(element, parent) {
      let props = JSON.parse(JSON.stringify(element.props));
      props.addTo = parent;
      let shape = new Zdog[element.type](props);
      console.log({
        id: element.id,
        shape: shape,
      });
      this.$emit("new-shape", {
        id: element.id,
        shape: shape,
      });
      for (let child of element.children) {
        this.construct(child, shape);
      }
    },

    setup() {
      this.$emit("setup");
      this.illustration = new Zdog.Illustration({
        element: this.$refs.canvas,
        dragRotate: true,
        resize: true,
      });

      let origin = new Zdog.Shape({
        addTo: this.illustration,
        color: "#636",
        stroke: 3,
      });

      new Zdog.Polygon({
        addTo: origin,
        radius: 5,
        sides: 3,
        translate: { x: 100 },
        rotate: {z: -Math.PI / 6},
        stroke: 5,
        fill: true,
        color: "#C25",
      });

      new Zdog.Shape({
        addTo: origin,
        path: [{}, { x: 100 }],
        color: "#C25",
        stroke: 3,
      });

      new Zdog.Polygon({
        addTo: origin,
        radius: 5,
        sides: 3,
        translate: { y: -100 },
        stroke: 5,
        fill: true,
        color: "#E62",
      });

      new Zdog.Shape({
        addTo: origin,
        path: [{}, { y: -100 }],
        color: "#E62",
        stroke: 3,
      });

      new Zdog.Polygon({
        addTo: origin,
        radius: 5,
        sides: 3,
        translate: { z: 100 },
        rotate: {z: -Math.PI / 6, y: Math.PI / 2},
        stroke: 5,
        fill: true,
        color: "#636",
      });

      new Zdog.Shape({
        addTo: origin,
        path: [{}, { z: 100 }],
        color: "#636",
        stroke: 3,
      });

      for (let element of this.scene) {
        this.construct(element, this.illustration);
      }

      // update & render
      this.illustration.updateRenderGraph();
    },

    render() {
      this.illustration.updateRenderGraph();
      requestAnimationFrame(this.render.bind(this));
    },
  },

  watch: {
    scene: {
      deep: true,
      handler() {
        let rotate = this.illustration.rotate;
        this.setup();
        this.illustration.rotate = rotate;
      },
    },
  },

  mounted() {
    this.setup();
    this.render();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
canvas {
  width: 100%;
  height: 100%;
  cursor: move;
  z-index: 0;
}
</style>
