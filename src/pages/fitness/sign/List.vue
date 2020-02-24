<template>
  <div class="sign">
    <!-- 按钮 -->
    <div class="btns">
      <el-form :inline="true" :model="formInline" class="demo-form-inline">
      <!-- <el-button class="sign_btn" type="primary" @click="SignInHandler">签到</el-button> -->
        <el-form-item label="课程名称">
          <el-select v-model="formInline.courseId" placeholder="课程名称" clearable>
            <el-option v-for="s in courses" :label="s.name" :value="s.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item>
          <!-- <el-button type="primary">查询</el-button> -->
        </el-form-item>
      </el-form>
    </div>
    <!-- {{formInline}} -->
    <!-- {{courses}} -->
    <!-- /按钮 -->
    <!-- 表格 -->
    <el-table :data="signs.list" size="small">
      <el-table-column label="编号" prop="id" width="50px" />
      <el-table-column label="名称" prop="type" />
      <el-table-column label="会员名" prop="userName" />
      <el-table-column label="签到时间" prop="signTime" />
      <el-table-column label="签到地址" prop="address" />
      <el-table-column label="课程名称" prop="courseName" />
      <el-table-column label="操作" width="100px" align="center">
        <template slot-scope="scope">
          <a href="" @click.prevent="toDetailsHandler(scope.row)">详情</a>
        </template>
      </el-table-column>
    </el-table>
    <!-- /表格 -->
    <!-- {{signs}} -->
    <!-- 分页 -->
    <el-pagination
      background
      layout="prev, pager, next"
      :total="signs.total"
      :page-size="signs.pageSize"
      :current-page="signs.page"
      @current-change="pageChangeHandler"
    />
    <!-- /分页 -->
  </div>
</template>

<script>
import { mapState,mapActions } from 'vuex'
import { get, post } from '@/utils/request'

export default { 
  data() {
    return {
      courses:{},
      formInline: {
        page:1,
        pageSize:5
      }
    }
  },
  computed: {
    ...mapState('sign',['signs'])
  },
  created() {
    this.PageQuerySign(this.formInline)
    this.loadCourses()
  },
  // 根据课程名称过滤签到信息
  watch:{
      formInline:{
        handler(now,old){
          this.PageQuerySign(this.formInline);
        },
        deep:true
      }
    },
  methods: {
    ...mapActions('sign',['PageQuerySign']),
    // 查询所有课程
    loadCourses() {
      const url = '/course/query'
      get(url).then(result => {
        this.courses = result.data
      })
    },
    // 签到
    SignInHandler(){

    },
    // 查看签到详情
    toDetailsHandler(){

    },
    // 分页
    pageChangeHandler(page){
      this.formInline.page = page
    }
  }

}
</script>

<style>
.sign_btn{
  margin-right: 2em;
}
</style>
