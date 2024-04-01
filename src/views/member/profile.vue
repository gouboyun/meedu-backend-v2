<template>
  <div class="user-main-body" v-if="user" v-loading="loading">
    <div class="float-left bg-white br-15 p-30">
      <back-bar class="mb-30" title="实名信息"></back-bar>
      <div class="panel-info-box">
        <div class="panel-info-item">
          真实姓名： {{ user.profile ? user.profile.real_name : "" }}
        </div>
        <div class="panel-info-item">
          身份证号码： {{ user.profile ? user.profile.id_number : "" }}
        </div>
      </div>
    </div>
    <div class="panel-box mt-30">
      <div class="panel-body">
        <div class="user-extra-info-box">
          <div class="float-left mb-15 d-flex">
            <div class="info-item">
              <div class="info-label">认证照片：</div>
              <div class="info-value">
                <template v-if="user.profile && user.profile.verify_image_url">
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
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { mapState } from "vuex";
export default {
  data() {
    return {
      id: null,
      user: null,
      loading: false,
    };
  },
  mounted() {
    this.id = this.$route.params.userId;

    this.getUser();
  },
  methods: {
    getUser() {
      if (this.loading) {
        return;
      }
      this.loading = true;
      this.$api.Member.Detail(this.id).then((res) => {
        this.user = res.data.data;
        this.loading = false;
      });
    },
  },
};
</script>
<style lang="less" scoped>
.user-main-body {
  width: 100%;
  height: auto;
  float: left;
  .panel-info-box {
    width: 100%;
    height: auto;
    float: left;
    box-sizing: border-box;
    padding-top: 20px;
    display: grid;
    row-gap: 30px;
    column-gap: 0px;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    .panel-info-item {
      height: 14px;
      font-size: 14px;
      font-weight: 400;
      color: #333333;
      line-height: 14px;
    }
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
