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

    <!-- 
        visible:是否显示 Dialog，支持 .sync 修饰符
        append-to-body:Dialog 自身是否插入至 body 元素上。嵌套的 Dialog 必须指定该属性并赋值为 true
        width:Dialog 的宽度 
     -->

    <el-upload
      class="upload-demo"
      ref="upload"
      action="http://localhost:90/api/finalexam/file/upload-attachment"
      :on-preview="handlePreview"
      :with-credentials="true"
      :on-remove="handleRemove"
      :file-list="fileList"
      :auto-upload="false"
      :on-success="uploadSuccess"
    >
      <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
      <el-button
        style="margin-left: 10px"
        size="small"
        type="success"
        @click="submitUpload"
        >上传</el-button
      >
    </el-upload>
  </div>
</template>

<script>
export default {
  data() {
    return {
      fileList: []
    };
  },

  methods: {
    submitUpload() {
      this.$refs.upload.submit();
    },

    uploadSuccess(response, file, fileList)	{
       console.log(response)
    },
    handleRemove(file, fileList) {
      console.log(file, fileList);
      // this.$http({
      // url:this.$http.adornUrl("/finalexam/attachment/delete",false),
      // method:'get',
      // params:this.$http.adornParms({ 
      // })
      // }).then(({data})=>{});
    },
    handlePreview(file) {
      console.log(file);
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
