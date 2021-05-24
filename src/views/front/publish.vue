<template>
  <div >
    <el-form
      :model="quesionForm"
      :rules="rules"
      ref="quesionForm"
      label-width="100px"
      style="margin:10px"
    >
      <el-form-item label="标题" prop="title">
        <el-input
          v-model="quesionForm.title"
          placeholder="输入不超过15字的标题"
        ></el-input>
      </el-form-item>
      <el-form-item label="相关" prop="type">
        <el-radio-group v-model="quesionForm.type">
          <el-radio-button :label="0">考试相关</el-radio-button>
          <el-radio-button :label="1">学习相关</el-radio-button>
          <el-radio-button :label="2">其他相关</el-radio-button>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="标签">
        <el-tag
          :key="tag"
          v-for="tag in quesionForm.dynamicTags"
          closable
          :disable-transitions="false"
          @close="handleClose(tag)"
        >
          {{ tag }}
        </el-tag>
        <el-input
          class="input-new-tag"
          v-if="inputVisible"
          v-model="inputValue"
          ref="saveTagInput"
          size="small"
          @keyup.enter.native="handleInputConfirm"
          @blur="handleInputConfirm"
        >
        </el-input>
        <el-button v-else class="button-new-tag" size="small" @click="showInput"
          >+ New Tag</el-button
        >
      </el-form-item>

      <el-form-item label="简短描述" prop="description">
        <el-input
          type="textarea"
          rows="4"
          v-model="quesionForm.description"
        ></el-input>
      </el-form-item>

      <el-form-item label="提问内容" prop="content">
        <mavon-editor
          v-model="quesionForm.content"
          :ishljs="true"
          :scrollStyle="true"
          :inshljs="true"
          ref="md"
          @imgAdd="$imgAdd"
          @imgDel="$imgDel"
          class="body"
        ></mavon-editor>
      </el-form-item>

      <el-form-item>
        <el-button
          type="primary"
          @click="uploadimg('quesionForm')"
          :loading="published"
          >发布</el-button
        >
        <el-button type="danger" @click="cancle()">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
import axios from "axios";
import { mavonEditor } from "mavon-editor";
import "mavon-editor/dist/css/index.css";
export default {
  components: { mavonEditor },
  data() {
    return {
      exceed: "no",
      inputVisible: false,
      inputValue: "",
      value: "请输入详细提问内容...",
      img_file: {},
      published: false,
      quesionForm: {
        dynamicTags: [],
        id: 0,
        type: 2,
        title: "",
        description: "",
        content: "",
        tag: "",
      },
      rules: {
        title: [{ required: true, message: "标题不能为空", trigger: "blur" }],
        description: [
          { required: true, message: "简短描述不能为空", trigger: "blur" },
        ],
      },
    };
  },
  watch: {
    "quesionForm.title": {
      handler(val) {
        if (val.length > 15) {
          this.quesionForm.title = String(val).slice(0, 15);
          this.$message({
            message: "标题字数超过15字,已自动清除多余字数",
            type: "error",
          });
        }
      },
      deep: true,
    },
    "quesionForm.description": {
      handler(val) {
        if (val.length > 140) {
          this.quesionForm.description = String(val).slice(0, 140);
          this.$message({
            message: "描述超过140字,已自动清除多余字数",
            type: "error",
          });
        }
      },
      deep: true,
    },
  },
  methods: {
    cancle(){
      this.$refs.quesionForm.resetFields();
    },
    handleClose(tag) {
      this.quesionForm.dynamicTags.splice(
        this.quesionForm.dynamicTags.indexOf(tag),
        1
      );
    },

    showInput() {
      this.inputVisible = true;
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },

    handleInputConfirm() {
      let inputValue = this.inputValue;
      if (inputValue) {
        this.quesionForm.dynamicTags.push(inputValue);
      }
      this.inputVisible = false;
      this.inputValue = "";
    },

    // 绑定@imgAdd event
    $imgAdd(pos, $file) {
      this.img_file[pos] = $file;
    },

    uploadimg(quesionForm) {
      this.$refs[quesionForm].validate(async (valid) => {
        if (valid) {
          this.published = true;
          // 第一步.将图片上传到服务器.
          if (JSON.stringify(this.img_file) != "{}" && this.img_file != null) {
            var formdata = new FormData();
            for (var _img in this.img_file) {
              formdata.append(_img, this.img_file[_img]);
            }
            await axios({
              url: this.$http.adornUrl("/finalexam/file/upload", false),
              method: "post",
              data: formdata,
              headers: { "Content-Type": "multipart/form-data" },
            })
              .then(({ data }) => {
                this.published = false;
                if (data && data.code === 0) {
                  var arr = data.urls;
                  if (data != null && data.code === 0) {
                    for (let index = 0; index < arr.length; index++) {
                      const elem = arr[index];
                      this.$refs.md.$img2Url(index + 1, elem);
                    }
                  } else {
                    this.$message.error(data.msg);
                  }
                }
              })
              .catch(() => {});
          }
          //第二步 上传表单
          this.$http({
            url: this.$http.adornUrl("/finalexam/question/save", false),
            method: "post",
            data: this.$http.adornData(
              {
                id: this.quesionForm.id || undefined,
                title: this.quesionForm.title,
                type: this.quesionForm.type,
                tags: this.quesionForm.dynamicTags.join(","),
                description: this.quesionForm.description,
                content: this.quesionForm.content,
              },
              false
            ),
          })
            .then(({ data }) => {
              this.published = false;
              if (data != null && data.code === 0) {
                this.$message.success("发布成功") 
                this.$refs.quesionForm.resetFields()
              } else {
                this.$message({
                  message: `发布失败,${data.msg}`,
                  type: "error",
                });
              }
            })
            .catch(() => {});
        } else {
          return false;
        }
      });
    },

    $imgDel(pos, $file) {
      delete this.img_file[pos];
    },
  },
};
</script>

<style scoped >
.body {
  -webkit-text-size-adjust: none;
}

.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>