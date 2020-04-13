<template>
  <div class="sign">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <el-col :span="24" style="text-align:right">
          <!-- <el-button type="primary" size="small" @click="toDownloadHandler">下载</el-button> -->
        </el-col>
      </el-row>
      
    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table size="small" :data="signData.list" v-loading="loading">
      <el-table-column label="序号" type="index" :index="1" fixed="left"></el-table-column>
      <el-table-column label="签到时间"  width="160" >
        <template slot-scope="scope">
          {{scope.row.signTime | fmtDate}}
        </template>
      </el-table-column>
      <el-table-column label="状态"  width="120" align="center">
        <template slot-scope="scope">
          <el-tag type="success">{{scope.row.status}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="签到地点" prop="signLocation" min-width="140" ></el-table-column>
      
      <el-table-column label="操作" width="100" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="toDeleteHandler(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 表格 -->
    <!-- 分页 -->
    <el-pagination background layout="prev, pager, next" small
      :hide-on-single-page="true"
      :total="signData.total" 
      :page-size="signData.pageSize" 
      :current-page="signData.page"
      @current-change="currentChangeHandler">
    </el-pagination>
    <!-- 分页 -->
  </div>
</template>
<script>
import {get,post} from '@/utils/request'
import _ from 'lodash'
export default {
  data(){
    return {
      loading:false,
      param:{
        page:1,
        pageSize:15
      },
      signData:{
        list:[]
      }
    }
  },
  created(){
    this.loadSign();
  },
  methods:{
    // 去删除
    toDeleteHandler(row){
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let url = '/sign/deleteById'
        get(url,{id:row.id}).then(response =>{
          this.$message({ type: 'success', message: '删除成功!' });
          this.loadSign();
        })
      })
    },
    //下载
    toDownloadHandler(){
      window.location.href="http://47.102.206.136:8888/sign/download"
    },
    // 查询签到
    loadSign(){
      this.loading = true;
      let url = "/sign/pageQuery"
      get(url,this.param).then(response => {
        this.signData = response.data;
        this.loading = false;
      })
    },
    // 当前页发生改变
    currentChangeHandler(page){
      this.param.page = page;
      this.loadSign();
    }
  }
}
</script>