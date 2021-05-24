<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
    :fullscreen="true"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm" 
      label-width="80px"
    >
      <el-form-item label="标题" prop="title">
        <el-input v-model="dataForm.title" placeholder="标题"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="description">
        <el-input v-model="dataForm.description" placeholder="描述"></el-input>
      </el-form-item>
      <el-form-item label="标签" prop="tags">
        <el-input
          v-model="dataForm.tags"
          placeholder="标签用英文逗号分隔(tag1,tag2,...)"
        ></el-input>
      </el-form-item>
      <el-form-item label="创建时间" prop="gmtCreate">
        <el-input v-model="dataForm.gmtCreate" disabled></el-input>
      </el-form-item>
      <el-form-item label="修改时间" prop="gmtModified">
        <el-input v-model="dataForm.gmtModified" disabled></el-input>
      </el-form-item>

      <el-form-item label="内容">
        <mavon-editor
          v-model="dataForm.content"
          :ishljs="true"
          :scrollStyle="true"
          :inshljs="true"
          ref="md"
          @imgAdd="$imgAdd"
          @imgDel="$imgDel"
          class="body"
        ></mavon-editor>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import axios from "axios";
import { mavonEditor } from "mavon-editor";
import "mavon-editor/dist/css/index.css";
export default {
  components: { mavonEditor },
  data() {
    return {
      img_file: {},
      visible: false,
      dataForm: {
        id: 0,
        title: "",
        description: "",
        tags: "",
        gmtCreate: "",
        gmtModified: "",
        content: "",
      },
      dataRule: {
        title: [{ required: true, message: "不能为空", trigger: "blur" }],
        description: [{ required: true, message: "不能为空", trigger: "blur" }],
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
              `/finalexam/question/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.title = data.question.title;
              this.dataForm.description = data.question.description;
              this.dataForm.tags = data.question.tags;
              this.dataForm.gmtCreate = mydateformat(data.question.gmtCreate);
              this.dataForm.gmtModified = mydateformat(data.question.gmtModified);
              this.dataForm.content = data.question.content;
            }
          });
        }
      });
    },
    $imgAdd(pos, $file) {
      this.img_file[pos] = $file;
    },

    uploadimg() {
      if (JSON.stringify(this.img_file)!='{}'&&this.img_file!=null) {
        // 第一步.将图片上传到服务器.
        var formdata = new FormData();
        for (var _img in this.img_file) {
          formdata.append(_img, this.img_file[_img]);
        }
        axios({
          url: this.$http.adornUrl("/finalexam/file/upload", false),
          method: "post",
          data: formdata,
          headers: { "Content-Type": "multipart/form-data" },
        }).then(({ data }) => {
          this.published = false;
          var arr = data.urls;
          if (data != null && data.code === 0) {
            for (let index = 0; index < arr.length; index++) {
              const elem = arr[index];
              this.$refs.md.$img2Url(index + 1, elem);
            }
          } else {
            this.$message.error(data.msg);
          }
        });
      }
    },

    $imgDel(pos, $file) {
      delete this.img_file[pos];
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          this.uploadimg();
          this.$http({
            url: this.$http.adornUrl(
              `/finalexam/question/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              title: this.dataForm.title,
              description: this.dataForm.description,
              tags: this.dataForm.tags,
              content:this.dataForm.content==null?"":this.dataForm.content
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
