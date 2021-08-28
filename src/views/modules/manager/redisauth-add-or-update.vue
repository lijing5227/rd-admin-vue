<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="项目路径" prop="projectPath">
      <el-input v-model="dataForm.projectPath" placeholder="项目路径"></el-input>
    </el-form-item>
    <el-form-item label="环境" prop="environment">
      <el-input v-model="dataForm.environment" placeholder="环境"></el-input>
    </el-form-item>
    <el-form-item label="密钥" prop="secretKey">
      <el-input v-model="dataForm.secretKey" placeholder="密钥"></el-input>
    </el-form-item>
    <el-form-item label="数据库" prop="index">
      <el-input v-model="dataForm.index" placeholder="数据库"></el-input>
    </el-form-item>
    <el-form-item label="前缀" prop="prefix">
      <el-input v-model="dataForm.prefix" placeholder="前缀"></el-input>
    </el-form-item>
    <el-form-item label="是否隔离" prop="isIsolate">
      <el-input v-model="dataForm.isIsolate" placeholder="是否隔离"></el-input>
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
          projectPath: '',
          environment: '',
          secretKey: '',
          index: '',
          prefix: '',
          isIsolate: '',
          host:''
        },
        dataRule: {
          projectPath: [
            { required: true, message: '项目路径不能为空', trigger: 'blur' }
          ],
          environment: [
            { required: true, message: '环境不能为空', trigger: 'blur' }
          ],
          secretKey: [
            { required: true, message: '密钥不能为空', trigger: 'blur' }
          ],
          index: [
            { required: true, message: '数据库不能为空', trigger: 'blur' }
          ],
          prefix: [
            { required: true, message: '前缀不能为空', trigger: 'blur' }
          ],
          isIsolate: [
            { required: true, message: '是否隔离不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id,host) {
        this.dataForm.id = id || 0
        this.visible = true
        this.dataForm.host = host
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/manager/redisauth/info/${this.dataForm.id}/${this.dataForm.host}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.projectPath = data.redisAuth.projectPath
                this.dataForm.environment = data.redisAuth.environment
                this.dataForm.secretKey = data.redisAuth.secretKey
                this.dataForm.index = data.redisAuth.index
                this.dataForm.prefix = data.redisAuth.prefix
                this.dataForm.isIsolate = data.redisAuth.isIsolate
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
              url: this.$http.adornUrl(`/manager/redisauth/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'projectPath': this.dataForm.projectPath,
                'environment': this.dataForm.environment,
                'secretKey': this.dataForm.secretKey,
                'index': this.dataForm.index,
                'prefix': this.dataForm.prefix,
                'isIsolate': this.dataForm.isIsolate,
                'host':this.dataForm.host
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
