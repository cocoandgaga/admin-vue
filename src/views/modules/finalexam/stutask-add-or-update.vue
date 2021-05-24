<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="所属老师发布的任务" prop="userTaskId">
      <el-input v-model="dataForm.userTaskId" placeholder="所属老师发布的任务"></el-input>
    </el-form-item>
    <el-form-item label="" prop="stuId">
      <el-input v-model="dataForm.stuId" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="stuName">
      <el-input v-model="dataForm.stuName" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="stuTaskName">
      <el-input v-model="dataForm.stuTaskName" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="stuTaskDescription">
      <el-input v-model="dataForm.stuTaskDescription" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="deptId">
      <el-input v-model="dataForm.deptId" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="startTime">
      <el-input v-model="dataForm.startTime" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="endTime">
      <el-input v-model="dataForm.endTime" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="1待确认 2进行中 3完成" prop="status">
      <el-input v-model="dataForm.status" placeholder="1待确认 2进行中 3完成"></el-input>
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
          taskId: 0,
          userTaskId: '',
          stuId: '',
          stuName: '',
          stuTaskName: '',
          stuTaskDescription: '',
          deptId: '',
          startTime: '',
          endTime: '',
          status: ''
        },
        dataRule: {
          userTaskId: [
            { required: true, message: '所属老师发布的任务不能为空', trigger: 'blur' }
          ],
          stuId: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          stuName: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          stuTaskName: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          stuTaskDescription: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          deptId: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          startTime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          endTime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          status: [
            { required: true, message: '1待确认 2进行中 3完成不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.taskId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.taskId) {
            this.$http({
              url: this.$http.adornUrl(`/finalexam/stutask/info/${this.dataForm.taskId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userTaskId = data.stuTask.userTaskId
                this.dataForm.stuId = data.stuTask.stuId
                this.dataForm.stuName = data.stuTask.stuName
                this.dataForm.stuTaskName = data.stuTask.stuTaskName
                this.dataForm.stuTaskDescription = data.stuTask.stuTaskDescription
                this.dataForm.deptId = data.stuTask.deptId
                this.dataForm.startTime = data.stuTask.startTime
                this.dataForm.endTime = data.stuTask.endTime
                this.dataForm.status = data.stuTask.status
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
              url: this.$http.adornUrl(`/finalexam/stutask/${!this.dataForm.taskId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'taskId': this.dataForm.taskId || undefined,
                'userTaskId': this.dataForm.userTaskId,
                'stuId': this.dataForm.stuId,
                'stuName': this.dataForm.stuName,
                'stuTaskName': this.dataForm.stuTaskName,
                'stuTaskDescription': this.dataForm.stuTaskDescription,
                'deptId': this.dataForm.deptId,
                'startTime': this.dataForm.startTime,
                'endTime': this.dataForm.endTime,
                'status': this.dataForm.status
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
