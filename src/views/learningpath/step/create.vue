<template>
  <div class="meedu-main-body">
    <back-bar class="mb-30" title="添加学习步骤"></back-bar>
    <div class="float-left">
      <el-form ref="form" :model="form" :rules="rules" label-width="200px">
        <div class="title mt-10">学习步骤基本信息</div>
        <el-form-item label="排序" prop="sort">
          <div class="d-flex">
            <div>
              <el-input
                type="number"
                v-model="form.sort"
                class="w-300px"
                placeholder="请输入整数"
              ></el-input>
            </div>
            <div class="ml-10">
              <helper-text
                text="请输入整数。小数排在前，大数排在后。"
              ></helper-text>
            </div>
          </div>
        </el-form-item>
        <el-form-item label="步骤名称" prop="name">
          <el-input
            v-model="form.name"
            class="w-300px"
            placeholder="请填写步骤名称"
          ></el-input>
        </el-form-item>
        <el-form-item prop="desc" label="步骤简介">
          <div class="d-flex">
            <div>
              <el-input
                v-model="form.desc"
                type="textarea"
                rows="4"
                maxlength="150"
                class="w-800px"
                placeholder="请填写此步骤简介"
              ></el-input>
            </div>
            <div class="ml-10">
              <helper-text
                text="该值会在学习步骤列表显示，建议不要超过150个字。"
              ></helper-text>
            </div>
          </div>
        </el-form-item>
        <div class="title mt-30">学习步骤关联课程</div>
        <div class="float-left mb-30">
          <el-button
            @click="showSelectResourceCoursesWin = true"
            type="primary"
          >
            添加课程
          </el-button>
          <select-resource
            :selectedVod="coursesVodId"
            :selectedLive="coursesLiveId"
            :selectedBook="bookId"
            :selectedPaper="paperId"
            :selectedPractice="practiceId"
            type="learnPath"
            :show="showSelectResourceCoursesWin"
            @close="showSelectResourceCoursesWin = false"
            @change="changeCourses"
            enabled-resource="vod,live,book,paper,practice"
          ></select-resource>
        </div>
        <div class="float-left">
          <el-table
            :header-cell-style="{ background: '#f1f2f9' }"
            :data="coursesData"
            class="float-left"
            :row-class-name="tableRowClassName"
          >
            <el-table-column label="关联课程" min-width="53%">
              <template slot-scope="scope">
                <template v-if="scope.row.type === 'book'">
                  <thumb-bar
                    :value="scope.row.thumb"
                    :width="90"
                    :height="120"
                    :title="scope.row.title"
                  ></thumb-bar>
                </template>
                <template
                  v-else-if="
                    scope.row.type === 'paper' || scope.row.type === 'practice'
                  "
                >
                  <thumb-bar
                    :value="require('@/assets/img/default-paper.png')"
                    :width="120"
                    :height="90"
                    :title="scope.row.title"
                  ></thumb-bar>
                </template>
                <template v-else>
                  <thumb-bar
                    :value="scope.row.thumb"
                    :width="120"
                    :height="90"
                    :title="scope.row.title"
                  ></thumb-bar>
                </template>
              </template>
            </el-table-column>
            <el-table-column label="课程类型" min-width="20%">
              <template slot-scope="scope">
                <span v-if="scope.row.type === 'vod'">录播课程</span>
                <span v-else-if="scope.row.type === 'live'">直播课程</span>
                <span v-else-if="scope.row.type === 'book'">电子书</span>
                <span v-else-if="scope.row.type === 'paper'">考试</span>
                <span v-else-if="scope.row.type === 'practice'">练习</span>
              </template>
            </el-table-column>
            <el-table-column label="单品价格" min-width="21%">
              <template slot-scope="scope">
                <span v-if="scope.row.charge === 0">免费</span>
                <span v-else>￥{{ scope.row.charge }}</span>
              </template>
            </el-table-column>
            <el-table-column
              fixed="right"
              label="操作"
              align="right"
              min-width="6%"
            >
              <template slot-scope="scope">
                <el-link
                  class="ml-5"
                  type="danger"
                  @click="delCourses(scope.row.row_index)"
                  >删除</el-link
                >
              </template>
            </el-table-column>
          </el-table>
        </div>
      </el-form>

      <div class="bottom-menus">
        <div class="bottom-menus-box">
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
import SelectResource from "@/components/select-resources/multiIndex";
export default {
  components: {
    SelectResource,
  },
  data() {
    return {
      form: {
        name: null,
        path_id: this.$route.query.path_id,
        sort: null,
        decs: "",
        desc: null,
        courses: [],
      },
      categories: [],
      rules: {
        name: [
          {
            required: true,
            message: "步骤名不能为空",
            trigger: "blur",
          },
        ],
        sort: [
          {
            required: true,
            message: "排序不能为空",
            trigger: "blur",
          },
        ],
        desc: [
          {
            required: true,
            message: "描述不能为空",
            trigger: "blur",
          },
        ],
      },
      loading: false,
      coursesData: [],
      showSelectResourceCoursesWin: false,
    };
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
      if (this.coursesData.length > 0) {
        for (let i = 0; i < this.coursesData.length; i++) {
          if (this.coursesData[i].type === "paper") {
            params.push(this.coursesData[i].id);
          }
        }
      }
      return params;
    },
    bookId() {
      let params = [];
      if (this.coursesData.length > 0) {
        for (let i = 0; i < this.coursesData.length; i++) {
          if (this.coursesData[i].type === "book") {
            params.push(this.coursesData[i].id);
          }
        }
      }
      return params;
    },
    practiceId() {
      let params = [];
      if (this.coursesData.length > 0) {
        for (let i = 0; i < this.coursesData.length; i++) {
          if (this.coursesData[i].type === "practice") {
            params.push(this.coursesData[i].id);
          }
        }
      }
      return params;
    },
  },
  mounted() {
    this.params();
  },
  methods: {
    tableRowClassName({ row, rowIndex }) {
      row.row_index = rowIndex;
    },
    changeCourses(data) {
      this.coursesData = this.coursesData.concat(data);
      this.showSelectResourceCoursesWin = false;
    },
    delCourses(index) {
      this.coursesData.splice(index, 1);
    },
    params() {
      this.$api.Course.LearnPath.Step.Create().then((res) => {
        this.categories = res.data.paths;
      });
    },
    formValidate() {
      this.$refs["form"].validate((valid) => {
        if (valid) {
          this.confirm();
        }
      });
    },
    confirm() {
      if (this.loading) {
        return;
      }
      if (this.coursesData.length === 0) {
        this.$message.warning("请添加学习步骤关联课程");
        return;
      }
      let params = [];
      if (this.coursesData.length > 0) {
        for (let i = 0; i < this.coursesData.length; i++) {
          let type = this.coursesData[i].type;
          if (type === "vod") {
            type = "course";
          }
          if (type === "practice") {
            type = "paper_practice";
          }
          if (type === "paper") {
            type = "paper_paper";
          }
          let item = {
            type: type,
            other_id: this.coursesData[i].id,
            name: this.coursesData[i].title,
            thumb: this.coursesData[i].thumb,
            charge: this.coursesData[i].charge,
          };
          params.push(item);
        }
      }
      this.form.courses = params;
      this.loading = true;
      this.$api.Course.LearnPath.NewStep.Store(this.form)
        .then(() => {
          this.$message.success(this.$t("common.success"));
          this.$router.back();
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
.title {
  width: 100%;
  height: 16px;
  border-left: 4px solid #3ca7fa;
  font-size: 14px;
  line-height: 16px;
  font-weight: 600;
  color: #333333;
  padding-left: 10px;
  margin-bottom: 30px;
}
</style>
