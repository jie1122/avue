<template>
  <div :class="b()">
    <div :class="b('mask')"
         v-if="ops.modal"
         @click="close"></div>
    <span class="el-image-viewer__btn el-image-viewer__close"
          @click="close">
      <el-icon>
        <close />
      </el-icon>
    </span>
    <span class="el-image-viewer__btn el-image-viewer__prev"
          @click="handlePrev()"
          v-if="isRrrow">
      <el-icon>
        <arrowLeft />
      </el-icon>
    </span>
    <span class="el-image-viewer__btn el-image-viewer__next"
          @click="handleNext()"
          v-if="isRrrow">
      <el-icon>
        <arrowRight />
      </el-icon>
    </span>
    <div :class="b('box')"
         ref="box">
      <el-carousel ref="carousel"
                   :show-indicators="false"
                   :initial-index="index"
                   :initial-swipe="index"
                   :interval="0"
                   arrow="never"
                   @change="handleChange"
                   indicator-position="none">
        <el-carousel-item @click.native.self="ops.closeOnClickModal?close():''"
                          v-for="(item,indexs) in datas"
                          :key="indexs">
          <component @click="handleClick(item,indexs)"
                     :id="'avue-image-preview__'+indexs"
                     :src="item.url"
                     :style="[styleName,styleBoxName]"
                     ref="item"
                     @mousedown="move"
                     controls="controls"
                     :is="isMediaType(item)"
                     v-if="isMediaType(item)"
                     ondragstart="return false"></component>
          <div v-else
               @click="handleClick(item,indexs,true)"
               :id="'avue-image-preview__'+indexs"
               :class="b('file')">
            <span>
              <el-icon>
                <Document />
              </el-icon>
              <p>{{item.name || getName(item.url)}}</p>
            </span>
          </div>
        </el-carousel-item>
      </el-carousel>
    </div>
    <div class="el-image-viewer__btn el-image-viewer__actions">
      <div class="el-image-viewer__actions__inner">
        <el-icon @click="subScale">
          <zoomOut />
        </el-icon>
        <el-icon @click="addScale">
          <zoomIn />
        </el-icon>
        <i class="el-image-viewer__actions__divider"></i>
        <el-icon @click="handlePrint">
          <printer />
        </el-icon>
        <i class="el-image-viewer__actions__divider"></i>
        <el-icon @click="rotate=rotate-90">
          <refreshLeft />
        </el-icon>
        <el-icon @click="rotate=rotate+90">
          <refreshRight />
        </el-icon>
      </div>
    </div>
  </div>
</template>
<script>
import { setPx, isMediaType } from 'utils/util';
import create from "core/create";
import { defineAsyncComponent } from 'vue'
import { Document, ArrowRight, ArrowLeft, Close, ZoomOut, ZoomIn, RefreshLeft, RefreshRight, Printer } from '@element-plus/icons-vue'
import { ElIcon, ElCarousel, ElCarouselItem } from 'element-plus'
import $Print from 'plugin/print/';
export default create({
  name: "image-preview",
  components: {
    ElIcon,
    ElCarousel,
    ElCarouselItem,
    Document,
    ArrowRight,
    ArrowLeft,
    Close,
    ZoomOut,
    ZoomIn,
    RefreshLeft,
    RefreshRight,
    Printer
  },
  props: {
    datas: Array,
    index: [Number, String],
    ops: Object,
    onDestroy: Function
  },
  data () {
    return {
      left: 0,
      top: 0,
      scale: 1,
      rotate: 0,
      count: this.index
    };
  },
  computed: {
    styleBoxName () {
      return {
        marginLeft: setPx(this.left),
        marginTop: setPx(this.top)
      }
    },
    styleName () {
      return {
        transform: `scale(${this.scale}) rotate(${this.rotate}deg)`,
        maxWidth: '100%',
        maxHeight: '100%',
      }
    },
    isRrrow () {
      return this.datas.length > 1
    }
  },
  methods: {
    getName (url) {
      return url.substring(url.lastIndexOf('/') + 1)
    },
    handlePrint () {
      $Print(`#avue-image-preview__${this.count}`)
    },
    handlePrev () {
      this.stopItem()
      this.$refs.carousel.prev()

    },
    handleNext () {
      this.stopItem()
      this.$refs.carousel.next()

    },
    stopItem () {
      this.left = 0;
      this.top = 0;
    },
    isMediaType (item) {
      return isMediaType(item.url, item.type)
    },
    subScale () {
      if (this.scale != 0.2) {
        this.scale = parseFloat((this.scale - 0.2).toFixed(2))
      }
    },
    addScale () {
      this.scale = parseFloat((this.scale + 0.2).toFixed(2))
    },
    handleChange (n, o) {
      this.scale = 1;
      this.rotate = 0;
      this.count = n

    },
    move (e) {       //获取目标元素s
      //算出鼠标相对元素的位置
      let disX = e.clientX;
      let disY = e.clientY;
      let scale = 2;
      document.onmousemove = (e) => {       //鼠标按下并移动的事件
        //用鼠标的位置减去鼠标相对元素的位置，得到元素的位置
        let left = e.clientX - disX;
        let top = e.clientY - disY;
        disX = e.clientX;
        disY = e.clientY;
        //移动当前元素
        this.left = this.left + (left * scale)
        this.top = this.top + (top * scale)

      };
      document.onmouseup = (e) => {
        document.onmousemove = null;
        document.onmouseup = null;
      };
    },
    handleClick (item, index, df = false) {
      if (typeof this.ops.click == "function") {
        this.ops.click(item, index);
      } else if (df) {
        window.open(item.url)
      }
    },
    close () {
      this.isShow = false
      if (typeof this.ops.beforeClose == "function") {
        this.ops.beforeClose(this.datas, this.count);
      }
      this.onDestroy()
    }
  }
});
</script>