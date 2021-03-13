<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="" prop="description">
      <el-input v-model="dataForm.description" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="cmtCreate">
      <el-input v-model="dataForm.cmtCreate" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="gmtModified">
      <el-input v-model="dataForm.gmtModified" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="creator">
      <el-input v-model="dataForm.creator" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="viewCnt">
      <el-input v-model="dataForm.viewCnt" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="likeCnt">
      <el-input v-model="dataForm.likeCnt" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="commentCnt">
      <el-input v-model="dataForm.commentCnt" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="tag">
      <el-input v-model="dataForm.tag" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="imageUrl">
      <el-input v-model="dataForm.imageUrl" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="videoUrl">
      <el-input v-model="dataForm.videoUrl" placeholder=""></el-input>
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
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          description: '',
          cmtCreate: '',
          gmtModified: '',
          creator: '',
          viewCnt: '',
          likeCnt: '',
          commentCnt: '',
          tag: '',
          imageUrl: '',
          videoUrl: ''
        },
        dataRule: {
          description: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          cmtCreate: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          gmtModified: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          creator: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          viewCnt: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          likeCnt: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          commentCnt: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          tag: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          imageUrl: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          videoUrl: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/finalexam/studymaterials/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.description = data.studyMaterials.description
                this.dataForm.cmtCreate = data.studyMaterials.cmtCreate
                this.dataForm.gmtModified = data.studyMaterials.gmtModified
                this.dataForm.creator = data.studyMaterials.creator
                this.dataForm.viewCnt = data.studyMaterials.viewCnt
                this.dataForm.likeCnt = data.studyMaterials.likeCnt
                this.dataForm.commentCnt = data.studyMaterials.commentCnt
                this.dataForm.tag = data.studyMaterials.tag
                this.dataForm.imageUrl = data.studyMaterials.imageUrl
                this.dataForm.videoUrl = data.studyMaterials.videoUrl
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/finalexam/studymaterials/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'description': this.dataForm.description,
                'cmtCreate': this.dataForm.cmtCreate,
                'gmtModified': this.dataForm.gmtModified,
                'creator': this.dataForm.creator,
                'viewCnt': this.dataForm.viewCnt,
                'likeCnt': this.dataForm.likeCnt,
                'commentCnt': this.dataForm.commentCnt,
                'tag': this.dataForm.tag,
                'imageUrl': this.dataForm.imageUrl,
                'videoUrl': this.dataForm.videoUrl
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
