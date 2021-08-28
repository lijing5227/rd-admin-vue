<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="100px">
    <el-form-item label="项目名称" prop="projectName">
      <el-input v-model="dataForm.projectName" placeholder="项目名称"></el-input>
    </el-form-item>
    <el-form-item label="项目级别" prop="projectLevel">
       <el-select v-model="dataForm.projectLevel" placeholder="请选择项目级别">
        <el-option label="P0" value="0"></el-option>
        <el-option label="P1" value="1"></el-option>
        <el-option label="P2" value="2"></el-option>
        <el-option label="P3" value="3"></el-option>
        <el-option label="P4" value="4"></el-option>
        <el-option label="P5" value="5"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="项目负责人" prop="projectLeader">
      <el-input v-model="dataForm.projectLeader" placeholder="项目负责人"></el-input>
    </el-form-item>
    <el-form-item label="项目状态" prop="projectState">
      <el-select v-model="dataForm.projectState" placeholder="请选择项目状态">
        <el-option label="待开发" value="0"></el-option>
        <el-option label="开发中" value="1"></el-option>
        <el-option label="开发完成" value="2"></el-option>
        <el-option label="联调" value="3"></el-option>
        <el-option label="测试" value="4"></el-option>
        <el-option label="完成" value="5"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="项目上线时间" prop="projectLanchTime">
      <el-date-picker
        v-model="dataForm.projectLanchTime"
        type="date" value-format="timestamp"
        placeholder="选择日期">
      </el-date-picker>
    </el-form-item>
    <el-form-item label="项目开发人员" prop="projectDeveloper">
      <el-input v-model="dataForm.projectDeveloper" placeholder="项目开发人员"></el-input>
    </el-form-item>
    <el-form-item label="研发负责人" prop="rdLeader">
      <el-input v-model="dataForm.rdLeader" placeholder="研发负责人"></el-input>
    </el-form-item>
    <el-form-item label="产品负责人" prop="pmLeader">
      <el-input v-model="dataForm.pmLeader" placeholder="产品负责人"></el-input>
    </el-form-item>
    <el-form-item label="开发时间">
      <el-col :span="6">
        <el-date-picker type="date" placeholder="选择开始日期" v-model="dataForm.developStartTime" value-format="timestamp"></el-date-picker>
      </el-col>
      <el-col class="line" :span="2">-</el-col>
      <el-col :span="6">
        <el-date-picker type="date" placeholder="选择结束日期" v-model="dataForm.developEndTime" value-format="timestamp"></el-date-picker>
      </el-col>
  </el-form-item>
    
    <el-form-item label="联调时间" prop="debugStartTime">
      <el-col :span="6">
        <el-date-picker type="date" placeholder="选择开始日期" v-model="dataForm.debugStartTime" value-format="timestamp"></el-date-picker>
      </el-col>
      <el-col class="line" :span="2">-</el-col>
      <el-col :span="6">
        <el-date-picker type="date" placeholder="选择结束日期" v-model="dataForm.debugEndTime" value-format="timestamp"></el-date-picker>
      </el-col>
    </el-form-item>
    <el-form-item label="测试时间" prop="testStartTime">
      <el-col :span="6">
        <el-date-picker type="date" placeholder="选择开始日期" v-model="dataForm.testStartTime" value-format="timestamp"></el-date-picker>
      </el-col>
      <el-col class="line" :span="2">-</el-col>
      <el-col :span="6">
        <el-date-picker type="date" placeholder="选择结束日期" v-model="dataForm.testEndTime" value-format="timestamp"></el-date-picker>
      </el-col>
    </el-form-item>
    <el-form-item label="项目进度" prop="projectProcess">
      <el-input v-model="dataForm.projectProcess" placeholder="项目进度"></el-input>
    </el-form-item>
    <el-form-item label="项目时间" prop="projectStartTime">
        <el-col :span="6">
        <el-date-picker type="date" placeholder="选择开始日期" v-model="dataForm.projectStartTime" value-format="timestamp"></el-date-picker>
      </el-col>
      <el-col class="line" :span="2">-</el-col>
      <el-col :span="6">
        <el-date-picker type="date" placeholder="选择结束日期" v-model="dataForm.projectEndTime" value-format="timestamp"></el-date-picker>
      </el-col>
    </el-form-item>
    <el-form-item label="备注" prop="remarks">
      <el-input v-model="dataForm.remarks" placeholder="备注"></el-input>
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
          projectName: '',
          projectLevel: '',
          projectLeader: '',
          projectState: '',
          projectLanchTime: '',
          projectDeveloper: '',
          rdLeader: '',
          pmLeader: '',
          developStartTime: '',
          developEndTime: '',
          debugStartTime: '',
          debugEndTime: '',
          testStartTime: '',
          testEndTime: '',
          projectStartTime: '',
          projectEndTime: '',
          remarks: '',
          createTime: '',
          updateTime: ''
        },
        dataRule: {
          projectName: [
            { required: true, message: '项目名称不能为空', trigger: 'blur' }
          ],
          projectLevel: [
            { required: true, message: '项目级别不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/project/project/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.projectName = data.project.projectName
                this.dataForm.projectLevel = data.project.projectLevel
                this.dataForm.projectLeader = data.project.projectLeader
                this.dataForm.projectState = data.project.projectState
                this.dataForm.projectLanchTime = data.project.projectLanchTime
                this.dataForm.projectDeveloper = data.project.projectDeveloper
                this.dataForm.rdLeader = data.project.rdLeader
                this.dataForm.pmLeader = data.project.pmLeader
                this.dataForm.developStartTime = data.project.developStartTime
                this.dataForm.developEndTime = data.project.developEndTime
                this.dataForm.debugStartTime = data.project.debugStartTime
                this.dataForm.debugEndTime = data.project.debugEndTime
                this.dataForm.testStartTime = data.project.testStartTime
                this.dataForm.testEndTime = data.project.testEndTime
                this.dataForm.projectStartTime = data.project.projectStartTime
                this.dataForm.projectEndTime = data.project.projectEndTime
                this.dataForm.projectProcess = data.project.projectProcess
                this.dataForm.remarks = data.project.remarks
                this.dataForm.createTime = data.project.createTime
                this.dataForm.updateTime = data.project.updateTime
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
              url: this.$http.adornUrl(`/project/project/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'projectName': this.dataForm.projectName,
                'projectLevel': this.dataForm.projectLevel,
                'projectLeader': this.dataForm.projectLeader,
                'projectState': this.dataForm.projectState,
                'projectLanchTime': this.dataForm.projectLanchTime,
                'projectDeveloper': this.dataForm.projectDeveloper,
                'rdLeader': this.dataForm.rdLeader,
                'pmLeader': this.dataForm.pmLeader,
                'developStartTime': this.dataForm.developStartTime,
                'developEndTime': this.dataForm.developEndTime,
                'debugStartTime': this.dataForm.debugStartTime,
                'debugEndTime': this.dataForm.debugEndTime,
                'testStartTime': this.dataForm.testStartTime,
                'testEndTime': this.dataForm.testEndTime,
                'projectStartTime': this.dataForm.projectStartTime,
                'projectEndTime': this.dataForm.projectEndTime,
                'projectProcess': this.dataForm.projectProcess,
                'remarks': this.dataForm.remarks,
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
