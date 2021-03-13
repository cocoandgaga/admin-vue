<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="" prop="descp">
      <el-input v-model="dataForm.descp" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="gmtCreate">
      <el-input v-model="dataForm.gmtCreate" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="gmtModified">
      <el-input v-model="dataForm.gmtModified" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="startTime">
      <el-input v-model="dataForm.startTime" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="endTime">
      <el-input v-model="dataForm.endTime" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="0未完成 1待办 2正在完成 3已完成" prop="isCompleted">
      <el-input v-model="dataForm.isCompleted" placeholder="0未完成 1待办 2正在完成 3已完成"></el-input>
    </el-form-item>
    <el-form-item label="" prop="subjectName">
      <el-input v-model="dataForm.subjectName" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="" prop="teacherName">
      <el-input v-model="dataForm.teacherName" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="0老师计划 1学生计划" prop="type">
      <el-input v-model="dataForm.type" placeholder="0老师计划 1学生计划"></el-input>
    </el-form-item>
    <el-form-item label="完成数目" prop="completeNum">
      <el-input v-model="dataForm.completeNum" placeholder="完成数目"></el-input>
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
          descp: '',
          gmtCreate: '',
          gmtModified: '',
          startTime: '',
          endTime: '',
          isCompleted: '',
          subjectName: '',
          teacherName: '',
          type: '',
          completeNum: ''
        },
        dataRule: {
          descp: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          gmtCreate: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          gmtModified: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          startTime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          endTime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          isCompleted: [
            { required: true, message: '0未完成 1待办 2正在完成 3已完成不能为空', trigger: 'blur' }
          ],
          subjectName: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          teacherName: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '0老师计划 1学生计划不能为空', trigger: 'blur' }
          ],
          completeNum: [
            { required: true, message: '完成数目不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/finalexam/todolist/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.descp = data.toDoList.descp
                this.dataForm.gmtCreate = data.toDoList.gmtCreate
                this.dataForm.gmtModified = data.toDoList.gmtModified
                this.dataForm.startTime = data.toDoList.startTime
                this.dataForm.endTime = data.toDoList.endTime
                this.dataForm.isCompleted = data.toDoList.isCompleted
                this.dataForm.subjectName = data.toDoList.subjectName
                this.dataForm.teacherName = data.toDoList.teacherName
                this.dataForm.type = data.toDoList.type
                this.dataForm.completeNum = data.toDoList.completeNum
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
              url: this.$http.adornUrl(`/finalexam/todolist/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'descp': this.dataForm.descp,
                'gmtCreate': this.dataForm.gmtCreate,
                'gmtModified': this.dataForm.gmtModified,
                'startTime': this.dataForm.startTime,
                'endTime': this.dataForm.endTime,
                'isCompleted': this.dataForm.isCompleted,
                'subjectName': this.dataForm.subjectName,
                'teacherName': this.dataForm.teacherName,
                'type': this.dataForm.type,
                'completeNum': this.dataForm.completeNum
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
