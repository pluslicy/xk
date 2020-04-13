<template>
  <div class="deviceType">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <el-col :span="18">
          <el-form :inline="true">
            <el-form-item label="名称">
              <el-input v-model="param.name" />
            </el-form-item>
            <el-button type="text" size="samll" @click="loadDeviceTypes">搜索</el-button>
          </el-form>
        </el-col>
        <el-col :span="6" style="text-align:right">
          <el-button type="primary" size="small" @click="toSaveHandler">新增</el-button>
        </el-col>
      </el-row>

    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table v-loading="loading" size="small" :data="deviceTypeData.list">
      <el-table-column label="序号" type="index" :index="1" fixed="left" />
      <el-table-column label="名称" prop="name" min-width="140" />
      <el-table-column label="操作" width="100" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="toDeleteHandler(scope.row)">删除</el-button>
          <el-button type="text" size="mini" @click="toEditHandler(scope.row)">修改</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 表格 -->
    <!-- 分页 -->
    <el-pagination
      background
      layout="prev, pager, next"
      small
      :hide-on-single-page="true"
      :total="deviceTypeData.total"
      :page-size="deviceTypeData.pageSize"
      :current-page="deviceTypeData.page"
      @current-change="currentChangeHandler"
    />
    <!-- 分页 -->
    <!-- 模态框 -->
    <!-- <el-dialog :title="title" :visible.sync="visible" width="50%" @close="clearValidate" class="customer_modal"> -->
    <Briupdrawer
      :drawer-visible="visible"
      :title="title"
      width="40%"
      @on-change-visible="handlerVisibleChange"
    >
      <div slot="content">
        <el-form ref="deviceTypeForm" :model="form" :rules="rules" label-width="80px">
          <el-form-item label="名称" prop="name">
            <el-input v-model="form.name" />
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button size="small" @click="visible = false">取消</el-button>
        <el-button size="small" type="primary" @click="submitHandler">确定</el-button>
      </span>
    </Briupdrawer>
    <!-- </el-dialog> -->
    <!-- 模态框 -->
  </div>
</template>
<script>
import { get, post } from '@/utils/request'
import _ from 'lodash'
export default {
  data() {
    return {
      visible: false,
      loading: false,
      title: '新增设备类型信息',
      param: {
        page: 1,
        pageSize: 15
      },
      deviceTypeData: {
        list: []
      },
      form: {},
      rules: {
        name: [
          { required: true, message: '请输入设备类型名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.loadDeviceTypes()
  },
  methods: {
    submitHandler() {
      console.log(this.$refs['deviceTypeForm'])
      this.$refs['deviceTypeForm'].validate((valid) => {
        if (valid) {
          const url = '/deviceType/saveOrUpdate'
          post(url, this.form).then(response => {
            this.$message({ type: 'success', message: response.message })
            this.loadDeviceTypes()
            this.visible = false
          })
        } else {
          return false
        }
      })
    },
    // 去保存
    toSaveHandler() {
      this.title = '新增设备类型信息'
      this.form = {}
      this.visible = true
    },
    // 去修改
    toEditHandler(row) {
      this.title = '修改设备类型信息'
      this.form = _.clone(row)
      this.visible = true
    },
    // 去删除
    toDeleteHandler(row) {
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const url = '/deviceType/deleteById'
        get(url, { id: row.id }).then(response => {
          this.$message({ type: 'success', message: '删除成功!' })
          this.loadDeviceTypes()
        })
      })
    },
    // 查询
    loadDeviceTypes() {
      this.loading = true
      const url = '/deviceType/pageQuery'
      get(url, this.param).then(response => {
        this.deviceTypeData = response.data
        this.loading = false
      })
    },
    // 当前页发生改变
    currentChangeHandler(page) {
      this.param.page = page
      this.loadDeviceTypes()
    },
    handlerVisibleChange(val) {
      this.visible = val
      this.$refs['deviceTypeForm'].clearValidate()
    }
  }
}
</script>
