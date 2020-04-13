<template>
  <div class="check">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <el-col :span="18">
          <el-form :inline="true">
            <el-form-item label="名称">
              <el-input v-model="param.spareName" />
            </el-form-item>
            <el-button type="text" size="samll" @click="loadSpareUse">搜索</el-button>
          </el-form>
        </el-col>
      </el-row>

    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table v-loading="loading" size="small" :data="spareUseData.list">
      <el-table-column label="序号" type="index" :index="1" fixed="left" />
      <el-table-column label="备件名称" prop="spareName" width="120" align="center" />
      <el-table-column label="备件使用数量" prop="spareNum" width="120" align="center" />
      <el-table-column label="申请人" prop="userRealname" width="120" align="center" />
      <el-table-column label="状态" width="120" align="center">
        <template slot-scope="scope">
          <el-tag type="success">{{ scope.row.status }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="申请时间" min-width="160">
        <template slot-scope="scope">
          {{ scope.row.useTime | fmtDate }}
        </template>
      </el-table-column>

      <el-table-column label="操作" width="100" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="toCheckHandler(scope.row,'申请通过')">通过</el-button>
          <el-button type="text" size="mini" @click="toCheckHandler(scope.row,'申请拒绝')">拒绝</el-button>
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
      :total="spareUseData.total"
      :page-size="spareUseData.pageSize"
      :current-page="spareUseData.page"
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
        <el-form ref="spareUseForm" :model="form" :rules="rules" label-width="80px">
          <el-form-item label="备件名称" prop="name">
            <el-input v-model="form.name" />
          </el-form-item>
          <el-form-item label="参数" prop="parameters">
            <el-input v-model="form.parameters" />
          </el-form-item>
          <el-form-item label="价格" prop="price">
            <el-input v-model="form.price" />
          </el-form-item>
          <el-form-item label="数量" prop="stock">
            <el-input v-model="form.stock" />
          </el-form-item>
          <el-form-item label="介绍" prop="introduce">
            <el-input v-model="form.introduce" type="text" />
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
      title: '新增备件信息',
      param: {
        page: 1,
        pageSize: 15
      },
      spareUseData: {
        list: []
      },
      spareUseTypeData: {
        list: []
      },
      form: {},
      rules: {
        name: [
          { required: true, message: '请输入备件名称', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入备件编号', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.loadSpareUse()
  },
  methods: {
    // 下载
    toDownloadHandler() {
      window.location.href = 'http://47.102.206.136:8888/spareUse/download'
    },
    submitHandler() {
      console.log(this.$refs['spareUseForm'])
      this.$refs['spareUseForm'].validate((valid) => {
        if (valid) {
          const url = '/spareUse/saveOrUpdate'
          post(url, this.form).then(response => {
            this.$message({ type: 'success', message: response.message })
            this.loadSpareUse()
            this.visible = false
          })
        } else {
          return false
        }
      })
    },
    // 去保存
    toSaveHandler() {
      this.title = '新增备件信息'
      this.form = {}
      this.visible = true
    },
    // 去修改
    toEditHandler(row) {
      this.title = '修改备件信息'
      this.form = _.clone(row)
      this.visible = true
    },
    // 去删除
    toCheckHandler(row, status) {
      this.$confirm('将该申请状态修改为 ' + status + ' , 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        row.status = status
        const url = '/spareUse/saveOrUpdate'
        post(url, row).then(response => {
          this.$message({ type: 'success', message: response.message })
          this.loadSpareUse()
          this.visible = false
        })
      })
    },
    // 查询备件
    loadSpareUse() {
      this.loading = true
      const url = '/spareUse/pageQuery'
      get(url, this.param).then(response => {
        this.spareUseData = response.data
        this.loading = false
      })
    },
    // 当前页发生改变
    currentChangeHandler(page) {
      this.param.page = page
      this.loadSpareUse()
    },
    handlerVisibleChange(val) {
      this.visible = val
      this.$refs['spareUseForm'].clearValidate()
    }
  }
}
</script>
