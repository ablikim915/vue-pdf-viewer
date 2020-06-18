<template>
  <div class="pdf-viewer">
    <h1>浏览pdf</h1>
    <div>
      <button id="prev" @click="onPrevPage">Previous</button>
      <button id="next" @click="onNextPage">Next</button>
      &nbsp; &nbsp;
      <span
        >Page: <span id="page_num"></span> / <span id="page_count"></span
      ></span>
    </div>
    <canvas id="the-canvas"></canvas>
  </div>
</template>

<script>
import pdfjsLib from "../../static/build/pdf.js";
pdfjsLib.GlobalWorkerOptions.workerSrc =
  "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.4.456/pdf.worker.min.js";

export default {
  props: {
    src: {
      type: String,
      defualt: ""
    }
  },
  data() {
    return {
      url: "http://img.souche.com/f2e/53cb734d23cc346d8e697d559a7dfa46.pdf",
      pdfDoc: null,
      pageNum: 1,
      pageRendering: false,
      pageNumPending: null,
      scale: 1
    };
  },
  mounted() {
    pdfjsLib.getDocument(this.url).promise.then(pdfDoc_ => {
      this.pdfDoc = pdfDoc_;
      document.getElementById("page_count").textContent = this.pdfDoc.numPages;

      // Initial/first page rendering
      this.renderPage(this.pageNum);
    });
  },
  methods: {
    renderPage(num) {
      this.pageRendering = true;
      // Using promise to fetch the page
      this.pdfDoc.getPage(num).then(page => {
        const canvas = document.getElementById("the-canvas");
        const ctx = canvas.getContext("2d");
        var viewport = page.getViewport({ scale: this.scale });
        canvas.height = viewport.height;
        canvas.width = viewport.width;

        // Render PDF page into canvas context
        var renderContext = {
          canvasContext: ctx,
          viewport: viewport
        };
        console.log('---renderContext', renderContext)
        var renderTask = page.render(renderContext);
        console.log('---renderTask', renderTask);
        // Wait for rendering to finish
        renderTask.promise.then(() => {
          this.pageRendering = false;
          if (this.pageNumPending !== null) {
            // New page rendering is pending
            this.renderPage(this.pageNumPending);
            this.pageNumPending = null;
          }
        });
      });

      // Update page counters
      document.getElementById("page_num").textContent = num;
    },
    queueRenderPage(num) {
      if (this.pageRendering) {
        this.pageNumPending = num;
      } else {
        this.renderPage(num);
      }
    },
    onPrevPage() {
      if (this.pageNum <= 1) {
        return;
      }
      this.pageNum--;
      this.queueRenderPage(this.pageNum);
    },
    onNextPage() {
      if (this.pageNum >= this.pdfDoc.numPages) {
        return;
      }
      this.pageNum++;
      this.queueRenderPage(this.pageNum);
    }
  }
};
</script>

<style lang="less">
.pdf-viewer {
  #the-canvas {
    width: 100vw;
  }
}
</style>
