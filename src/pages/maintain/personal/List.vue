<template>
  <div class="personal_repairs">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <!-- <el-col :span="18">
          <el-form :inline="true">
            <el-form-item label="名称">
              <el-input v-model="param.name"></el-input>
            </el-form-item>
            <el-button type="text" size="samll" @click="loadRepairs">搜索</el-button>
          </el-form>
        </el-col> -->
        <el-col :span="24" style="text-align:right">
          <el-button type="primary" size="small" @click="toSaveHandler">报修</el-button>
        </el-col>
      </el-row>

    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table v-loading="loading" size="small" :data="repairsData.list">
      <el-table-column type="expand" fixed="left">
        <template slot-scope="props">
          <el-form label-width="100px">
            <el-form-item label="报修地点">
              {{ props.row.repairsAddress }}
            </el-form-item>
            <el-form-item label="报修时间">
              {{ props.row.repairsTime | fmtDate }}
            </el-form-item>
            <el-form-item label="状态">
              <el-tag type="success">{{ props.row.status }}</el-tag>
            </el-form-item>
            <el-form-item v-if="props.row.device" label="故障设备">
              <strong>名称：{{ props.row.device.name }}</strong>
              <strong>编号：{{ props.row.device.code }}</strong>
            </el-form-item>
            <el-form-item label="图片" style="position:absolute;right:1em;top:1em;width:400px">
              <img :src="'http://134.175.154.93:8888/group1/'+props.row.picture" alt="" style="width:100%">
            </el-form-item>
            <el-form-item v-if="props.row.user" label="报修人">
              {{ props.row.user.realname }}
            </el-form-item>
            <el-form-item v-if="props.row.workers" label="处理日志">
              <el-table :data="props.row.workers">
                <el-table-column label="序号" type="index" :index="1" />
                <el-table-column label="记录" prop="handlerContent" />
                <el-table-column label="时间" prop="handlerTime">
                  <template slot-scope="s">
                    {{ s.row.handlerTime | fmtDate }}
                  </template>
                </el-table-column>
              </el-table>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>
      <el-table-column label="序号" type="index" :index="1" fixed="left" />
      <el-table-column label="报修地点" prop="repairsAddress" width="140" />
      <el-table-column label="报修时间" prop="repairsTime" width="160">
        <template slot-scope="scope">
          {{ scope.row.repairsTime | fmtDate }}
        </template>
      </el-table-column>
      <el-table-column label="状态" width="120" align="center">
        <template slot-scope="scope">
          <el-tag type="success">{{ scope.row.status }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="报修人" prop="user.realname" width="80" />
      <el-table-column label="报修信息" prop="introduce" min-width="200" />
      <el-table-column label="操作" width="100" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" :disabled="!(scope.row.status ==='未受理')" size="mini" @click="toDeleteHandler(scope.row)">撤销</el-button>
          <el-button type="text" :disabled="!(scope.row.status ==='未受理')" size="mini" @click="toEditHandler(scope.row)">修改</el-button>
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
      :total="repairsData.total"
      :page-size="repairsData.pageSize"
      :current-page="repairsData.page"
      @current-change="currentChangeHandler"
    />
    <!-- 分页 -->
    <!-- 模态框 -->
    <Briupdrawer
      :drawer-visible="visible"
      :title="title"
      width="40%"
      @on-change-visible="handlerVisibleChange"
    >
      <div slot="content">
        <el-form ref="repairsForm" :model="form" :rules="rules" label-width="80px">
          <el-form-item label="报修地点" prop="repairsAddress">
            <el-input v-model="form.repairsAddress" />
          </el-form-item>
          <el-form-item label="报修信息" prop="introduce">
            <el-input v-model="form.introduce" type="textarea" />
          </el-form-item>
          <el-form-item label="上传图片" prop="photo">
            <el-upload
              class="upload-demo"
              action="http://47.102.206.136:8888/file/upload"
              :on-success="uploadSuccessHandler"
              :file-list="fileList"
            >
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">文件大小不允许超过3M</div>
            </el-upload>
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
import { mapGetters } from 'vuex'
export default {
  data() {
    return {
      visible: false,
      loading: false,
      title: '新增报修信息',
      param: {
        page: 1,
        pageSize: 15
      },
      repairsData: {
        list: []
      },
      repairsTypeData: {
        list: []
      },
      form: {},
      fileList: [],
      rules: {
        name: [
          { required: true, message: '请输入地址信息', trigger: 'blur' }
        ],
        introduce: [
          { required: true, message: '请输入描述信息', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.loadRepairs()
  },
  computed: {
    ...mapGetters(['user'])
  },
  methods: {
    // 下载
    submitHandler() {
      this.form.userId = this.user.id
      this.$refs['repairsForm'].validate((valid) => {
        if (valid) {
          const url = '/repairs/saveOrUpdate'
          post(url, this.form).then(response => {
            this.$message({ type: 'success', message: response.message })
            this.loadRepairs()
            this.visible = false
          })
        } else {
          return false
        }
      })
    },
    // 上传成功
    uploadSuccessHandler(response) {
      if (response.status == 200) {
        this.$set(this.form, 'picture', response.data.id)
      } else {
        this.$message({ type: 'error', message: '附件服务器异常！' })
        this.visible = false
      }
    },
    // 去保存
    toSaveHandler() {
      this.title = '新增报修信息'
      this.form = {}
      this.visible = true
    },
    // 去修改
    toEditHandler(row) {
      this.title = '修改报修信息'
      this.form = _.clone(row)
      this.visible = true
    },
    // 去撤销
    toDeleteHandler(row) {
      this.$confirm('此操作将永久撤销该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const url = '/repairs/deleteById'
        get(url, { id: row.id }).then(response => {
          this.$message({ type: 'success', message: '撤销成功!' })
          this.loadRepairs()
        })
      })
    },
    // 查询报修
    loadRepairs() {
      this.loading = true
      const url = '/repairs/pageQuery'
      get(url, this.param).then(response => {
        this.repairsData = response.data
        this.loading = false
      })
    },
    // 当前页发生改变
    currentChangeHandler(page) {
      this.param.page = page
      this.loadRepairs()
    },
    handlerVisibleChange(val) {
      this.visible = val
      this.$refs['repairsForm'].clearValidate()
    }
  }
}
</script>
