<template>
  <div class="spareUse">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <el-col :span="18">
          <el-form :inline="true">
            <el-form-item label="名称">
              <el-input v-model="param.spareName"></el-input>
            </el-form-item>
            <el-button type="text" size="samll" @click="loadSpareUse">搜索</el-button>
          </el-form>
        </el-col>
        <el-col :span="6" style="text-align:right">
          <el-button type="primary" size="small" @click="toDownloadHandler">下载</el-button>
        </el-col>
      </el-row>
      
    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table size="small" :data="spareUseData.list" v-loading="loading">
      <el-table-column label="序号" type="index" :index="1" fixed="left"></el-table-column>
      <el-table-column label="备件名称" prop="spareName" width="120" align="center"></el-table-column>
      <el-table-column label="备件使用数量" prop="spareNum" width="120" align="center"></el-table-column>
      <el-table-column label="申请人" prop="userRealname" width="120" align="center"></el-table-column>
      <el-table-column label="状态"  width="120" align="center">
        <template slot-scope="scope">
          <el-tag type="success">{{scope.row.status}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="申请时间"  min-width="160" >
        <template slot-scope="scope">
          {{scope.row.useTime | fmtDate}}
        </template>
      </el-table-column>
    </el-table>
    <!-- 表格 -->
    <!-- 分页 -->
    <el-pagination background layout="prev, pager, next" small
      :hide-on-single-page="true"
      :total="spareUseData.total" 
      :page-size="spareUseData.pageSize" 
      :current-page="spareUseData.page"
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
      spareUseData:{ list:[] },
      spareUseTypeData:{ list:[] },
    }
  },
  created(){
    this.loadSpareUse();
  },
  methods:{
    //下载
    toDownloadHandler(){
      window.location.href="http://47.102.206.136:8888/spareUse/download"
    },
    // 查询备件
    loadSpareUse(){
      this.loading = true;
      let url = "/spareUse/pageQuery"
      get(url,this.param).then(response => {
        this.spareUseData = response.data;
        this.loading = false;
      })
    },
    // 当前页发生改变
    currentChangeHandler(page){
      this.param.page = page;
      this.loadSpareUse();
    }
  }
}
</script>