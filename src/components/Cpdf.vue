<template>
  <div class="cpdf">
    <div class="center">
      <div class="contor">
        <button @click="prev">上一页</button>
        <button @click="next">下一页</button>
        <span>Page:
          <span v-text="page_num"></span> /
          <span v-text="page_count"></span>
        </span>
        <button @click="addscale" icon="plus">放大</button>
        <button @click="minus" icon="minus">缩小</button>
        <button id="prev" @click="closepdf">关闭</button>
      </div>
      <canvas class="canvasstyle" id="the-canvas"></canvas>
    </div>
  </div>
</template>
<script>
import PDFJS from "../../static/pdf.js";
//import { mapActions,mapGetters } from 'vuex';
export default {
  name: "CPdf",
  props: ["pdfurl"],
  data() {
    return {
      pdfDoc: null, //pdfjs 生成的对象
      pageNum: 1, //
      pageRendering: false,
      pageNumPending: null,
      scale: 1.2, //放大倍数
      page_num: 0, //当前页数
      page_count: 0, //总页数
      maxscale: 2, //最大放大倍数
      minscale: 0.8, //最小放大倍数
      PIXEL_RATIO: ""
    };
  },
  created() {
    // 获取设备像素比
    this.PIXEL_RATIO = (function() {
      var ctx = document.createElement("canvas").getContext("2d"),
        dpr = window.devicePixelRatio || 1,
        bsr =
          ctx.webkitBackingStorePixelRatio ||
          ctx.mozBackingStorePixelRatio ||
          ctx.msBackingStorePixelRatio ||
          ctx.oBackingStorePixelRatio ||
          ctx.backingStorePixelRatio ||
          1;
      return dpr / bsr;
    })();
    console.log(this.PIXEL_RATIO);
  },
  methods: {
    renderPage(num) {
      //渲染pdf
      let vm = this;
      this.pageRendering = true; //页面呈现
      let canvas = document.getElementById("the-canvas");
      var ctx = canvas.getContext("2d"),
        file,
        reader = new FileReader();
      // 使用承诺获取页面
      this.pdfDoc.getPage(num).then(function(page) {
        var viewport = page.getViewport(vm.scale);
        //alert(vm.canvas.height)
        canvas.height = viewport.height * vm.PIXEL_RATIO;
        canvas.width = viewport.width * vm.PIXEL_RATIO;
        canvas.style.width = viewport.width * vm.PIXEL_RATIO;
        canvas.style.height = viewport.height * vm.PIXEL_RATIO;
        ctx.scale(2, 2);
        // 将PDF页面呈现在画布上下文中。
        var renderContext = {
          canvasContext: vm.ctx,
          viewport: viewport
        };
        var renderTask = page.render(renderContext);

        // 等待渲染完成。
        renderTask.promise.then(function() {
          vm.pageRendering = false;
          if (vm.pageNumPending !== null) {
            // 新的页面呈现正在等待中。
            vm.renderPage(vm.pageNumPending);
            vm.pageNumPending = null;
          }
        });
      });
      vm.page_num = vm.pageNum;
    },
    addscale() {
      //放大
      if (this.scale >= this.maxscale) {
        return;
      }
      this.scale += 0.1;
      this.queueRenderPage(this.pageNum);
    },
    minus() {
      //缩小
      if (this.scale <= this.minscale) {
        return;
      }
      this.scale -= 0.1;
      this.queueRenderPage(this.pageNum);
    },
    prev() {
      //上一页
      let vm = this;
      if (vm.pageNum <= 1) {
        return;
      }
      vm.pageNum--;
      vm.queueRenderPage(vm.pageNum);
    },
    next() {
      //下一页
      let vm = this;
      if (vm.pageNum >= vm.page_count) {
        return;
      }
      vm.pageNum++;
      vm.queueRenderPage(vm.pageNum);
    },
    closepdf() {
      //关闭PDF
      this.$emit("closepdf");
    },
    queueRenderPage(num) {
      if (this.pageRendering) {
        this.pageNumPending = num;
      } else {
        this.renderPage(num);
      }
    }
  },
  computed: {
    ctx() {
      let id = document.getElementById("the-canvas");
      return id.getContext("2d");
    }
  },
  mounted() {
    let vm = this;
    PDFJS.getDocument(vm.pdfurl).then(function(pdfDoc_) {
      //初始化pdf
      vm.pdfDoc = pdfDoc_;
      vm.page_count = vm.pdfDoc.numPages;
      vm.renderPage(vm.pageNum);
    });
  }
};
</script>
<style>
.cpdf {
  position: fixed;
  top: 0;
  left: 0;
  background-color: rgba(0, 0, 0, 0.5);
  width: 100%;
  height: 100%;
  z-index: 99999;
  display: flex;
  justify-content: center;
  align-items: center;
}

.center {
  text-align: center;
  height: 100%;
  overflow: auto;
  padding-top: 20px;
}

.contor {
  margin-bottom: 10px;
}
</style>
