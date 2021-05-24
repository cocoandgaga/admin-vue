<template>
  <div class="mod-config">
    <el-form
      :inline="true"
      :model="dataForm"
      @keyup.enter.native="getDataList()"
    >
      <el-form-item>
        <el-input
          v-model="dataForm.key"
          placeholder="任务名"
          clearable
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('finalexam:usertask:save')"
          type="primary"
          @click="addOrUpdateHandle()"
          >新增</el-button
        >
        <el-button
          v-if="isAuth('finalexam:usertask:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
          >批量删除</el-button
        >
        <el-button
          type="primary"
          @click="exportXls(1)"
          :disabled="dataListSelections.length <= 0"
          >批量导出学生平时成绩明细表</el-button
        >
      </el-form-item>
      <el-form-item label="科目" prop="subjectNames">
        <el-select
          multiple
          v-model="dataForm.subjectNames"
          placeholder="请选择要导出的科目"
        >
          <el-option
            v-for="item in options"
            :key="item.name"
            :label="item.name"
            :value="item.name"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          @click="exportXls(2)"
          :disabled="dataForm.subjectNames.length <= 0"
          >批量导出学生平时成绩汇总表</el-button
        >
      </el-form-item>
    </el-form>

    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%"
    >
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50"
      >
      </el-table-column>
      <el-table-column
        prop="userTaskName"
        header-align="center"
        align="center"
        label="任务名"
      >
      </el-table-column>
      <el-table-column
        prop="subjectName"
        header-align="center"
        align="center"
        label="课程"
      >
      </el-table-column>
      <el-table-column
        prop="userTaskStatus"
        header-align="center"
        align="center"
        label="1 发布中 2 已截止"
      >
      </el-table-column>
      <el-table-column
        prop="objStatus"
        header-align="center"
        align="center"
        label="0 显示 1 不显示"
      >
      </el-table-column>
      <el-table-column
        prop="startTime"
        header-align="center"
        align="center"
        :formatter="dateFormat"
        label="开始时间"
      >
      </el-table-column>
      <el-table-column
        prop="endTime"
        header-align="center"
        align="center"
        :formatter="dateFormat"
        label="结束时间"
      >
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作"
      >
        <template slot-scope="scope">
          <el-button
            type="text"
            size="small"
            @click="addOrUpdateHandle(scope.row.userTaskId)"
            >修改</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="deleteHandle(scope.row.userTaskId)"
            >删除</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="getCompleteStatus(scope.row.userTaskId)"
            >任务确认汇报</el-button
          >
          <el-button
            type="text"
            size="small"
            @click="getMessage(scope.row.userTaskId,scope.row.userTaskName)"
            >留言</el-button
          >
        </template>
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
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update
      v-if="addOrUpdateVisible"
      ref="addOrUpdate"
      @refreshDataList="getDataList"
    ></add-or-update>
    <!-- 任务汇报 -->
    <el-dialog
      title="任务确认汇报"
      :visible.sync="dialogTableVisible1"
      :fullscreen="true"
    >
      <el-table
        :data="completed"
        v-loading="tableLoading"
        :show-summary="true"
        :summary-method="getCompleteSum"
        border
        style="width: 100%"
      >
        <el-table-column prop="stuId" label="学号" width="180">
        </el-table-column>
        <el-table-column prop="nickName" label="姓名" width="180">
        </el-table-column>
      </el-table>

      <el-table
        v-loading="tableLoading"
        :show-summary="true"
        :summary-method="getUnCompleteSum"
        :data="uncompleted"
        border
        style="width: 100%"
      >
        <el-table-column prop="stuId" label="学号" width="180">
        </el-table-column>
        <el-table-column prop="nickName" label="姓名" width="180">
        </el-table-column>
      </el-table>
    </el-dialog>
    <!-- 留言消息 -->
    <el-dialog
      title="任务留言"
      :visible.sync="dialogTableVisible2"
      v-loading="messageLoading"
      :fullscreen="true"
    >
      <div v-for="(cardMsg, index) in cardMessages" :key="cardMsg.id">
        <el-card>
          <span>{{ cardMsg.userName }}</span>
          于{{ mydateformat(cardMsg.createTime) }}留言:
          <p>{{ cardMsg.content }}</p>
          <el-button
            type="text"
            @click="backMsg(cardMsg.userId, cardMsg.id, index)"
            >退回</el-button
          >
        </el-card>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import moment from "moment";
