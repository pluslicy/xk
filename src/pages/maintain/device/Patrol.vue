<template>
  <div class="repairs">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <!-- <el-col :span="18">
          <el-form :inline="true">
            <el-form-item label="名称">
              <el-input v-model="param.name"></el-input>
            </el-form-item>
            <el-button type="text" size="samll" @click="loadWorker">搜索</el-button>
          </el-form>
        </el-col> -->
        <!-- <el-col :span="6" style="text-align:right">
          <el-button type="primary" size="small" @click="toSaveHandler">新增</el-button>
          <el-button type="primary" size="small" @click="toDownloadHandler">下载</el-button>
        </el-col> -->
      </el-row>
      
    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table size="small" :data="workerData.list" v-loading="loading">
      
      <el-table-column label="序号" type="index" :index="1" fixed="left"></el-table-column>
      <el-table-column label="工单类型" prop="type" width="80" align="center" ></el-table-column>
      <el-table-column label="工单状态"  width="120" align="center">
        <template slot-scope="scope">
          <el-tag type="success">{{scope.row.status}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="报修地点"  width="140" >
        <template slot-scope="scope">
          <span v-if="scope.row.repairs">{{scope.row.repairs.repairsAddress}}</span>
        </template>
      </el-table-column>
      <el-table-column label="报修时间" width="160" >
         <template slot-scope="scope">
           <span v-if="scope.row.repairs">
            {{scope.row.repairs.repairsTime | fmtDate}}
           </span>
        </template>
      </el-table-column>
      <el-table-column label="工单负责人" prop="user.realname" width="100" ></el-table-column>
      <el-table-column label="工单处理描述" prop="handlerContent" min-width="100" ></el-table-column>
     
      <el-table-column label="操作" width="80" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="toDetailsHandler(scope.row)">详情</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 表格 -->
    <!-- 分页 -->
    <el-pagination background layout="prev, pager, next" small
      :hide-on-single-page="true"
      :total="workerData.total" 
      :page-size="workerData.pageSize" 
      :current-page="workerData.page"
      @current-change="currentChangeHandler">
    </el-pagination>
    <!-- 分页 -->
    <!-- 抽屉 -->
    <Briupdrawer 
      @on-change-visible="handlerVisibleChange"
      :drawerVisible="visible" :title="title"  width='40%'>
      <div slot="content">
        <!-- 报修 -->
        <div v-if="form.repairs" style="border-bottom:1px solid #f4f4f4">
          <el-form label-width="80px">
            <el-form-item label="报修描述">
              {{form.repairs.introduce}}
            </el-form-item>
            <el-form-item label="报修照片">
              <div  style="width:300px;padding:.5em">
                <img :src="'http://134.175.154.93:8888/group1/'+form.repairs.picture" alt="" style="width:100%">
              </div>
            </el-form-item>
          </el-form>
        </div>
        <!-- /报修 -->
        <!-- 维修 -->
        <div v-if="form.handlerContent">
          <el-form label-width="80px">
            <el-form-item label="报修描述">
              {{form.handlerContent}}
            </el-form-item>
            <el-form-item label="报修照片">
              <div  style="width:300px;padding:.5em">
                <img :src="'http://134.175.154.93:8888/group1/'+form.picture" alt="" style="width:100%">
              </div>
            </el-form-item>
          </el-form>
        </div>
        <!-- 维修 -->
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button size="small" @click="visible = false">取消</el-button>
        <el-button size="small" type="primary" @click="submitHandler">确定</el-button>
      </span>
    </Briupdrawer>
    <!-- /抽屉 -->
  </div>
</template>
<script>
import {get,post} from '@/utils/request'
import _ from 'lodash'
export default {
  data(){
    return {
      visible:false,
      form:{},
      workerData:{ list:[] },
      deviceData:{ list:[] },
      loading:false,
      title:"新增工单信息",
      param:{
        page:1,
        pageSize:15
      }
    }
  },
  created(){
    this.loadWorker();
    this.loadDevices();
  },
  methods:{
    // 查看详情
    toDetailsHandler(row){
      this.title = "记录详情"
      this.form = _.clone(row);
      this.visible = true;
    },
    // 查询工单
    loadWorker(){
      this.loading = true;
      let url = "/worker/pageQuery"
      get(url,this.param).then(response => {
        this.workerData = response.data;
        this.loading = false;
      })
    },
    // 查询设备
    loadDevices(){
      let url = "/device/pageQuery"
      get(url,{
        page:1,
        pageSize:100
      }).then(response => {
        this.deviceData = response.data;
      })
    },
    // 当前页发生改变
    currentChangeHandler(page){
      this.param.page = page;
      this.loadWorker();
    },
     handlerVisibleChange(val){
      this.visible = val;
      this.$refs['workerForm'].clearValidate();
    }
  }
}
</script>