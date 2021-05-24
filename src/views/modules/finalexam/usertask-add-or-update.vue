<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="80px"
    >
      <el-form-item label="任务名" prop="userTaskName">
        <el-input
          v-model="dataForm.userTaskName"
          placeholder="请输入任务名"
        ></el-input>
      </el-form-item>
      <el-form-item label="详细描述" prop="userTaskDescription">
        <el-input
          v-model="dataForm.userTaskDescription"
          placeholder="请输入任务详细描述"
        ></el-input>
      </el-form-item>
      <el-form-item label="分数" prop="score">
        <el-input-number
          v-model="dataForm.score"
          :min="0"
          :max="10"
          :precision="2" :step="0.5"
          label="分数"
        ></el-input-number>
      </el-form-item>
      <el-form-item label="科目" prop="subjectName">
        <el-select v-model="dataForm.subjectName" placeholder="请选择">
          <el-option
            v-for="item in options"
            :key="item.name"
            :label="item.name"
            :value="item.name"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="发布状态" prop="status">
        <el-radio v-model="dataForm.status" :label="1"> 发布中</el-radio>
        <el-radio v-model="dataForm.status" :label="2">已截止</el-radio>
      </el-form-item>
      <el-form-item label="显示状态" prop="objStatus">
        <el-radio v-model="dataForm.objStatus" :label="0"> 显示</el-radio>
        <el-radio v-model="dataForm.objStatus" :label="1">不显示</el-radio>
      </el-form-item>
      <el-form-item label="开始时间" prop="startTime">
        <el-date-picker
          v-model="dataForm.startTime"
          type="date"
          placeholder="请选择开始时间"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" prop="endTime">
        <el-date-picker
          v-model="dataForm.endTime"
          type="date"
          placeholder="请选择结束时间"
        >
        </el-date-picker>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      options: [],
      dataForm: {
        objStatus: 0,
        userTaskId: 0,
        userId: "",
        userName: "",
        userTaskName: "",
        userTaskDescription: "",
        subjectName: "",
        deptId: "",
        status: 1,
        score: 0,
        startTime: "",
        endTime: "",
      },
      dataRule: {
        score: [{ required: true, message: "不能为空", trigger: "blur" }],
        userId: [{ required: true, message: "不能为空", trigger: "blur" }],
        userName: [{ required: true, message: "不能为空", trigger: "blur" }],
        userTaskName: [
          { required: true, message: "不能为空", trigger: "blur" },
        ],
        userTaskDescription: [
          { required: true, message: "不能为空", trigger: "blur" },
        ],
        subjectName: [{ required: true, message: "不能为空", trigger: "blur" }],
        deptId: [{ required: true, message: "不能为空", trigger: "blur" }],
        status: [
          { required: true, message: "发布状态不能为空", trigger: "blur" },
        ],
        startTime: [{ required: true, message: "不能为空", trigger: "blur" }],
        endTime: [{ required: true, message: "不能为空", trigger: "blur" }],
        objStatus: [
          { required: true, message: "显示状态不能为空", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.$http({
      url: this.$http.adornUrl("/finalexam/category/level/subjects", false),
      method: "get",
    }).then(({ data }) => {
      this.options = data.subjects;
    });
  },
  methods: {
    init(id) {
      this.dataForm.userTaskId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.userTaskId) {
          this.$http({
            url: this.$http.adornUrl(
              `/finalexam/usertask/info/${this.dataForm.userTaskId}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.userTaskName = data.userTask.userTaskName;
              this.dataForm.score = data.userTask.score;
              this.dataForm.userTaskDescription =
                data.userTask.userTaskDescription;
              this.dataForm.subjectName = data.userTask.subjectName;
              this.dataForm.status = data.userTask.userTaskStatus;
              this.dataForm.objStatus = data.userTask.objStatus;
              this.dataForm.startTime = data.userTask.startTime;
              this.dataForm.endTime = data.userTask.endTime;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/finalexam/usertask/${
                !this.dataForm.userTaskId ? "save" : "update"
              }`
            ),
            method: "post",
            data: this.$http.adornData({
              userTaskId: this.dataForm.userTaskId || undefined,
              userTaskName: this.dataForm.userTaskName,
              score: this.dataForm.score,
              userTaskDescription: this.dataForm.userTaskDescription,
              subjectName: this.dataForm.subjectName,
              userTaskStatus: this.dataForm.status,
              objStatus: this.dataForm.objStatus,
              startTime: this.dataForm.startTime,
              endTime: this.dataForm.endTime,
            }),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                },
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
  },
};
</script>
