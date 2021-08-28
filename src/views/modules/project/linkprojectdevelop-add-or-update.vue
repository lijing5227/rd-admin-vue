<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="项目id" prop="projectId">
      <el-input v-model="dataForm.projectId" placeholder="项目id"></el-input>
    </el-form-item>
    <el-form-item label="开发人员id" prop="developId">
      <el-input v-model="dataForm.developId" placeholder="开发人员id"></el-input>
    </el-form-item>
    <el-form-item label="" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder=""></el-input>
    </el-form-item>
    <el-form-item label="更新时间" prop="updateTime">
      <el-input v-model="dataForm.updateTime" placeholder="更新时间"></el-input>
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
          projectId: '',
          developId: '',
          createTime: '',
          updateTime: ''
        },
        dataRule: {
          projectId: [
            { required: true, message: '项目id不能为空', trigger: 'blur' }
          ],
          developId: [
            { required: true, message: '开发人员id不能为空', trigger: 'blur' }
          ],
          createTime: [
            { required: true, message: '不能为空', trigger: 'blur' }
          ],
          updateTime: [
            { required: true, message: '更新时间不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/project/linkprojectdevelop/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.projectId = data.linkProjectDevelop.projectId
                this.dataForm.developId = data.linkProjectDevelop.developId
                this.dataForm.createTime = data.linkProjectDevelop.createTime
                this.dataForm.updateTime = data.linkProjectDevelop.updateTime
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
              url: this.$http.adornUrl(`/project/linkprojectdevelop/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'projectId': this.dataForm.projectId,
                'developId': this.dataForm.developId,
                'createTime': this.dataForm.createTime,
                'updateTime': this.dataForm.updateTime
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
