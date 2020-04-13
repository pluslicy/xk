<template>
  <div class="device_patrol">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <el-col :span="18">
          <el-form :inline="true">
            <el-form-item label="名称">
              <el-input v-model="param.name"></el-input>
            </el-form-item>
            <el-button type="text" size="samll" @click="loadDevices">搜索</el-button>
          </el-form>
        </el-col>
        <el-col :span="6" style="text-align:right">
          <el-button type="primary" size="small" @click="toPatrolHandler">批量设置巡检任务</el-button>
        </el-col>
      </el-row>
      
    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table size="small" :data="deviceData.list" v-loading="loading">
      <el-table-column label="序号" type="index" :index="1" fixed="left"></el-table-column>
      <el-table-column label="设备编号" prop="code" width="100" fixed="left"></el-table-column>
      <el-table-column label="名称" prop="name" min-width="240" ></el-table-column>
      <el-table-column label="设备类型" prop="deviceType.name" width="140" ></el-table-column>
      <el-table-column label="价格" prop="price" width="80"></el-table-column>
      <el-table-column label="参数" prop="parameters"  width="160"></el-table-column>
      <el-table-column label="安装地点" prop="installAddress" width="160" ></el-table-column>
      <el-table-column label="安装时间" prop="installTime" width="160" >
         <template slot-scope="scope">
          {{scope.row.installTime | fmtDate}}
        </template>
      </el-table-column>
      <el-table-column label="最近巡检时间" prop="lastAccessTime" width="160">
         <template slot-scope="scope">
          {{scope.row.lastAccessTime | fmtDate}}
        </template>
      </el-table-column>
      <el-table-column label="介绍" prop="introduce" min-width="200"></el-table-column>
      <el-table-column label="操作" width="100" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="toPatrolHandler(scope.row)">设置巡检任务</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 表格 -->
    <!-- 分页 -->
    <el-pagination background layout="prev, pager, next" small
      :hide-on-single-page="true"
      :total="deviceData.total" 
      :page-size="deviceData.pageSize" 
      :current-page="deviceData.page"
      @current-change="currentChangeHandler">
    </el-pagination>
    <!-- 分页 -->
    <!-- 模态框 -->
    <!-- <el-dialog :title="title" :visible.sync="visible" width="50%" @close="clearValidate" class="customer_modal"> -->
    <Briupdrawer 
      @on-change-visible="handlerVisibleChange"
      :drawerVisible="visible" :title="title"  width='40%'>
      <div slot="content">
        <el-form :model="repair" size="mini" ref="deviceForm" :rules="rules" label-width="80px">
          <el-form-item label="名称名称" prop="name">
            {{form.name}}
          </el-form-item>
          <el-form-item label="设备编号" prop="code">
            {{form.code}}
          </el-form-item>
          <el-form-item label="设备参数" prop="parameters">
            {{form.parameters}}
          </el-form-item>
          <el-form-item label="安装地址" prop="installAddress">
            {{form.installAddress}}
          </el-form-item>
          {{repair}}
          <el-form-item label="指派工人" prop="userId">
              <el-select v-model="repair.userId">
                <el-option v-for="user in usersData.list" :key="user.id" :value="user.id" :label="user.realname"></el-option>
              </el-select>
            </el-form-item>
          <el-form-item label="巡检要求" prop="patrolIntroduce">
            <el-input type="textarea" v-model="repair.patrolIntroduce"></el-input>
          </el-form-item>
           <el-form-item label="结束时间" prop="patrolEndTime">
             <el-date-picker v-model="repair.patrolEndTime" type="date" placeholder="选择日期" value-format="timestamp">
            </el-date-picker>
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
import {get,post} from '@/utils/request'
import _ from 'lodash'
export default {
  data(){
    return {
      visible:false,
      loading:false,
      title:"新增设备信息",
      param:{
        page:1,
        pageSize:15
      },
      deviceData:{
        list:[]
      },
      deviceTypeData:{
        list:[]
      },
      usersData:{ list:[] },
      form:{},
      rules:{
        patrolIntroduce: [
          { required: true, message: '请输入巡检要求', trigger: 'blur' }
        ],
        userId: [
          { required: true, message: '请选择工人', trigger: 'blur' }
        ],
        endTime: [
          { required: true, message: '请选择结束时间', trigger: 'blur' }
        ]
      },
      repair:{}
    }
  },
  created(){
    this.loadDevices();
    this.loadDeviceTypes();
    this.loadUsers();
  },
  methods:{
    //下载
    toDownloadHandler(){
      window.location.href="http://47.102.206.136:8888/device/download"
    },
    submitHandler(){
      console.log(this.$refs['deviceForm']);
      this.$refs['deviceForm'].validate((valid) => {
        if (valid) {
          let url = '/worker/patrol'
          post(url,this.repair).then(response => {
            this.$message({type:"success",message:response.message})
            this.loadDevices();
            this.visible = false;
          })
        } else {
          return false;
        }
      });
    },
    // 去修改
    toPatrolHandler(row){
      this.title = "设置巡检任务"
      this.form = _.clone(row);
      this.repair.deviceId = row.id;
      this.repair.type = '巡检';
      this.visible = true;
    },
    // 查询设备
    loadDevices(){
      this.loading = true;
      let url = "/device/pageQuery"
      get(url,this.param).then(response => {
        this.deviceData = response.data;
        this.loading = false;
      })
    },
    loadUsers(){
      let url = "/baseUser/pageQuery"
      get(url,{
        page:1,
        pageSize:20,
        rolename:'工人'
      }).then(response => {
        this.usersData = response.data;
      })
    },
     // 查询设备类型
    loadDeviceTypes(){
      let url = "/deviceType/pageQuery"
      get(url,{
        page:1,
        pageSize:100
      }).then(response => {
        this.deviceTypeData = response.data;
      })
    },
    // 当前页发生改变
    currentChangeHandler(page){
      this.param.page = page;
      this.loadDevices();
    },
    handlerVisibleChange(val){
      this.visible = val;
      this.$refs['deviceForm'].clearValidate();
    }
  }
}
</script>