<template>
  <div>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      style="width: 100%;margin:20px"
    >
      <el-table-column
        prop="courseName"
        header-align="center"
        align="center"
        label="课程名"
      >
      </el-table-column>
      <el-table-column
        prop="startTime"
        :formatter="dateFormat"
        header-align="center"
        align="center"
        label="考试开始时间"
      >
      </el-table-column>
      <el-table-column
        prop="endTime"
        :formatter="dateFormat"
        header-align="center"
        align="center"
        label="考试结束时间"
      >
      </el-table-column>
      <el-table-column
        prop="place"
        header-align="center"
        align="center"
        label="考试地点"
      >
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    >
    </el-pagination>
  </div>
</template>
<script>
import moment from "moment";
export default {
  created() {
    this.getDataList();
  },
  data() {
    return {
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
    };
  },
  methods: {
    dateFormat(row, column) {
      var date = row[column.property];

      if (date == undefined) {
        return "";
      }

      return moment(date).format("YYYY-MM-DD HH:mm:ss");
    },
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/finalexam/examtime/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
        }),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
  },
};
</script>
