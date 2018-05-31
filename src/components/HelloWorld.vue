<template>
  <div class="dingidng">
    <c-pdf @closepdf="closepdf" v-show="isshowpdf" :pdfurl="testpdfurl"></c-pdf>
  </div>
</template>

<script>
import pdf from "vue-pdf";
import CPdf from "./Cpdf.vue";
export default {
  name: "HelloWorld",
  data() {
    return {
      msg: "Welcome to Your Vue.js App",
      isshowpdf: true,
      testpdfurl:
        "http://10.0.0.5:90/File/Patient/0000084672_1/PDF/TiWenDan.pdf"
    };
  },
  components: {
    pdf,
    CPdf
  },
  methods: {
    closepdf() {
      this.isshowpdf = false;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.dingidng {
  width: 100%;
  height: 100vh;
}

/* .move {
    touch-action: none;
    position: absolute;
  } */
</style>
