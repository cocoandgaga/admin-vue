<template>
  <div>
    <div class="profile" v-loading="loading">
      <h4>专业: {{ userForm.deptName }}</h4>
      <h4>账号: {{ userForm.username }}</h4>
      <h4>学生名: {{ userForm.nickName }}</h4>
      <h4>邮箱: {{ userForm.email }}</h4>
      <h4>手机号: {{ userForm.mobile }}</h4>

      <el-button
        style="margin: 30px"
        type="danger"
        size="small"
        @click="modifiedStuInfoVisible = true"
        >修改信息</el-button
      >

      <el-button
        type="danger"
        size="small"
        @click="modifiedPasswordVisible = true"
        >修改密码</el-button
      >
      <div>
        <el-button
          style="margin: 30px"
          size="small"
          type="danger"
          v-loading="logoutLoading"
          @click="logoutHandle"
          >退出登录</el-button
        >
      </div>
    </div>
    <el-dialog title="修改信息" :visible.sync="modifiedStuInfoVisible">
      <el-form
        :model="modifyStuInfoForm"
        :rules="stuInfoRule"
        ref="modifyStuInfoForm"
      >
        <el-form-item label="邮箱" prop="email" label-width="120px">
          <el-input
            type="email"
            v-model="modifyStuInfoForm.email"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile" label-width="120px">
          <el-input
            type="phone"
            v-model="modifyStuInfoForm.mobile"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="modifiedStuInfoVisible = false">取 消</el-button>
        <el-button type="primary" @click="modifyStuInfo">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog title="修改密码" :visible.sync="modifiedPasswordVisible">
      <el-form
        :model="modifyPasswordForm"
        :rules="passwordRule"
        ref="modifyPasswordForm"
      >
        <el-form-item label="旧密码" prop="oldPassword" label-width="120px">
          <el-input
            type="password"
            v-model="modifyPasswordForm.oldPassword"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="新密码" prop="newPassword" label-width="120px">
          <el-input
            type="password"
            v-model="modifyPasswordForm.newPassword"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item
          label="确认密码"
          prop="confirmPassword"
          label-width="120px"
        >
          <el-input
            type="password"
            v-model="modifyPasswordForm.confirmPassword"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="modifiedPasswordVisible = false">取 消</el-button>
        <el-button type="primary" @click="modifyPassword">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { clearLoginInfo } from '@/utils'
export default {
  data() {
    var validateConfirmPassword = (rule, value, callback) => {
      if (this.modifyPasswordForm.newPassword !== value) {
        callback(new Error("确认密码与新密码不一致"));
      } else {
        callback();
      }
    };
    var checkPhone = (rule, value, callback) => {
      const phoneReg = /^1[3|4|5|7|8][0-9]{9}$/;
      if (!value) {
        return callback(new Error("电话号码不能为空"));
      }
      setTimeout(() => {
        // Number.isInteger是es6验证数字是否为整数的方法,实际输入的数字总是识别成字符串
        // 所以在前面加了一个+实现隐式转换

        if (!Number.isInteger(+value)) {
          callback(new Error("请输入数字值"));
        } else {
          if (phoneReg.test(value)) {
            callback();
          } else {
            callback(new Error("电话号码格式不正确"));
          }
        }
      }, 100);
    };
    var checkEmail = (rule, value, callback) => {
      const mailReg = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/;
      if (!value) {
        return callback(new Error("邮箱不能为空"));
      }
      setTimeout(() => {
        if (mailReg.test(value)) {
          callback();
        } else {
          callback(new Error("请输入正确的邮箱格式"));
        }
      }, 100);
    };

    return {
      logoutLoading:false,
      modifiedPasswordVisible: false,
      modifiedStuInfoVisible: false,
      loading: false,
      userForm: {
        id: 0,
        deptName: "",
        username: "",
        nickName: "",
        email: "",
        mobile: "",
      },
      modifyStuInfoForm: {
        email: "",
        mobile: "",
      },
      modifyPasswordForm: {
        oldPassword: "",
        newPassword: "",
        confirmPassword: "",
      },
      passwordRule: {
        oldPassword: [
          { required: true, message: "旧密码不能为空", trigger: "blur" },
        ],
        newPassword: [
          { required: true, message: "新密码不能为空", trigger: "blur" },
        ],
        confirmPassword: [
          { required: true, message: "确认密码不能为空", trigger: "blur" },
          { validator: validateConfirmPassword, trigger: "blur" },
        ],
      },
      stuInfoRule: {
        email: { validator: checkEmail, trigger: "blur" },
        mobile: { validator: checkPhone, trigger: "blur" },
      },
    };
  },
  created() {
    this.getInfo();
  },
  methods: {
    modifyStuInfo() {
      this.$refs["modifyStuInfoForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              "/finalexam/stuuser/modify-stuInfo",
              false
            ),
            method: "post",
            data: this.$http.adornData(
              {
                userId: this.userForm.id,
                mobile: this.modifyStuInfoForm.mobile,
                email: this.modifyStuInfoForm.email,
              },
              false
            ),
          }).then(({ data }) => {
            if (data.code == 0) {
              this.$message.success("修改成功");
              this.userForm.mobile = this.modifyStuInfoForm.mobile;
              this.userForm.email = this.modifyStuInfoForm.email;
              this.$refs["modifyStuInfoForm"].resetFields();
              this.modifiedStuInfoVisible = false;
            }
          });
        }
      });
    },
    // 退出
    logoutHandle() {
      this.$confirm(`确定[退出登录]?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.logoutLoading=true
          this.$http({
            url: this.$http.adornUrl("/finalexam/stuuser/logout"),
            method: "get", 
          }).then(({ data }) => {
            if (data && data.code === 0) {
              clearLoginInfo();
              this.$router.push({ name: "login" });
            }
             this.logoutLoading=false
          });
        })
        .catch(() => {});
    },
    modifyPassword() {
      this.$refs["modifyPasswordForm"].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              "/finalexam/stuuser/modify-password",
              false
            ),
            method: "post",
            data: this.$http.adornData(
              {
                userId: this.userForm.id,
                oldPassword: this.modifyPasswordForm.oldPassword,
                newPassword: this.modifyPasswordForm.newPassword,
                confirmPassword: this.modifyPasswordForm.confirmPassword,
              },
              false
            ),
          }).then(({ data }) => {
            if (data.code == 0) {
              this.$refs["modifyPasswordForm"].resetFields();
              this.$message({
                message: "修改成功",
                type: "success",
              });
              this.modifiedPasswordVisible = false;
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    },
    getInfo() {
      this.loading = true;
      this.$http({
        url: this.$http.adornUrl("/finalexam/stuuser/info", false),
        method: "get",
      }).then(({ data }) => {
        if (data.code == 0) {
          this.userForm.id = data.user.userId;
          this.userForm.deptName = data.user.deptName;
          this.userForm.username = data.user.username;
          this.userForm.nickName = data.user.nickName;
          this.userForm.email = data.user.email;
          this.userForm.mobile = data.user.mobile;
        }
        this.loading = false;
      });
    },
  },
};
</script>

<style scoped>
.profile {
  position: absolute;
  left: 6%;
  width: 90%;
  height: 80%;
  padding: 10px;
  color: red;
  background: rgba(0, 0, 0, 0) url("~@/assets/img/profile.jpg") no-repeat scroll
    100% 100%;
  background-size: auto;
  background-size: cover;
}

h4 {
  display: block;
  font-family: "Franklin Gothic Medium", "Arial Narrow", Arial, sans-serif;
  margin-block: 30px;
  margin-left: 30px;
}
</style>
