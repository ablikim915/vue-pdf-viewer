<template>
  <div id="app">
    <pdfViewer></pdfViewer>
  </div>
</template>

<script>
import pdfViewer from "./components/pdf-viewer.vue";

export default {
  name: "App",
  components: {
    pdfViewer
  }
};
</script>

<style lang="less">
html,
body {
  margin: 0;
  padding: 0;
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
</style>
