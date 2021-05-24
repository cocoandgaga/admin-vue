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
        <el-input
          v-model="dataForm.title"
          placeholder="输入不超过15字的标题"
        ></el-input>
      </el-form-item>
      <el-form-item label="标签">
        <el-tag
          :key="tag"
          v-for="tag in dataForm.dynamicTags"
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
          v-model="dataForm.description"
        ></el-input>
      </el-form-item>
      <el-form-item label="创建时间">
        <el-date-picker
          format="yyyy-MM-dd HH:mm:ss"
          v-model="dataForm.gmtCreate"
          :disabled="true"
          type="date"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item label="修改时间">
        <el-date-picker
          format="yyyy-MM-dd HH:mm:ss"
          v-model="dataForm.gmtModified"
          :disabled="true"
          type="date"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item label="详细内容" prop="context">
        <mavon-editor
          v-model="dataForm.context"
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
      <el-button type="primary" @click="dataFormSubmit('dataForm')">确定</el-button>
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
      visible: false,
      inputVisible: false,
      inputValue: "",
      value: "请输入详细提问内容...",
      img_file: {},
      published: false,
      dataForm: {
        id: 0,
        description: "",
        gmtCreate: "",
        gmtModified: "",
        title: "",
        context: "",
        dynamicTags: [],
      },
      dataRule: {
        description: [{ required: true, message: "不能为空", trigger: "blur" }],
        title: [{ required: true, message: "不能为空", trigger: "blur" }],
        context: [{ required: true, message: "不能为空", trigger: "blur" }],
      },
    };
  },

  watch: {
    "dataForm.title": {
      handler(val) {
        if (val.length > 15) {
          this.dataForm.title = String(val).slice(0, 15);
          this.$message({
            message: "标题字数超过15字,已自动清除多余字数",
            type: "error",
          });
        }
      },
      deep: true,
    },
    "dataForm.description": {
      handler(val) {
        if (val.length > 100) {
          this.dataForm.description = String(val).slice(0, 100);
          this.$message({
            message: "描述超过100字,已自动清除多余字数",
            type: "error",
          });
        }
      },
      deep: true,
    },
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        this.dataForm.dynamicTags=[]
          this.dataForm.gmtCreate = '';
              this.dataForm.gmtModified = '';
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/finalexam/articles/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams(),
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.description = data.article.description;
              this.dataForm.gmtCreate = data.article.gmtCreate;
              this.dataForm.gmtModified = data.article.gmtModified;
              this.dataForm.commentCnt = data.article.commentCnt;
              this.dataForm.title = data.article.title;
              this.dataForm.dynamicTags = data.article.tag.split(",");
              this.dataForm.context = data.article.context;
            }
          });
        }
      });
    },
    dataFormSubmit(dataForm) {
      this.$refs[dataForm].validate(async (valid) => {
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
            url: this.$http.adornUrl(
              `/finalexam/articles/${!this.dataForm.id ? "save" : "update"}`,
              false
            ),
            method: "post",
            data: this.$http.adornData(
              {
                id: this.dataForm.id || undefined,
                description: this.dataForm.description,
                title: this.dataForm.title,
                tag: this.dataForm.dynamicTags.join(","),
                context: this.dataForm.context,
              },
              false
            ),
          })
            .then(({ data }) => {
              this.published = false;

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
            })
            .catch(() => {});
        } else {
          return false;
        }
      });
    },

    handleClose(tag) {
      this.dataForm.dynamicTags.splice(
        this.dataForm.dynamicTags.indexOf(tag),
        1
      );
    },
    cancle() {
      this.$refs.dataForm.resetFields();
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
        this.dataForm.dynamicTags.push(inputValue);
      }
      this.inputVisible = false;
      this.inputValue = "";
    },

    // 绑定@imgAdd event
    $imgAdd(pos, $file) {
      this.img_file[pos] = $file;
    },

    $imgDel(pos, $file) {
      delete this.img_file[pos];
    },
  },
};
</script>
