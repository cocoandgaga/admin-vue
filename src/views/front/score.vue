<template>
  <div>
    <el-table :data="dataList" 
       v-loading="dataListLoading" border style="width: 100%; margin: 20px">
      <el-table-column
        prop="subjectName"
        header-align="center"
        align="center"
        label="科目"
      >
      </el-table-column>
      <el-table-column
        prop="sumScore"
        header-align="center"
        align="center"
        label="总平时分"
      >
        <template slot-scope="scope">
          <div
            style="color: #409eff; text-decoration: underline; cursor: pointer"
            @click="getDetailScore(scope.row)"
          >
            {{ scope.row.sumScore }}
          </div>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="分数详情" :visible.sync="dialogTableVisible">
      <el-table
        v-loading="dataListLoading"
        ref="singleTable"
        border
        :data="gridData"
        highlight-current-row
        style="width: 100%"
      >
        <el-table-column type="index" width="50"> </el-table-column>
        <el-table-column property="taskName" label="任务名"></el-table-column>
        <el-table-column property="score" label="分数"></el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      dataList: [],
      gridData: [],
      dataListLoading: true,
      dialogTableVisible: false,
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
    };
  },
  mounted() {
    this.getSujectAll();
  },
  methods: {
    getDetailScore(course) {
      this.dataListLoading = true;
      this.dialogTableVisible = true;
      this.$http({
        url: this.$http.adornUrl(
          "/finalexam/cardMessage/getDetailScore",
          false
        ),
        method: "post",
        params: this.$http.adornParams(
          {
            subjectName: course.subjectName,
          },
          false
        ),
      }).then(({ data }) => {
        if (data.code == 0) {
          this.gridData = data.detailScore;
        }
      });
      this.dataListLoading = false;
    },
    getSujectAll() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl(
          "/finalexam/cardMessage/getAllCourseScore",
          false
        ),
        method: "get",
      }).then(({ data }) => {
        if (data.code == 0) {
          this.dataList = data.courseScores;
        }
      });
      this.dataListLoading = false;
    },
  },
};
</script>
