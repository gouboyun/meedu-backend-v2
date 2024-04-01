<template>
  <transition name="fade">
    <div class="meedu-dialog-mask">
      <div class="meedu-cate-dialog-box">
        <div class="meedu-dialog-header">实名信息</div>
        <div class="meedu-dialog-body">
          <div class="dialog-body">
            <template v-if="user.profile">
              <el-button @click="confirm" :loading="loading" type="primary">
                重置实名信息
              </el-button>
              <div class="panel-info-box">
                <div class="panel-info-item">
                  真实姓名： {{ user.profile ? user.profile.real_name : "" }}
                </div>
                <div class="panel-info-item">
                  身份证号码： {{ user.profile ? user.profile.id_number : "" }}
                </div>
              </div>
              <div class="float-left mb-15 d-flex">
                <div class="info-item">
                  <div class="info-label">认证照片：</div>
                  <div class="info-value">
                    <template
                      v-if="user.profile && user.profile.verify_image_url"
                    >
                      <el-image
                        style="width: 150px; height: 200px; border-radius: 8px"
                        :src="user.profile.verify_image_url"
                        lazy
                        :preview-src-list="[user.profile.verify_image_url]"
                      >
                      </el-image>
                    </template>
                    <div v-else class="image"></div>
                  </div>
                </div>
              </div>
            </template>
            <template v-else>
              <div class="none">
                <span>暂无实名信息</span>
              </div>
            </template>
          </div>
        </div>
        <div class="meedu-dialog-footer">
          <el-button @click="close"> 取消 </el-button>
        </div>
      </div>
    </div>
  </transition>
</template>
<script>
export default {
  props: ["id"],
  data() {
    return {
      user: {},
      loading: false,
      init: false,
    };
  },

  mounted() {
    if (this.id) {
      this.getUser();
    }
  },
  methods: {
    getUser() {
      this.$api.Member.Detail(this.id).then((res) => {
        this.user = res.data.data;
        this.$nextTick(() => {
          this.init = true;
        });
      });
    },
    close() {
      this.$emit("close");
    },
    confirm() {
      if (this.loading) {
        return;
      }
      this.loading = true;
      this.$api.Member.ResetProfile(this.id)
        .then(() => {
          this.loading = false;
          this.$message.success(this.$t("common.success"));
          this.$emit("success");
        })
        .catch((e) => {
          this.loading = false;
          this.$message.error(e.message);
        });
    },
  },
};
</script>
<style lang="less" scoped>
.none {
  width: 100%;
  height: 190px;
  display: flex;
  align-items: center;
  justify-content: center;
  span {
    color: #909399;
    font-size: 14px;
  }
}

.dialog-body {
  width: 100%;
  height: auto;
  box-sizing: border-box;
  padding: 10px 30px 30px 30px;
}
.panel-info-box {
  width: 100%;
  height: auto;
  float: left;
  box-sizing: border-box;
  padding-bottom: 20px;
  padding-top: 10px;
  display: grid;
  row-gap: 30px;
  column-gap: 0px;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  .panel-info-item {
    height: 14px;
    font-size: 14px;
    font-weight: 400;
    color: #333333;
    line-height: 14px;
  }
}

.info-item {
  width: 324px;
  height: auto;
  float: left;
  display: flex;
  flex-direction: column;
  .info-label {
    width: 100%;
    float: left;
    height: 14px;
    font-size: 14px;
    font-weight: 400;
    color: #333333;
    line-height: 14px;
    margin-bottom: 20px;
  }
  .info-value {
    width: 100%;
    height: auto;
    float: left;
    .image {
      width: 150px;
      height: 200px;
      background: #f0f0f8;
      border-radius: 8px;
    }
  }
}
</style>
