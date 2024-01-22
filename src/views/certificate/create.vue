<template>
  <div class="bg" v-loading="loading">
    <div class="certificate-content">
      <div class="top-box">
        <div class="btn-back" @click="$router.back()">
          <i class="el-icon-back"></i>
          返回
        </div>
        <div class="line"></div>
        <div class="name">新建证书</div>
      </div>
      <div
        :class="{
          noleft: leftArrrow,
        }"
        class="left-arrrow"
        @click="leftArrrow = !leftArrrow"
      >
        <img
          v-if="leftArrrow"
          src="@/assets/images/certificate/icon-unfold.png"
          width="44"
          height="44"
        />
        <img
          v-else
          src="@/assets/images/certificate/icon-fold.png"
          width="44"
          height="44"
        />
      </div>
      <transition name="slider">
        <div class="certificate-blocks-box" v-show="!leftArrrow">
          <div class="title">基本信息</div>
          <div class="float-left">
            <el-form
              ref="form"
              :model="course"
              :rules="rules"
              class="float-left"
              label-width="90px"
            >
              <div class="float-left">
                <el-form-item label="证书名称" prop="name">
                  <div class="d-flex">
                    <div>
                      <el-input
                        maxlength="64"
                        v-model="course.name"
                        class="w-250px"
                        placeholder="填写证书名称"
                      ></el-input>
                    </div>
                  </div>
                </el-form-item>

                <el-form-item prop="template_image" label="证书背景">
                  <upload-image
                    :height="240"
                    v-model="course.template_image"
                    :name="uploadName"
                    :hideImage="true"
                  ></upload-image>
                  <div
                    class="left-preview-box float-left mt-15"
                    v-if="course.template_image"
                  >
                    <img
                      :style="{
                        'max-width': '180px',
                        width: 'auto',
                        'max-height': '240px',
                      }"
                      :src="course.template_image"
                    />
                  </div>
                </el-form-item>
                <el-form-item
                  class="is-required"
                  label="证书元素"
                  v-if="course.template_image"
                >
                  <helper-text
                    text="拖动元素到证书背景上编辑参数"
                  ></helper-text>
                </el-form-item>
              </div>
            </el-form>
          </div>
          <draggable
            v-if="course.template_image"
            class="blocks"
            :sort="false"
            :group="{ name: 'blocks', pull: 'clone', put: false }"
          >
            <div class="block-item" sign="text-v1">
              <div class="btn">
                <div class="icon">
                  <img
                    src="@/assets/images/certificate/icon-txt.png"
                    width="44"
                    height="44"
                  />
                </div>
                <div class="name">文本</div>
              </div>
            </div>
            <div class="block-item" sign="image-v1">
              <div class="btn">
                <div class="icon">
                  <img
                    src="@/assets/images/certificate/icon-img.png"
                    width="44"
                    height="44"
                  />
                </div>
                <div class="name">图片</div>
              </div>
            </div>
            <div class="block-item" sign="qrcode-v1">
              <div class="btn">
                <div class="icon">
                  <img
                    src="@/assets/images/certificate/icon-qrcode.png"
                    width="44"
                    height="44"
                  />
                </div>
                <div class="name">证书二维码</div>
              </div>
            </div>
          </draggable>
          <template v-if="course.template_image">
            <div class="d-flex float-left mt-30">
              <div class="label">关联学习</div>
              <helper-text text="添加证书二维码查询"></helper-text>
            </div>
            <div class="float-left mt-30">
              <div class="d-flex float-left">
                <div class="label-item">关联课程</div>
                <el-button @click="showSelectResourceCoursesWin = true"
                  >添加关联</el-button
                >
                <select-resource
                  :selectedVod="coursesVodId"
                  :selectedLive="coursesLiveId"
                  :show="showSelectResourceCoursesWin"
                  @close="showSelectResourceCoursesWin = false"
                  @change="changeCourses"
                  enabled-resource="vod,live"
                ></select-resource>
              </div>
              <div
                class="courses-multi-box float-left"
                v-if="coursesData.length > 0"
              >
                <template v-for="(item, index) in coursesData">
                  <div :key="index" class="courses-multi-item">
                    <img
                      @click="delCourses(index)"
                      class="close"
                      src="@/assets/img/icon-close-h.png"
                      width="15"
                      height="15"
                    />
                    <img :src="item.thumb" width="80" height="60" />
                  </div>
                </template>
              </div>
              <div class="d-flex float-left mt-30">
                <div class="label-item">关联考试</div>
                <el-button @click="showSelectResourcePaperWin = true"
                  >添加关联</el-button
                >
                <select-resource
                  :selectedPaper="paperId"
                  :show="showSelectResourcePaperWin"
                  @close="showSelectResourcePaperWin = false"
                  @change="changePaper"
                  enabled-resource="paper,mock_paper"
                ></select-resource>
              </div>
              <div
                class="paper-multi-box float-left"
                v-if="paperData.length > 0"
              >
                <template v-for="(item, index) in paperData">
                  <div :key="index" class="paper-multi-item">
                    <img
                      @click="delPaper(index)"
                      class="close"
                      src="@/assets/img/icon-close-h.png"
                      width="15"
                      height="15"
                    />
                    {{ item.type === "paper" ? "考试卷-" : "模拟卷-"
                    }}{{ item.title }}
                  </div>
                </template>
              </div>
            </div>
          </template>
        </div>
      </transition>
      <div
        class="choose_size_box"
        :class="{ rightArrow: curBlockIndex !== null }"
        v-if="image"
      >
        <div class="tab_narrow" @click="changeSize(-1)">
          <img
            src="@/assets/images/certificate/low.png"
            width="12"
            height="12"
          />
        </div>
        <div class="choose_size ml-15 mr-15">
          <el-dropdown trigger="click">
            <span class="el-dropdown-link"> {{ size * 100 }}% </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item @click.native="changeSize(2)"
                >200%</el-dropdown-item
              >
              <el-dropdown-item @click.native="changeSize(1.5)"
                >150%</el-dropdown-item
              >
              <el-dropdown-item @click.native="changeSize(1)"
                >100%</el-dropdown-item
              >
              <el-dropdown-item @click.native="changeSize(0.5)"
                >50%</el-dropdown-item
              >
              <el-dropdown-item @click.native="changeSize(0.25)"
                >25%</el-dropdown-item
              >
            </el-dropdown-menu>
          </el-dropdown>
        </div>
        <div class="tab_enlarge" @click="changeSize(0)">
          <img
            src="@/assets/images/certificate/high.png"
            width="12"
            height="12"
          />
        </div>
      </div>
      <vue-drag-resize
        class="certificate-preview-box"
        w="auto"
        h="auto"
        :x="dragX"
        :y="dragY"
        ref="dragBox"
        @dragging="onDrag"
        :isResizable="false"
        :isActive="true"
      >
        <draggable
          v-if="originalWidth && originalHeight"
          ref="preview-box"
          class="preview-box"
          :style="{
            width: originalWidth + 'px',
            height: originalHeight + 'px',
          }"
          draggable=".block-item"
          group="blocks"
          @add="dragChange"
        >
          <div
            class="image-box"
            :style="{
              'background-image': 'url(' + image + ')',
              'background-position': 'center center',
              'background-size': 'cover',
              'background-repeat': 'no-repeat',
            }"
          >
            <template v-for="(item, index) in blocksData">
              <render-image-v1
                :key="index"
                v-if="item.sign === 'image-v1' && fresh"
                :current="index"
                :config="item.config"
                :status="curBlockIndex"
                :size="size"
                @dragend="changePosition"
                @change="changeIndex"
                @del="delBlockMethod"
              ></render-image-v1>
              <render-qrcode-v1
                :key="index"
                v-else-if="item.sign === 'qrcode-v1' && fresh"
                :current="index"
                :config="item.config"
                :status="qrcodeStatus"
                :curindex="curBlockIndex"
                :size="size"
                @dragend="changePosition"
                @change="changeIndex"
                @del="delBlockMethod"
              ></render-qrcode-v1>
              <render-text-v1
                :key="index"
                v-else-if="item.sign === 'text-v1' && fresh"
                :current="index"
                :config="item.config"
                :status="curBlockIndex"
                :maxw="originalWidth"
                :size="size"
                @dragend="changePosition"
                @change="changeIndex"
                @del="delBlockMethod"
                @fresh="freshing"
              ></render-text-v1>
            </template>
          </div>
        </draggable>
      </vue-drag-resize>
      <div
        class="certificate-config-box"
        :class="{
          active: rightIndex,
        }"
        v-if="curBlockIndex !== null"
      >
        <div class="float-left mb-15">
          <el-button class="ml-15 mt-15" @click="curBlockIndex = null">
            <i class="el-icon-close"></i><span class="ml-5">关闭配置</span>
          </el-button>
        </div>
        <config-setting
          :block="blocksData[curBlockIndex]"
          :index="curBlockIndex"
          @create="createQrcode"
          @change="getIndex"
        ></config-setting>
      </div>
      <div class="bottom-menus">
        <div class="bottom-menus-box" style="left: 0; z-index: 1000">
          <div>
            <el-button @click="formValidate" :loading="loading" type="primary"
              >保存</el-button
            >
          </div>
          <div class="ml-24">
            <el-button @click="$router.back()">取消</el-button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import config from "@/js/config";
