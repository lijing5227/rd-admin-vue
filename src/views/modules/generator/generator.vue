<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
     <el-select v-model="q.dsName" placeholder="请选择数据源" @change="handleCommand">
              <el-option
                v-for="item in dataSourceList"
                :key="item.id"
                :label="item.name"
                :value="item.name"/>
      </el-select>
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="表名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('sys:generator:code')" type="primary" @click="initForm(),dialogFormVisible = true" :disabled="dataListSelections.length <= 0">批量生成</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      </el-table-column>
      <el-table-column
        prop="tableName"
        header-align="center"
        align="center"
        label="表名">
      </el-table-column>
      <el-table-column
        prop="ENGINE"
        header-align="center"
        align="center"
        label="引擎">
      </el-table-column>
      <el-table-column
        prop="tableComment"
        header-align="center"
        align="center"
        label="表备注">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:generator:code')" type="text" size="small" @click="initForm(scope.row.tableName),dialogFormVisible = true">生成</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="生成配置" :visible.sync="dialogFormVisible">
    <el-form :model="form">
    <el-form-item label="表名称" :label-width="formLabelWidth">
      <el-input v-model="form.tableName" placeholder="" autocomplete="off" disabled=disabled></el-input>
    </el-form-item>
    <el-form-item label="表前缀" :label-width="formLabelWidth">
      <el-input v-model="form.tablePrefix" placeholder="可为空，加载系统默认配置" autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item label="包名" :label-width="formLabelWidth">
      <el-input v-model="form.packageName" placeholder="可为空，加载系统默认配置" autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item label="作者" :label-width="formLabelWidth">
      <el-input v-model="form.author" placeholder="可为空，加载系统默认配置" autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item label="模块" :label-width="formLabelWidth">
      <el-input v-model="form.moduleName" placeholder="可为空，加载系统默认配置" autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item label="注释" :label-width="formLabelWidth">
      <el-input v-model="form.comments" placeholder="可为空，加载表备注" autocomplete="off"></el-input>
    </el-form-item>
  </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button @click="dialogFormVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogFormVisible = false,submitForm()">确 定</el-button>
  </div>
</el-dialog>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './generator-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dsName:'',
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        dataSourceList:[],
        env:'',
        q: {},
        dialogFormVisible: false,
        form: {
          tableName: '',
          packageName: '',
          author: '',
          moduleName: '',
          comments: '',
          dsName:''
        },
        formLabelWidth: '60px'
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    created(){
        this.getDataSourceList();
    },
    methods: {
      initForm(tableName){
         var tableNames = tableName ? [tableName] : this.dataListSelections.map(item => {
          return item.tableName
        })
        var finalTables = tableNames.join(',')
        this.form.tableName = finalTables
      },
      submitForm(){
        const formData = JSON.stringify(this.form)
        return this.generator(formData)
      },
      generator(table) {
        return  this.$http({
          url: this.$http.adornUrl('/generator/code'),
          method: 'post',
          data: table,
          responseType: 'arraybuffer'
        }).then((response) => { 
          const blob = new Blob([response.data], {type: 'application/zip'})
          const filename =  'code.zip'
          const link = document.createElement('a')
          link.href = URL.createObjectURL(blob)
          link.download = filename
          document.body.appendChild(link)
          link.click()
          window.setTimeout(function () {
            URL.revokeObjectURL(blob)
            document.body.removeChild(link)
          }, 0)
        })
      },
      handleCommand(command) {
        this.form.dsName = command
        this.dsName = command
        this.getDataList();
      },
      deleteHandle(row){
        alert("开发中")
      },
      getDataSourceList(){
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/ds/datasource/listAll'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataSourceList = data.data
          } else {
            this.dataSourceList = []
          }
        })
      },
      // 获取数据列表
      getDataList () {
        const dataSource = this.dsName;
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/generator/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'tableName': this.dataForm.key,
            'dsName':dataSource
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
    // 下载文件
    download (data) {
        if (!data) {
            return
        }
        const fileName = "code.zip"
        //const blob = new Blob([data,{type: "application/octet-stream;charset=utf-8"}])
        const blob = new Blob([data,{type: "application/x-zip-compressed;charset=utf-8"}])
        let url = window.URL.createObjectURL(blob)
        if("download" in document.createElement('a')){
        let link = document.createElement('a')
        link.download=fileName
        link.style.display = 'none'
        link.href = url
        document.body.appendChild(link)
        link.click()
        window.URL.revokeObjectURL(link)
        document.body.removeChild(link)
        }else{
          navigator.msSaveBlob(blob,fileName)
        }
    }
    }
  }
</script>
