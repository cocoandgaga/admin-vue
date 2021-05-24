<template>
  <div>
    <!-- 
           action:必选参数,上传的地址
           list-type:文件列表的类型  text/picture/picture-card
           on-success:文件上传成功时的钩子  function(file, fileList)
           file-list:上传的文件列表,例如: [{name: 'food.jpg', url: 'https://xxx.cdn.com/xxx.jpg'}]
           on-progress:文件上传时的钩子 function(event, file, fileList)
           data:上传时附带的额外参数  
           accept:接受上传的文件类型（thumbnail-mode 模式下此参数无效）
           headers:设置上传的请求头部
           disabled:是否禁用

            accept="video/mp4, video/ogg, video/flv,video/avi,video/wmv,video/rmvb"
         -->
    <el-form :model="ruleForm" label-width="100px">
      <el-form-item label="视频标题">
        <el-input v-model="ruleForm.title"></el-input>
      </el-form-item>
      <el-form-item label="视频描述">
        <el-input type="textarea" rows="5" v-model="ruleForm.desc"></el-input>
      </el-form-item>
      <el-form-item label="添加视频">
        <el-upload
          :action="action"
          ref="upload"
          :auto-upload="false"
          list-type="text"
          :on-success="handleSuccess"
          :file-list="fileLists"
          :on-progress="handleProgress"
          :before-upload="beforeUpload"
          :data="upData"
          :disabled="fileLists.length >= limit || uploadBtn"
        >
          <!-- 
                使用 scoped-slot 去设置缩略图模版。 
            -->
          <i slot="default" class="el-icon-plus"></i>
          <div slot="file" slot-scope="{ file }"> 
              <span
                class="el-upload-list__item-delete"
                @click="handleRemove(file)"
              >
                <i class="el-icon-delete"></i>
              </span>
            </span>
            <!--
            type:进度条类型 line/circle/dashboard
            color:进度条背景色（会覆盖 status 状态颜色）
            percentage:百分比 0-100
             -->
            <el-progress
              type="circle"
              class="progressModule"
              :color="colors"
              :percentage="Number(uploadPercentage)"
              v-if="showProgress && file.url == uploadUrl"
            >
            </el-progress>
          </div>
        </el-upload>
      </el-form-item>
      <el-form-item>
        <el-button
          style="margin-left: 10px"
          size="small"
          type="success"
          @click="submitUpload"
          >发布</el-button
        >
      </el-form-item>
    </el-form>
    <!-- 
        visible:是否显示 Dialog，支持 .sync 修饰符
        append-to-body:Dialog 自身是否插入至 body 元素上。嵌套的 Dialog 必须指定该属性并赋值为 true
        width:Dialog 的宽度 
     -->

    <el-dialog
      :visible.sync="dialogVisible"
      append-to-body
      :before-close="handleClose"
      width="40%"
      style="text-align: center"
    >
      <video
        :src="dialogImageUrl"
        alt=""
        id="my_video"
        autoplay
        class="video"
        controls="controls"
      ></video>
    </el-dialog>

    <el-dialog :visible.sync="editView" width="40%" append-to-body>
      <el-input
        type="textarea"
        :rows="4"
        v-model="editForm.url"
        @input="editVideo"
      ></el-input>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "uploadVideo",
  props: {
    limit: {
      type: Number,
      default: 12,
    },
    action: {
      type: String,
      default: "http://localhost:90/api/finalexam/file/upload",
    },
    fileList: {
      type: Array,
      default: () => {
        return [];
      },
    },
    data: {
      type: Object,
      default: () => {
        return {};
      },
    },
  },
  data() {
    return {
      ruleForm: {
        title: "",
        desc: "",
      },
      autoUpload: false,
      dialogImageUrl: "",
      dialogVisible: false,
      fileLists: this.fileList,
      editForm: {
        url: "",
        uid: null,
      },
      editView: false,
      uploadPercentage: 0,
      showProgress: false,
      uploadUrl: "",
      colors: [
        { color: "#ADD8E6", percentage: 20 },
        { color: "#87CEEB", percentage: 40 },
        { color: "#87CEFA", percentage: 60 },
        { color: "#00BFFF", percentage: 80 },
        { color: "#1296DB", percentage: 100 },
      ],
      uploadBtn: false,
    };
  },
  computed: {
    // 这里定义上传文件时携带的参数，即表单数据
    upData: function () {
      return {
        title: this.ruleForm.title,
        desc: this.ruleForm.desc,
      };
    },
  },
  methods: {
    submitUpload() {
      this.$refs.upload.submit();
    },
    // 移除视频
    handleRemove(file) {
      for (let i in this.fileLists) {
        if (this.fileLists[i].uid == file.uid) {
          this.fileLists.splice(i, 1);
          this.uploadBtn = false;
        }
      }
      this.submitFile();
    }, 
    handleClose(done) {
      document.getElementById("my_video").pause();
      done();
    }, 

    // 上传完成
    handleSuccess(response, file, fileList) {
      this.showProgress = false;
      // this.uploadBtn = true;
      console.log("==fileList==", fileList);
      console.log("==file==", file);  
      for (var i = 0; i < fileList.length; i++) {
        console.log("---------------")
        var arr = fileList[i].response.urls;
        for (let index = 0; index < arr.length; index++) {
          const url = arr[index];
          let obj = {
            uid: fileList[i].uid,
            status: "success",
            name: fileList[i].name,
            url: url,
          };
          this.fileLists.push(obj);
           console.log("file111:",fileLists)
        }
        this.$message.success(response.msg);
      }

      console.log(fileLists)
      this.submitFile();
    },
    // 将图片文件传回给父组件
    submitFile() {
      this.$emit("submitImg", this.fileLists);
    },
    // 上传进度
    handleProgress(response, file, fileList) {
      // this.uploadBtn = true;
      this.uploadUrl = file.url;
      this.showProgress = true;
      this.uploadPercentage = file.percentage.toFixed(0);
    },
  },
};
</script>

<style scoped>
.el-icon-plus {
  font-size: 30px !important;
}
.el-icon-edit {
  font-size: 18px !important;
}
.el-icon-video-play {
  font-size: 18px !important;
}
.el-icon-delete {
  font-size: 18px !important;
  color: rgb(243, 143, 130);
}
.el-input .el-textarea__inner {
  font-size: 18px !important;
}
.video {
  min-height: 200px;
  max-height: 600px;
  min-width: 200px;
  max-width: 100%;
}
.progressModule .el-progress__text {
  color: #1296db;
  font-size: 15px !important;
}
</style>
