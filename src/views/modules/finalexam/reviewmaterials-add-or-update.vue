<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="" prop="imageUrl">
      <el-input v-model="dataForm.imageUrl" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="description">
      <el-input v-model="dataForm.description" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="gmtCreate">
      <el-input v-model="dataForm.gmtCreate" placeholder=""></el-input>
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
          imageUrl: '',
          description: '',
          gmtCreate: '',
          gmtModified: '',
          creator: '',
          viewCnt: '',
          likeCnt: '',
          commentCnt: '',
          tag: '',
          videoUrl: ''
        },
        dataRule: {
          imageUrl: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          description: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          gmtCreate: [
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
              url: this.$http.adornUrl(`/finalexam/reviewmaterials/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.imageUrl = data.reviewMaterials.imageUrl
                this.dataForm.description = data.reviewMaterials.description
                this.dataForm.gmtCreate = data.reviewMaterials.gmtCreate
                this.dataForm.gmtModified = data.reviewMaterials.gmtModified
                this.dataForm.creator = data.reviewMaterials.creator
                this.dataForm.viewCnt = data.reviewMaterials.viewCnt
                this.dataForm.likeCnt = data.reviewMaterials.likeCnt
                this.dataForm.commentCnt = data.reviewMaterials.commentCnt
                this.dataForm.tag = data.reviewMaterials.tag
                this.dataForm.videoUrl = data.reviewMaterials.videoUrl
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
              url: this.$http.adornUrl(`/finalexam/reviewmaterials/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'imageUrl': this.dataForm.imageUrl,
                'description': this.dataForm.description,
                'gmtCreate': this.dataForm.gmtCreate,
                'gmtModified': this.dataForm.gmtModified,
                'creator': this.dataForm.creator,
                'viewCnt': this.dataForm.viewCnt,
                'likeCnt': this.dataForm.likeCnt,
                'commentCnt': this.dataForm.commentCnt,
                'tag': this.dataForm.tag,
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