import demoImg from "@/assets/home/demo.png";
import UploadImage from "@/components/upload-image";
import VueDragResize from "vue-drag-resize";
import draggable from "vuedraggable";
import ConfigSetting from "./components/certificate-config.vue";
import RenderTextV1 from "./render/text-v1.vue";
import RenderImageV1 from "./render/image-v1.vue";
import RenderQrcodeV1 from "./render/qrcode-v1.vue";
import SelectResource from "@/components/select-resources/multiIndex";
export default {
  components: {
    VueDragResize,
    draggable,
    ConfigSetting,
    RenderTextV1,
    RenderImageV1,
    RenderQrcodeV1,
    UploadImage,
    SelectResource,
  },
  data() {
    return {
      loading: false,
      params: "",
      uploadName: "上传背景",
      blocksData: [],
      curBlockIndex: null,
      qrcodeStatus: null,
      image: null,
      originalHeight: null,
      originalWidth: null,
      imgHeight: null,
      imgWidth: null,
      course: {
        name: null,
        template_image: null,
        params: null,
        courses: null,
      },
      fresh: true,
      rules: {
        name: [
          {
            required: true,
            message: "证书名不能为空",
            trigger: "blur",
          },
        ],
        template_image: [
          {
            required: true,
            message: "请上传证书",
            trigger: "blur",
          },
        ],
      },
      categories: [],
      leftArrrow: false,
      size: 0.5,
      dragX: 0,
      dragY: 106,
      rightIndex: null,
      showSelectResourceCoursesWin: false,
      showSelectResourcePaperWin: false,
      coursesData: [],
      paperData: [],
    };
  },
  watch: {
    "course.template_image"(val) {
      this.image = val;
      this.getImgInfo();
      if (val) {
        this.uploadName = "重新上传";
      } else {
        this.uploadName = "上传背景";
      }
    },
    size() {
      this.fresh = false;
      this.$nextTick(() => {
        this.fresh = true;
      });
    },
  },
  computed: {
    coursesVodId() {
      let params = [];
      if (this.coursesData.length > 0) {
        for (let i = 0; i < this.coursesData.length; i++) {
          if (this.coursesData[i].type === "vod") {
            params.push(this.coursesData[i].id);
          }
        }
      }
      return params;
    },
    coursesLiveId() {
      let params = [];
      if (this.coursesData.length > 0) {
        for (let i = 0; i < this.coursesData.length; i++) {
          if (this.coursesData[i].type === "live") {
            params.push(this.coursesData[i].id);
          }
        }
      }
      return params;
    },
    paperId() {
      let params = [];
      if (this.paperData.length > 0) {
        for (let i = 0; i < this.paperData.length; i++) {
          params.push(this.paperData[i].id);
        }
      }
      return params;
    },
  },
  mounted() {
    window.addEventListener("mousewheel", this.handleScroll, {
      passive: false,
    });
    this.keyDown();
  },
  beforeDestroy() {
    window.removeEventListener("mousewheel", this.handleScroll, {
      passive: false,
    });
  },
  methods: {
    changeCourses(data) {
      this.coursesData = this.coursesData.concat(data);
      this.showSelectResourceCoursesWin = false;
    },
    delCourses(index) {
      this.coursesData.splice(index, 1);
    },
    changePaper(data) {
      this.paperData = this.paperData.concat(data);
      this.showSelectResourcePaperWin = false;
    },
    delPaper(index) {
      this.paperData.splice(index, 1);
    },
    freshing() {
      this.fresh = false;
      this.$nextTick(() => {
        this.fresh = true;
      });
    },
    keyDown() {
      document.onkeydown = (e) => {
        let e1 =
          e || event || window.event || arguments.callee.caller.arguments[0];
        if (
          (e1.ctrlKey === true || e1.metaKey === true) &&
          (e1.which === 61 ||
            e1.which === 107 ||
            e1.which === 173 ||
            e1.which === 109 ||
            e1.which === 187 ||
            e1.which === 189)
        ) {
          e1.preventDefault();
          if (e1.which === 187) {
            this.changeSize(0);
          } else if (e1.which === 189) {
            this.changeSize(-1);
          }
        }
        //键盘按键判断:左箭头-37;上箭头-38；右箭头-39;下箭头-40
        if (e1 && e1.keyCode == 37) {
          if (!this.image) {
            return;
          }
          this.dragX -= 50;
        } else if (e1 && e1.keyCode == 39) {
          if (!this.image) {
            return;
          }
          this.dragX += 50;
        } else if (e1 && e1.keyCode == 38) {
          if (!this.image) {
            return;
          }
          this.dragY -= 50;
        } else if (e1 && e1.keyCode == 40) {
          if (!this.image) {
            return;
          }
          this.dragY += 50;
        }
      };
    },
    handleScroll(e) {
      // 判断是不是按下ctrl键
      if (e.ctrlKey || e.metaKey) {
        // 取消浏览器默认的放大缩小网页行为
        e.preventDefault();
        // 判断是向上滚动还是向下滚动
        if (e.deltaY > 0) {
          // 放大重写，业务代码
          this.changeSize(0);
        } else {
          // 缩小重写，业务代码
          this.changeSize(-1);
        }
      }
    },
    isJSON(str) {
      if (typeof str == "string") {
        try {
          var obj = JSON.parse(str);
          if (typeof obj == "object" && obj) {
            return true;
          } else {
            return false;
          }
        } catch (e) {
          return false;
        }
      }
    },
    changeIndex(index) {
      this.curBlockIndex = index;
    },
    delBlockMethod(index) {
      this.blockDestroy(index);
    },
    getIndex(val) {
      this.rightIndex = val;
    },
    onDrag(e) {
      this.dragX = e.left;
      this.dragY = e.top;
    },
    changeSize(val) {
      if (!this.image) {
        this.$message.error("请上传证书背景后在改变缩放比例");
        return;
      }
      if (val === -1) {
        if (this.size === 0.25) {
          return;
        }
        this.size -= 0.25;
        this.originalHeight = this.size * this.imgHeight;
        this.originalWidth = this.size * this.imgWidth;
        this.dragX = 0.5 * (window.screen.width - this.originalWidth);
        this.dragY = 106;
      } else if (val === 0) {
        if (this.size === 2) {
          return;
        }
        this.size += 0.25;
        this.originalHeight = this.size * this.imgHeight;
        this.originalWidth = this.size * this.imgWidth;
        this.dragX = 0.5 * (window.screen.width - this.originalWidth);
        this.dragY = 106;
      } else {
        this.size = val;
        this.originalHeight = this.size * this.imgHeight;
        this.originalWidth = this.size * this.imgWidth;
        this.dragX = 0.5 * (window.screen.width - this.originalWidth);
        this.dragY = 106;
      }
    },
    getImgInfo() {
      let self = this;
      let img = new Image();
      img.src = self.image;
      img.onload = function () {
        self.imgHeight = img.height;
        self.imgWidth = img.width;
        self.originalHeight = self.size * img.height;
        self.originalWidth = self.size * img.width;
        console.log("图片原始高度", img.height);
        console.log("图片原始宽度", img.width);
        self.dragX = 0.5 * (window.screen.width - self.originalWidth);
        self.dragY = 106;
      };
    },
    formValidate() {
      this.$refs["form"].validate((valid) => {
        if (valid) {
          this.confirm();
        }
      });
    },
    changePosition(sign, index, moveX, moveY) {
      for (let i = 0; i < this.blocksData.length; i++) {
        if (this.blocksData[i].sign === sign && i === index) {
          let mx = moveX;
          let my = moveY;
          let maxHeight;
          let maxWidth;
          if (my >= 0 && mx >= 0) {
            this.blocksData[i].config.x = mx;
            this.blocksData[i].config.y = my;
          }
        }
      }
      // console.log(
      //   "图片坐标：",
      //   this.blocksData[i].config.x,
      //   this.blocksData[i].config.y
      // );
    },
    close() {
      this.$emit("close");
    },
    createQrcode(val) {
      this.qrcodeStatus = val;
    },
    confirm() {
      if (this.loading) {
        return;
      }
      if (this.blocksData.length === 0) {
        this.$message.warning("请配置好证书元素");
        return;
      }
      let params = [];
      let reg = new RegExp("[\\u4E00-\\u9FFF]+", "g");
      let reg2 =
        /^(?:(http|https):\/\/)?((?:[\w-]+\.)+[a-z0-9]+)((?:\/[^/?#]*)+)?(\?[^#]+)?(#.+)?$/i;
      for (let i = 0; i < this.blocksData.length; i++) {
        if (this.blocksData[i].sign === "text-v1") {
          params.push({
            text: this.blocksData[i].config,
          });
        } else if (this.blocksData[i].sign === "image-v1") {
          params.push({
            image: this.blocksData[i].config,
          });
        } else if (this.blocksData[i].sign === "qrcode-v1") {
          let obj = this.blocksData[i].config.text;
          // if (reg.test(obj)) {
          //   this.curBlockIndex = i;
          //   this.$message.error("二维码网址仅支持英文、数字和符号");
          //   return;
          // } else if (!reg2.test(obj)) {
          //   this.curBlockIndex = i;
          //   this.$message.error("请输入正确的URL地址");
          //   return;
          // }
          params.push({
            qrcode: this.blocksData[i].config,
          });
        }
      }

      //console.log(params);
      this.course.params = JSON.stringify(params);

      // if (!this.isJSON(this.course.params)) {
      //   this.$message.error("请输入JSON字符串");
      //   return;
      // }
      let courses = [];
      if (this.coursesData.length > 0) {
        courses = courses.concat(this.coursesData);
      }
      if (this.paperData.length > 0) {
        courses = courses.concat(this.paperData);
      }
      this.course.courses = courses;

      this.loading = true;
      this.$api.Certificate.Store(this.course)
        .then(() => {
          this.$message.success(this.$t("common.success"));
          this.$router.back();
        })
        .catch((e) => {
          this.loading = false;
          this.$message.error(e.message);
        });
    },
    dragChange(e) {
      if (this.loading) {
        return;
      }
      this.loading = true;
      let blockSign = e.item.getAttribute("sign");
      // 移除dom
      e.item.parentNode.removeChild(e.item);
      // 默认数据
      let defaultConfig = null;
      if (blockSign === "text-v1") {
        defaultConfig = {
          x: e.originalEvent.layerX / this.size,
          y: e.originalEvent.layerY / this.size,
          font: "default",
          size: 40,
          color: "#333333",
          text: "默认文字",
        };
      } else if (blockSign === "image-v1") {
        defaultConfig = {
          x: e.originalEvent.layerX / this.size,
          y: e.originalEvent.layerY / this.size,
          width: 200,
          height: 200,
          url: demoImg,
        };
      } else if (blockSign === "qrcode-v1") {
        defaultConfig = {
          x: e.originalEvent.layerX / this.size,
          y: e.originalEvent.layerY / this.size,
          width: 200,
          height: 200,
          text: config.url + "/addons/Cert/dist/index.html",
        };
      }
      // 添加block
      this.blocksData.push({
        sign: blockSign,
        config: defaultConfig,
      });
      this.curBlockIndex = this.blocksData.length - 1;

      this.loading = false;
    },
    blockDestroy(index) {
      this.blocksData.splice(index, 1);
      this.curBlockIndex = null;
      this.fresh = false;
      this.$nextTick(() => {
        this.fresh = true;
      });
    },
    saveSetting(index, sign, config) {
      this.blocksData[index] = {
        sign: sign,
        config: config,
      };
    },
  },
};
</script>

<style lang="less" scoped>
@keyframes slideLeft {
  0% {
    transform: translateX(-400px);
  }
  100% {
    transform: translateX(0);
  }
}

@keyframes sliderRight {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(-400px);
  }
}

/deep/.el-form-item__label {
  width: 80px !important;
  text-align: left;
  padding: 0;
}

.certificate-preview-box.active:before {
  display: none;
}

.slider-enter-active {
  animation: slideLeft 0.2s linear;
  animation-fill-mode: forwards;
}

.slider-leave-active {
  animation: sliderRight 0.2s linear;
  animation-fill-mode: forwards;
}

.bg {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  width: 100%;
  height: 100%;
  background-color: #f1f2f9;
  z-index: 1000;
}
.left-preview-box {
  display: flex;
  width: 180px;
  height: 240px;
  justify-content: center;
  align-items: center;
}
.label {
  width: 80px;
  text-align: right;
  vertical-align: middle;
  float: left;
  font-size: 14px;
  font-weight: 600;
  color: #333333;
  line-height: 16px;
  padding: 0 12px 0 0;
  box-sizing: border-box;
  border-left: 4px solid #3ca7fa;
  margin-right: 10px;
}
.label-item {
  width: 90px;
  text-align: left;
  vertical-align: middle;
  float: left;
  font-size: 14px;
  color: #606266;
  line-height: 40px;
  box-sizing: border-box;
  padding: 0 0 0 10px;
  margin-right: 10px;
}
.courses-multi-box {
  width: 100%;
  box-sizing: border-box;
  padding-top: 22px;
  display: grid;
  row-gap: 22px;
  column-gap: 40px;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  .courses-multi-item {
    position: relative;
    width: 80px;
    height: 60px;
    .close {
      position: absolute;
      right: -7px;
      top: -7px;
      cursor: pointer;
      &:hover {
        opacity: 0.8;
      }
    }
  }
}
.paper-multi-box {
  width: 100%;
  box-sizing: border-box;
  padding-top: 22px;
  display: flex;
  flex-direction: column;
  .paper-multi-item {
    width: 100%;
    height: 40px;
    background: #f1f2f9;
    border-radius: 4px;
    margin-bottom: 22px;
    position: relative;
    font-size: 14px;
    font-weight: 400;
    color: #333333;
    display: flex;
    align-items: center;
    justify-content: center;
    .close {
      position: absolute;
      right: -7px;
      top: -7px;
      cursor: pointer;
      &:hover {
        opacity: 0.8;
      }
    }
    &:last-child {
      margin-bottom: 0px;
    }
  }
}
.top-box {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  display: flex;
  align-items: center;
  height: 56px;
  float: left;
  box-sizing: border-box;
  padding-left: 30px;
  padding-right: 30px;
  background-color: white;
  line-height: 56px;
  border-bottom: 1px solid #f2f2f2;
  z-index: 10;

  .btn-back {
    font-size: 14px;
    font-weight: 400;
    color: #333333;
    cursor: pointer;

    &:hover {
      color: @primary-color;
    }
  }

  .line {
    width: 1px;
    height: 14px;
    background-color: #d8d8d8;
    margin-right: 15px;
    margin-left: 15px;
  }

  .name {
    font-size: 14px;
    font-weight: 600;
    color: #333333;
  }
}
.left-arrrow {
  position: absolute;
  top: 75px;
  left: 384px;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  overflow: hidden;
  cursor: pointer;
  z-index: 10;
  &.noleft {
    left: -15px;
  }
}

.certificate-blocks-box {
  height: calc(100vh - 138px);
  position: absolute;
  top: 57px;
  /* left: 70px; */
  left: 0;
  /* border-radius: 4px; */
  /* will-change: contents; */
  transition: left ease 0.5s;
  width: 400px;
  background-color: white;
  box-sizing: border-box;
  padding: 30px;
  overflow-y: auto;
  z-index: 11111;
  .title {
    width: 100%;
    height: 16px;
    border-left: 4px solid #3ca7fa;
    font-size: 14px;
    line-height: 16px;
    font-weight: 600;
    color: #333333;
    box-sizing: border-box;
    padding-left: 10px;
    margin-bottom: 30px;
  }
  .line {
    width: 100%;
    height: 1px;
    background: #f1f2f9;
  }
  .blocks {
    width: 100%;
    height: auto;
    float: left;
    display: grid;
    box-sizing: border-box;
    padding: 0 20px;
    gap: 30px;
    grid-template-columns: repeat(3, minmax(0, 1fr));

    .block-item {
      text-align: center;

      .btn {
        display: inline-block;
        cursor: move;
        width: 60px;
        height: 60px;
        &:hover {
          box-shadow: 0px 4px 8px 0px rgba(102, 102, 102, 0.15);
          border-radius: 8px;
        }

        .name {
          font-size: 12px;
          font-weight: 400;
          color: #666666;
          line-height: 12px;
        }
      }
    }
  }
}
.certificate-content {
  /* will-change: auto; */
  width: 100%;
  height: 100%;
  position: absolute;
  overflow: hidden;
  box-sizing: border-box;
  top: 0;
  left: 0;
  background: #f0f2f5;
  z-index: 1002;
}
.choose_size_box {
  position: absolute;
  top: 86px;
  right: 30px;
  width: 103px;
  height: 40px;
  background: #ffffff;
  box-shadow: 0px 4px 8px 0px rgba(102, 102, 102, 0.15);
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
  cursor: pointer;
  &.rightArrow {
    right: 430px;
  }
  .tab_narrow {
    width: 20px;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: right;
  }
  .choose_size {
    display: inline-block;
    font-size: 14px;
    font-weight: 400;
    color: #333333;
    line-height: 20px;
  }
  .tab_enlarge {
    width: 20px;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: left;
  }
}
.certificate-preview-box {
  background: #f1f2f9;
  box-sizing: border-box;
  position: absolute;
  top: 0;
  left: 0;
  transition: none;
  background: transparent;
  z-index: 0;
  box-shadow: none;
  .preview-box {
    overflow: hidden;
  }
  .image-box {
    width: 100%;
    height: 100%;
    margin: 0 auto;
    position: relative;
    .image-render {
      width: 100%;
      height: 100%;
      display: block;
      .image {
        width: 100%;
        height: 100%;
      }
    }
    .item {
      position: absolute;
      cursor: pointer;
      &.active {
        box-sizing: border-box;
        border: 2px solid @primary-color;
      }

      .item-options {
        position: absolute;
        top: -2px;
        right: -36px;
        width: auto;
        height: 36px;

        .btn-item {
          color: white;
          background-color: @primary-color;
          width: 36px;
          height: 36px;
          float: left;
          text-align: center;
          line-height: 36px;

          &:hover {
            background-color: rgba(@primary-color, 0.8);
          }

          &:first-child {
            border-top-left-radius: 2px;
            border-bottom-left-radius: 2px;
          }

          &:last-child {
            border-top-right-radius: 2px;
            border-bottom-right-radius: 2px;
          }
        }
      }
    }
  }
}
.certificate-config-box {
  height: calc(100vh - 57px);
  position: absolute;
  top: 57px;
  right: 0;
  bottom: 0;
  width: 400px;
  box-sizing: border-box;
  background-color: white;
  border-left: 1px solid #f2f2f2;
  z-index: 100;
  &.active {
    z-index: 11111;
  }
}
.certificate-content-footer {
  width: 100%;
  height: auto;
  float: left;
  box-sizing: border-box;
  padding: 15px 30px;
  box-shadow: 0px -2px 4px 0px rgba(102, 102, 102, 0.05);
  .footer_btn {
    width: 88px;
  }
}
</style>
