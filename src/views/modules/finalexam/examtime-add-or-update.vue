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
      <el-form-item label="科目" prop="courseName">
        <el-select v-model="dataForm.courseName" placeholder="请选择">
          <el-option
            v-for="item in options"
            :key="item.name"
            :label="item.name"
            :value="item.name"
          >
          </el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="考试开始时间" prop="startTime">
        <el-date-picker
          v-model="dataForm.startTime"
          type="datetime"
          placeholder="考试开始时间"
        >
        </el-date-picker>
      </el-form-item>

      <el-form-item label="考试结束时间" prop="endTime">
        <el-date-picker
          v-model="dataForm.endTime"
          type="datetime"
          placeholder="考试结束时间"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item label="考试地点" prop="place">
        <el-input v-model="dataForm.place" placeholder="考试地点"></el-input>
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
  created() {
    this.$http({
      url: this.$http.adornUrl("/finalexam/category/level/subjects", false),
      method: "get",
    }).then(({ data }) => {
      this.options = data.subjects;
    });
  },
  data() {
    return {
      options: [],
      visible: false,
      dataForm: {
        id: 0,
        deptId: "",
        courseName: "",
        startTime: "",
        endTime:"",
        place: "",
      },
      dataRule: {
        courseName: [
          { required: true, message: "课程名不能为空", trigger: "blur" },
        ],
        startTime: [
          { required: true, message: "考试开始时间不能为空", trigger: "blur" },
        ],
        endTime: [
          { required: true, message: "考试结束时间不能为空", trigger: "blur" },
        ],
        place: [
          { required: true, message: "考试地点不能为空", trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/finalexam/examtime/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.courseName = data.examTime.courseName;
              this.dataForm.startTime = data.examTime.startTime; 
              this.dataForm.endTime = data.examTime.endTime;
              this.dataForm.place = data.examTime.place;
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
              `/finalexam/examtime/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              courseName: this.dataForm.courseName,
              startTime: this.dataForm.startTime,
              endTime: this.dataForm.endTime,
              place: this.dataForm.place,
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