import AddOrUpdate from "./usertask-add-or-update";
export default {
  data() {
    return {
      tableLoading: false,
      messageLoading: false,
      backSubmitLoading: false,
      curUserTaskId: "",
      curUserTaskName:'',
      cardMessages: [],
      completed: [],
      uncompleted: [],
      completedCnt: 0,
      uncompletedCnt: 0,
      dialogTableVisible1: false,
        dialogTableVisible2: false,
      dataForm: {
        key: "",
        subjectNames: [] 
      },
      options: [],
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
    };
  },
  components: {
    AddOrUpdate,
  },
  activated() {
    this.getDataList();
    this.getSubjectNames();
  },
  methods: {
    backMsg(userId, cardMsgId, index) { 
      this.backSubmitLoading = true;
      this.$http({
        url: this.$http.adornUrl("/finalexam/cardMessage/backMsg", false),
        method: "post",
        data: this.$http.adornData(
          {
            userTaskId: this.curUserTaskId,
            content: "你在["+this.curUserTaskName+ "]任务下发表的回复不合格,已驳回",
            receiver: userId,
            cardMsgId: cardMsgId,
          },
          false
        ),
      }).then(({ data }) => {
        this.$message.success("退回成功");
        this.dataForm.backMsg = "";
        this.cardMessages.splice(index, 1);
      });
      this.backSubmitLoading = false;
    },
    getCompleteSum(param) {
      const sums = [];
      sums[0] = "已确认任务总人数";
      sums[1] = this.completedCnt;
      return sums;
    },
    getUnCompleteSum(param) {
      const sums = [];
      sums[0] = "未确认任务总人数";
      sums[1] = this.uncompletedCnt;
      return sums;
    },
    getMessage(userTaskId,userTaskName) {
      this.curUserTaskName=userTaskName
      this.curUserTaskId = userTaskId;
      this.messageLoading = true;
      this.dialogTableVisible2 = true;
      this.$http({
        url: this.$http.adornUrl(
          `/finalexam/cardMessage/queryCardMessage`,
          false
        ),
        method: "post",
        data: this.$http.adornData(
          {
            userTaskId: userTaskId,
            type: 1,
          },
          false
        ),
      }).then(({ data }) => {
        if (data.code == 0) {
          this.cardMessages = data.cardMessages;
        }
      });
      this.messageLoading = false;
    },
    getCompleteStatus(userTaskId) {
      this.dialogTableVisible1 = true;
      this.tableLoading = true;
      this.$http({
        url: this.$http.adornUrl(
          `/finalexam/usertask/getCompleteStatus/${userTaskId}`,
          false
        ),
        method: "get",
      }).then(({ data }) => {
        if (data.code == 0) {
          this.completed = data.completed;
          this.uncompleted = data.uncompleted;
          this.uncompletedCnt = data.uncompletedCnt;
          this.completedCnt = data.completedCnt;
        }
        this.tableLoading = false;
      });
    },
    getSubjectNames() {
      this.$http({
        url: this.$http.adornUrl("/finalexam/category/level/subjects", false),
        method: "get",
      }).then(({ data }) => {
        this.options = data.subjects;
      });
    },
    dateFormat(row, column) {
      var date = row[column.property];
      if (date == undefined) {
        return "";
      }
      return moment(date).format("YYYY-MM-DD");
    },
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/finalexam/usertask/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
        }),
      }).then(({ data }) => {
        if (data && data.code === 0) {
          console.log(data.page.list);
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    exportXls(type) {
      let keys = [];
      if (type == 1) {
        keys = this.dataListSelections.map((item) => {
          return item.userTaskId;
        });
      } else if (type == 2) {
        keys = this.dataForm.subjectNames;
      }

      this.$http({
        url: this.$http.adornUrl("/finalexam/export/exportXls", false), //这里进行过封装
        method: "post",
        data: this.$http.adornData(
          {
            keys: keys,
            type: type,
          },
          false
        ),
        headers: {
          "Content-Type": "application/json; application/octet-stream",
        },
        responseType: "blob", // 服务器返回的数据类型
      })
        .then(function (res) {
          // 此处有个坑。这里用content保存文件流，最初是content=res，但下载的test.xls里的内容如下图1，
          // 检查了下才发现，后端对文件流做了一层封装，所以将content指向res.data即可
          // 另外，流的转储属于浅拷贝，所以此处的content转储仅仅是便于理解，并没有实际作用=_=
          const content = res.data;
          const blob = new Blob([content], { type: "application/excel" }); // 构造一个blob对象来处理数据
          const fileName =
            type == 1 ? "学生平时成绩明细表.xlsx" : "学生平时成绩汇总表.xlsx"; // 导出文件名
          // 对于<a>标签，只有 Firefox 和 Chrome（内核） 支持 download 属性
          // IE10以上支持blob但是依然不支持download
          if ("download" in document.createElement("a")) {
            // 支持a标签download的浏览器
            const link = document.createElement("a"); // 创建a标签
            link.download = fileName; // a标签添加属性
            link.style.display = "none";
            link.href = URL.createObjectURL(blob);
            document.body.appendChild(link);
            link.click(); // 执行下载
            URL.revokeObjectURL(link.href); // 释放url
            document.body.removeChild(link); // 释放标签
          } else {
            // 其他浏览器
            navigator.msSaveBlob(blob, fileName);
          }
        })
        .catch((error) => {
          console.log(error);
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
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map((item) => {
            return item.userTaskId;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/finalexam/usertask/delete"),
          method: "post",
          data: this.$http.adornData(ids, false),
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              },
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    },
  },
};
</script>

