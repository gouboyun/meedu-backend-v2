<template>
  <div>
    <transition name="fade">
      <div class="meedu-dialog-mask">
        <div class="meedu-table-dialog-box">
          <div class="meedu-table-dialog-header">
            <span>学习进度</span>
            <img
              class="icon-close"
              @click="close()"
              src="@/assets/img/close.png"
            />
          </div>
          <div class="meedu-table-dialog-body" v-loading="loading">
            <el-table
              :data="list"
              class="float-left"
              row-key="id"
              :tree-props="{ children: 'children' }"
              v-loading="loading"
            >
              <el-table-column prop="video_title" label="标题">
              </el-table-column>
              <el-table-column label="进度" width="300">
                <template slot-scope="scope">
                  <div>
                    <duration-text
                      v-if="scope.row.watch_seconds > 0"
                      :duration="scope.row.watch_seconds"
                    ></duration-text>
                    <span v-else>0:00</span>
                    /
                    <duration-text
                      :duration="scope.row.duration"
                    ></duration-text>
                  </div>
                </template>
              </el-table-column>
              <el-table-column prop="url" label="状态" width="120">
                <template slot-scope="scope">
                  <span
                    class="c-green"
                    v-if="scope.row.watched_at"
                  >
                    已学完
                  </span>
                  <span v-else>未学完</span>
                </template>
              </el-table-column>
              <template v-if="enabledAddons['Snapshot']">
                <el-table-column label="已拍照片" width="120">
                  <template slot-scope="scope">
                    <el-link
                      type="primary"
                      v-if="
                        images[scope.row.video_id] &&
                        images[scope.row.video_id].images.length > 0
                      "
                      @click="showDialog(scope.row.video_id)"
                    >
                      {{ images[scope.row.video_id].images.length }}
                    </el-link>
                    <span v-else>-</span>
                  </template>
                </el-table-column>
              </template>
            </el-table>
          </div>
        </div>
      </div>
    </transition>
    <snapshot-dialog
      v-if="visiable"
      :uid="uid"
      :vid="vid"
      @close="hideDialog()"
    ></snapshot-dialog>
  </div>
</template>

<script>
import { mapState } from "vuex";
import DurationText from "@/components/duration-text";
import SnapshotDialog from "@/components/snapshot-dialog";

export default {
  components: {
    DurationText,
    SnapshotDialog,
  },
  props: ["cid", "uid"],
  data() {
    return {
      list: [],
      loading: false,
      visiable: false,
      vid: null,
      images: {},
    };
  },
  computed: {
    ...mapState(["enabledAddons"]),
  },
  mounted() {
    this.getData();
  },
  methods: {
    showDialog(vid) {
      this.vid = vid;
      this.visiable = true;
    },
    hideDialog() {
      this.vid = null;
      this.visiable = false;
      this.getData();
    },
    getData() {
      if (this.loading) {
        return;
      }
      this.loading = true;
      this.$api.Course.Vod.Records.Detail(this.cid, this.uid).then((res) => {
        this.loading = false;
        this.list = res.data.data;
        let arr = [];
        let data = res.data.data;
        if (data.length > 0) {
          data.map((item) => {
            arr.push(item.video_id);
          });
        }
        if (this.enabledAddons["Snapshot"]) {
          this.getImages(arr);
        }
      });
    },
    getImages(ids) {
      let params = {
        type: "vod",
        user_id: this.uid,
        other_ids: ids,
      };
      this.$api.Snapshot.Images.List(params)
        .then((res) => {
          this.images = res.data;
        })
        .catch((e) => {
          this.$message.error(e.message);
        });
    },
    close() {
      this.$emit("close");
    },
  },
};
</script>
