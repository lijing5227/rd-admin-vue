<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="网站名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="网站名称"></el-input>
    </el-form-item>
    <el-form-item label="网站链接" prop="link">
      <el-input v-model="dataForm.link" placeholder="网站链接"></el-input>
    </el-form-item>
    <el-form-item label="环境" prop="env">
      <el-select v-model="dataForm.env" placeholder="环境">
        <el-option label="开发环境" value="dev"></el-option>
        <el-option label="测试-dev环境" value="docker-dev"></el-option>
        <el-option label="测试-hotfix环境" value="docker-hotfix"></el-option>
        <el-option label="测试-stress环境" value="docker-stress"></el-option>
        <el-option label="生产环境" value="prod"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="排序" prop="sort">
      <el-input v-model="dataForm.sort" placeholder="排序"></el-input>
    </el-form-item>
    <el-form-item label="备注" prop="remark">
      <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
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
          name: '',
          link: '',
          env: '',
          sort: '',
          remark:''
        },
        dataRule: {
          name: [
            { required: true, message: '网站名称不能为空', trigger: 'blur' }
          ],
          link: [
            { required: true, message: '网站链接不能为空', trigger: 'blur' }
          ],
          env: [
            { required: true, message: '环境不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/nav/navmap/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.navMap.name
                this.dataForm.link = data.navMap.link
                this.dataForm.env = data.navMap.env
                this.dataForm.sort = data.navMap.sort
                this.dataForm.remark = data.navMap.remark
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
              url: this.$http.adornUrl(`/nav/navmap/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'link': this.dataForm.link,
                'env': this.dataForm.env,
                'sort': this.dataForm.sort,
                'remark':this.dataForm.remark
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
