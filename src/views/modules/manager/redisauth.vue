<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
    <el-form-item>
        <el-input v-model="dataForm.host" placeholder="点击获取连接或者输入" clearable></el-input>
    </el-form-item>
    <el-dropdown @command="handleCommand">
      <el-button type="primary">
        选择环境<i class="el-icon-arrow-down el-icon--right"></i>
      </el-button>
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item  v-for = "item in hostList.list" :key=item.id v-bind:command="item">{{ item }} </el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('manager:redisauth:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
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
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="序号">
      </el-table-column>
      <el-table-column
        prop="projectPath"
        header-align="center"
        align="center"
        label="项目路径">
      </el-table-column>
      <el-table-column
        prop="environment"
        header-align="center"
        align="center"
        label="环境">
      </el-table-column>
      <el-table-column
        prop="secretKey"
        header-align="center"
        align="center"
        label="密钥">
      </el-table-column>
      <el-table-column
        prop="index"
        header-align="center"
        align="center"
        label="数据库">
      </el-table-column>
      <el-table-column
        prop="prefix"
        header-align="center"
        align="center"
        label="前缀">
      </el-table-column>
      <el-table-column
        prop="isIsolate"
        header-align="center"
        align="center"
        label="是否隔离">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
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
  import AddOrUpdate from './redisauth-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: '',
          host: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        hostList:[]
      }
    },
    
    components: {
      AddOrUpdate
    },
    created(){
        this.getHostList();
    },
    activated () {
      this.getDataList()
    },

    methods: {
      handleCommand(command) {
        this.dataForm.host = command;
        this.getDataList();
      },
      getHostList(){
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/redis/env/list'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.hostList = data
          } else {
            this.hostList = []
          }
        })
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/manager/redisauth/list'),
          method: 'post',
          data: this.$http.adornData({
            'currentPage': this.pageIndex,
            'pageSize': this.pageSize,
            'projectPath' : this.dataForm.key,
            'host' : this.dataForm.host
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
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id,this.dataForm.host)
        })
      },
      // 删除
      deleteHandle (row) {
        var ids = row.id ? [row.id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${row.id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/manager/redisauth/deleteByParam'),
            method: 'post',
            data: this.$http.adornData({"projectPath":row.projectPath,"environment":row.environment,"host":this.dataForm.host})
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
