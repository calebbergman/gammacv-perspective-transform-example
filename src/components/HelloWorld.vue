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
import { Plugins, FilesystemDirectory, FilesystemEncoding } from '@capacitor/core';
const { Filesystem } = Plugins;

export default {
  name: "HelloWorld",

  data: () => ({
    tensorTransform: null,
    vertices: {
      tl: { x: 234, y: 213 },
      tr: { x: 1846, y: 211 },
      br: { x: 1835, y: 2761 },
      bl: { x: 225, y: 2755 }
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
    async transform(input, { tl, tr, br, bl }) {
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

      const dir = await this.readdir("gammacv")
      if (!dir)
        await this.mkdir("gammacv")

      this.fileWrite("gammacv", output.data.slice(0, 100))

      // show output
      const canvas = document.createElement("canvas");
      canvas.width = output.shape[1];
      canvas.height = output.shape[0];
      gm.canvasFromTensor(canvas, output);

      return canvas;
    },
    fileWrite(dir, data) {
      try {
        Filesystem.writeFile({
          path: `${dir}/output.txt`,
          data: data.toString(),
          directory: FilesystemDirectory.Documents,
          encoding: FilesystemEncoding.UTF8
        })
      } catch(e) {
        console.error('Unable to write file', e);
      }
    },
    async fileDelete(path) {
      await Filesystem.deleteFile({
        path,
        directory: FilesystemDirectory.Documents
      });
    },
    async mkdir(dir) {
      try {
        let ret = await Filesystem.mkdir({
          path: dir,
          directory: FilesystemDirectory.Documents,
          createIntermediateDirectories: false // like mkdir -p
        });
      } catch(e) {
        console.error('Unable to make directory', e);
      }
    },
    async readdir(dir) {
      try {
        let ret = await Filesystem.readdir({
          path: dir,
          directory: FilesystemDirectory.Documents
        });
      } catch(e) {
        console.error('Unable to read dir', e);
      }
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
