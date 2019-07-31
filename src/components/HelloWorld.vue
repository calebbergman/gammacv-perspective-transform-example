<template>
  <div id="container">
    <h3>TRANSFORMED</h3>
    <div ref="result">
      <!-- Transformed canvas injected here -->
    </div>

    <h3>ORIGINAL</h3>
    <canvas ref="canvas"></canvas>

    <img src="/doc.png" ref="image" v-show="false" />
  </div>
</template>

<script>
import * as gm from "gammacv";

export default {
  name: "HelloWorld",

  data: () => ({
    tensorTransform: null,
    vertices: {
      tr: { x: 1846, y: 211 },
      tl: { x: 234, y: 213 },
      bl: { x: 225, y: 2755 },
      br: { x: 1835, y: 2761 }
    }
  }),

  methods: {
    doTransform() {
      const documentCanvas = this.transform(
        this.tensorTransform,
        this.vertices
      );
      this.$refs.result.appendChild(documentCanvas);
    },
    transform(input, { tl, tr, br, bl }) {
      // The Rect TL, TR, BR, BL
      const rect = new gm.Rect(tl.x, tl.y, tr.x, tr.y, br.x, br.y, bl.x, bl.y);

      const tTransform = new gm.Tensor("float32", [3, 1, 4]);

      let heightRight = Math.hypot(tr.x - br.x, tr.y - br.y);
      let heightLeft = Math.hypot(tl.x - bl.x, tr.y - bl.y);
      let maxHeight = Math.floor(Math.max(heightRight, heightLeft));

      // Assume standard document scale of 8.5 x 11.
      let maxWidth = Math.floor(maxHeight * (8.5 / 11));

      gm.generateTransformMatrix(rect, [maxHeight, maxWidth], tTransform);

      const operation = gm.perspectiveProjection(input, tTransform, [
        maxHeight,
        maxWidth,
        4
      ]);
      const sess = new gm.Session();
      const output = gm.tensorFrom(operation);

      sess.init(operation);
      sess.runOp(operation, 0, output);

      // show output
      const canvas = document.createElement("canvas");
      canvas.width = output.shape[1];
      canvas.height = output.shape[0];
      gm.canvasFromTensor(canvas, output);

      return canvas;
    }
  },

  mounted() {
    const { canvas, image } = this.$refs;

    image.onload = async () => {
      const context = canvas.getContext("2d");
      const height = image.naturalHeight;
      const width = image.naturalWidth;

      canvas.height = height;
      canvas.width = width;

      context.drawImage(image, 0, 0, width, height);

      const tensorTransform = new gm.Tensor("uint8", [height, width, 4]);
      await gm.canvasToTensor(canvas, tensorTransform);
      this.tensorTransform = tensorTransform;

      this.doTransform();
    };
  }
};
</script>

<style>
canvas {
  max-width: 100%;
}
</style>
