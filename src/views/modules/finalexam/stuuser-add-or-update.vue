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
      <el-form-item label="专业">
        <!-- <el-input v-model="dataForm.deptName" placeholder="专业"></el-input> -->
        <el-cascader
          v-model="dataForm.categoryIds"
          :options="options"
          :props="props"
        ></el-cascader>
      </el-form-item>
      <el-form-item label="账号" prop="username">
        <el-input v-model="dataForm.username" placeholder="账号"></el-input>
      </el-form-item>
       <el-form-item label="学生名" prop="nickName">
        <el-input v-model="dataForm.nickName" placeholder="学生名"></el-input>
      </el-form-item>
      <el-form-item label="新密码" prop="newPassword">
        <el-input type="password" v-model="dataForm.newPassword"></el-input>
      </el-form-item>
      <el-form-item label="确认密码" prop="confirmPassword">
        <el-input type="password" v-model="dataForm.confirmPassword"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
      </el-form-item>
      <el-form-item label="手机号" prop="mobile">
        <el-input v-model="dataForm.mobile" placeholder="手机号"></el-input>
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <div>
          <el-radio v-model="dataForm.status" :label="0" border>禁用</el-radio>
          <el-radio v-model="dataForm.status" :label="1" border>正常</el-radio>
        </div>
      </el-form-item>
      <el-form-item label="创建时间" prop="createTime">
        <el-input v-model="dataForm.createTime" disabled></el-input>
      </el-form-item>
      <el-form-item label="修改时间" prop="updateTime">
        <el-input v-model="dataForm.updateTime" disabled></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import { mydateformat } from "@/utils";
export default {
  data() {
    var validateConfirmPassword = (rule, value, callback) => {
      if (this.dataForm.newPassword !== value) {
        callback(new Error("确认密码与新密码不一致"));
      } else {
        callback();
      }
    };
    return {
      options: [],
      props: {
        expandTrigger: "hover",
        value: "catId",
        label: "name",
        children: "children",
      },
      visible: false,
      dataForm: {
        nickName:'',
        categoryIds: [],
        userId: 0,
        deptName: "",
        username: "",
        salt: "",
        email: "",
        mobile: "",
        status: "",
        createUserId: "",
        createTime: "",
        updateTime: "",
        objStatus: "",
        newPassword: "",
        confirmPassword: "",
      },
      dataRule: {
        newPassword: [
          { required: true, message: "新密码不能为空", trigger: "blur" },
        ],
        confirmPassword: [
          { required: true, message: "确认密码不能为空", trigger: "blur" },
          { validator: validateConfirmPassword, trigger: "blur" },
        ],
        categoryIds: [
          { required: true, message: "专业名称不能为空", trigger: "blur" },
        ],
        username: [
          { required: true, message: "用户名不能为空", trigger: "blur" },
        ],
        password: [
          { required: true, message: "密码不能为空", trigger: "blur" },
        ],
        status: [
          {
            required: true,
            message: "状态  0：禁用   1：正常不能为空",
            trigger: "blur",
          },
        ],
      },
    };
  },
  created() {
    this.getDepts();
  },
  methods: {
    getDepts() {
      this.$http({
        url: this.$http.adornUrl("/finalexam/category/level/list2", false),
        method: "get",
      }).then(({ data }) => {
        this.options = data.menus;
      });
    },
    init(id) {
      this.dataForm.userId = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.userId) {
          this.$http({
            url: this.$http.adornUrl(
              `/finalexam/stuuser/info/${this.dataForm.userId}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.deptName = data.user.deptName;
              this.dataForm.username = data.user.username;
              this.dataForm.nickName=data.user.nickName;
              this.dataForm.email = data.user.email;
              this.dataForm.mobile = data.user.mobile;
              this.dataForm.status = data.user.status;
              this.dataForm.createUserId = data.user.createUserId;
              this.dataForm.createTime = mydateformat(data.user.createTime);
              if (data.user.updateTime) {
                this.dataForm.updateTime = mydateformat(data.user.updateTime);
              }
              this.dataForm.categoryIds = data.categroyIds;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          console.log(
            "生成:",
            this.dataForm.categoryIds[this.dataForm.categoryIds.length - 2]
          );
          this.$http({
            url: this.$http.adornUrl(
              `/finalexam/stuuser/${!this.dataForm.userId ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              userId: this.dataForm.userId || undefined,
              deptId: this.dataForm.categoryIds[
                this.dataForm.categoryIds.length - 1
              ],
              username: this.dataForm.username,
              password: this.dataForm.newPassword,
              email: this.dataForm.email,
              nickName:this.dataForm.nickName,
              mobile: this.dataForm.mobile,
              status: this.dataForm.status,
              createUserId: this.dataForm.createUserId,
              objStatus: this.dataForm.objStatus,
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
