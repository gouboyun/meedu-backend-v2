<template>
  <div class="meedu-main-body">
    <back-bar class="mb-30" title="付费学员"></back-bar>
    <div class="float-left mb-30">
      <p-button
        text="添加学员"
        type="primary"
        p="addons.learnPaths.path.user.store"
        @click="showUserAddWin = true"
      ></p-button>
      <el-button @click="exportexcel" type="primary">导出表格</el-button>
    </div>
    <div class="float-left" v-loading="loading">
      <div class="float-left">
        <el-table
          :header-cell-style="{ background: '#f1f2f9' }"
          :data="list"
          @selection-change="handleSelectionChange"
          class="float-left"
        >
          <!--<el-table-column type="selection" width="55"></el-table-column>-->
          <el-table-column prop="id" label="ID" width="120"> </el-table-column>
          <el-table-column prop="user_id" label="学员ID" width="120">
          </el-table-column>
          <el-table-column label="学员" width="300">
            <template slot-scope="scope">
              <div class="user-item d-flex" v-if="scope.row.user">
                <div class="avatar">
                  <img :src="scope.row.user.avatar" width="40" height="40" />
                </div>
                <div class="ml-10">
                  {{ scope.row.user.nick_name }}
                </div>
              </div>
              <span v-else class="c-red">学员不存在</span>
            </template>
          </el-table-column>
          <el-table-column label="价格" width="150">
            <template slot-scope="scope">
              <span>{{ scope.row.charge }}元</span>
            </template>
          </el-table-column>
          <el-table-column label="时间">
            <template slot-scope="scope">{{
              scope.row.updated_at | dateFormat
            }}</template>
          </el-table-column>
          <el-table-column fixed="right" label="操作" width="120" align="right">
            <template slot-scope="scope">
              <p-link
                text="删除"
                type="danger"
                @click="delUser(scope.row.user_id)"
                p="addons.learnPaths.path.user.destroy"
              ></p-link>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <div class="float-left mt-30 text-center">
        <el-pagination
          @size-change="paginationSizeChange"
          @current-change="paginationPageChange"
          :current-page="pagination.page"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="pagination.size"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        >
        </el-pagination>
      </div>
    </div>

    <user-single-add
      :show="showUserAddWin"
      @close="showUserAddWin = false"
      @confirm="userAddChange"
    ></user-single-add>
  </div>
</template>

<script>
import moment from "moment";
import UserSingleAdd from "@/components/user-single-add";

export default {
  components: {
    UserSingleAdd,
  },
  data() {
    return {
      pageName: "learningUser-list",
      showUserAddWin: false,
      pagination: {
        page: 1,
        size: 10,
      },
      filter: {
        id: this.$route.query.id,
        user_id: "",
      },
      total: 0,
      loading: false,
      list: [],
      selectedRows: null,
    };
  },
  watch: {
    "$route.query.id"() {
      this.pagination.page = 1;
      this.filter.user_id = "";
    },
  },
  activated() {
    this.getData();
    this.$utils.scrollTopSet(this.pageName);
  },
  beforeRouteLeave(to, from, next) {
    this.$utils.scrollTopRecord(this.pageName);
    next();
  },
  methods: {
    firstPageLoad() {
      this.pagination.page = 1;
      this.getData();
    },
    paginationReset() {
      this.pagination.page = 1;
      this.filter.user_id = null;
      this.getData();
    },
    paginationSizeChange(size) {
      this.pagination.page = 1;
      this.pagination.size = size;
      this.getData();
    },
    paginationPageChange(page) {
      this.pagination.page = page;
      this.getData();
    },
    handleSelectionChange(rows) {
      this.selectedRows = rows;
    },
    getData() {
      if (this.loading) {
        return;
      }
      this.loading = true;
      let params = {};
      this.filter.id = this.$route.query.id;
      Object.assign(params, this.filter, this.pagination);
      this.$api.Course.LearnPath.Path.Users(this.$route.query.id, params).then(
        (res) => {
          this.loading = false;
          this.list = res.data.data;
          this.total = res.data.total;
        }
      );
    },
    delUser(uid) {
      this.$confirm("确认删除此学员？", "警告", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$api.Course.LearnPath.Path.DelUser(this.$route.query.id, uid)
            .then(() => {
              this.$message.success(this.$t("common.success"));
              this.firstPageLoad();
            })
            .catch((e) => {
              this.$message.error(e.message);
            });
        })
        .catch(() => {});
    },
    userAddChange(uid) {
      this.$api.Course.LearnPath.Path.AddUser(this.$route.query.id, {
        user_id: uid,
      })
        .then(() => {
          this.$message.success(this.$t("common.success"));
          this.firstPageLoad();
          this.showUserAddWin = false;
        })
        .catch((e) => {
          this.$message.error(e.message);
        });
    },
    exportexcel() {
      if (this.loading) {
        return;
      }
      this.loading = true;

      let params = {
        page: 1,
        size: this.total,
      };
      this.filter.topic_id = this.$route.query.id;
      Object.assign(params, this.filter);

      this.$api.Course.LearnPath.Path.Users(this.$route.query.id, params).then(
        (res) => {
          if (res.data.total === 0) {
            this.$message.error("数据为空");
            this.loading = false;
            return;
          }

          let filename = "学习路径订阅学员.xlsx";
          let sheetName = "sheet1";

          let data = [["学员ID", "学员", "手机号", "价格", "时间"]];
          res.data.data.forEach((item) => {
            data.push([
              item.user_id,
              item.user.nick_name,
              item.user.mobile,
              item.charge === 0 ? "-" : "￥" + item.charge,
              item.updated_at
                ? moment(item.updated_at).format("YYYY-MM-DD HH:mm")
                : "",
            ]);
          });
          let wscols = [
            { wch: 10 },
            { wch: 20 },
            { wch: 15 },
            { wch: 20 },
            { wch: 20 },
          ];
          this.$utils.exportExcel(data, filename, sheetName, wscols);
          this.loading = false;
        }
      );
    },
  },
};
</script>

<style lang="less" scoped></style>
