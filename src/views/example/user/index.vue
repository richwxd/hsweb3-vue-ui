<!-- 基础信息 - 品牌 -->
<template>
<div id="user">
  <search-bar :data="searchBarData" @search="searchAction" @add="showAdd()" @command="clickMoreCommand"></search-bar>
  <table-contain :height.sync="table.maxHeight">
    <el-table :data="table.data" slot="table" :size="table.size" :max-height="table.maxHeight" style="width: 100%;">
      <el-table-column label="序号" width="50" align="center">
        <template slot-scope="scope">
          <span>{{scope.$index + 1}}</span>
        </template>
      </el-table-column>
      <el-table-column prop="name" label="名称" align="center"></el-table-column>
      <el-table-column prop="code" label="编码" align="center"></el-table-column>
      <el-table-column prop="title" label="标题" align="center"></el-table-column>
      <el-table-column prop="updateUser" label="最后修改人" align="center"></el-table-column>
      <el-table-column prop="enable" label="状态" align="center">
        <template slot-scope="scope">
          <template v-if="scope.row.state == 0">
            <el-tag size="mini" type="info">禁用</el-tag>
          </template>
          <template v-else>
            <el-tag size="mini">启用</el-tag>
          </template>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="180">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="clickEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button type="danger" size="mini" @click="clickDelete(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      slot="footer"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagination.page"
      :page-sizes="pagination.pageSizes"
      :page-size="pagination.size"
      layout="total, sizes, prev, pager, next"
      :total="pagination.total">
    </el-pagination>
  </table-contain> 
  <add v-if="add.visiable" v-model="add.visiable" :data="add.data" @add="handleCurrentChange(1)" @edit="fetchData"></add>
</div>
</template>
<script>
import { fetch, deleteId } from '@/api/index.js'
import model from '@/public/indexModel.js'
import Add from './add.vue'
export default {
  mixins: [model],
  name: 'user',
  components: { Add },
  data() {
    return {
      searchBarData: [
        [
          { type: 'option', value: null, key: 'status', placeholder: '状态', options: [
            { label: '启用', value: 1 },
            { label: '禁用', value: 0 }]
          },
          { type: 'input', value: null, key: 'name', placeholder: '编码或名称模糊检索', class: 'w130' },
          { type: 'search', name: '查询' },
          { type: 'reset', name: '重置' }
        ], [
          { type: 'add', name: '新增' }
        ]
      ]
    }
  },
  mounted() {
    this.createDefaultMLoading('#user')
    this.fetchData()
  },
  methods: {
    fetchData(params = {}) {
      this.mloading.show()
      const { page, size } = this.pagination
      params.page = page
      params.size = size
      fetch('/admin/api/sysUser', params).then(({ data }) => {
        this.mloading.hide()
        this.table.data = data.items
        this.pagination.total = data.total
      }).catch((error) => {
        this.mloading.error(error.message, () => {
          this.fetchData()
        })
      })
    },
    searchAction(params) {
      this.fetchData(params)
    },
    handleSizeChange(val) {
      this.pagination.size = val
      this.fetchData()
    },
    handleCurrentChange(val) {
      this.pagination.page = val
      this.fetchData()
    },
    clickMoreCommand(command) {
      this.$message({ message: command, type: 'success' })
    },
    clickEdit(index, row) {
      this.$setKeyValue(this.add, { visiable: true, data: { type: 'edit', obj: row }})
    },
    showAdd() {
      this.$setKeyValue(this.add, { visiable: true, data: { type: 'add' }})
    },
    clickDelete(index, row) {
      this.$confirm('是否需要删除数据?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteId('/admin/api/sysUser', row.id).then(() => {
          this.$message({ type: 'success', message: '删除成功!' })
          this.fetchData()
        }).catch(() => {
          this.$message({ type: 'error', message: '删除失败!' })
        })
      })
    }
  }
}
</script>
