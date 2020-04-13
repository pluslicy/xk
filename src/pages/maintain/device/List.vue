<template>
  <div class="device">
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
          <el-button type="primary" size="small" @click="toSaveHandler">新增</el-button>
          <el-button type="primary" size="small" @click="toDownloadHandler">下载</el-button>
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
      <el-table-column label="最近访问时间" prop="lastAccessTime" width="160">
         <template slot-scope="scope">
          {{scope.row.lastAccessTime | fmtDate}}
        </template>
      </el-table-column>
      <el-table-column label="介绍" prop="introduce" min-width="200"></el-table-column>
      <el-table-column label="操作" width="100" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="toDeleteHandler(scope.row)">删除</el-button>
          <el-button type="text" size="mini" @click="toEditHandler(scope.row)">修改</el-button>
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
        <el-form :model="form" ref="deviceForm" :rules="rules" label-width="80px">
          <el-form-item label="名称" prop="name">
            <el-input v-model="form.name"></el-input>
          </el-form-item>
          <el-form-item label="编号" prop="code">
            <el-input v-model="form.code"></el-input>
          </el-form-item>
          <el-form-item label="设备类型" prop="deviceTypeId">
            <el-select v-model="form.deviceTypeId">
              <el-option v-for="type in deviceTypeData.list" :key="type.id" :label="type.name" :value="type.id"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="参数" prop="parameters">
            <el-input v-model="form.parameters"></el-input>
          </el-form-item>
          <el-form-item label="安装地址" prop="installAddress">
            <el-input v-model="form.installAddress"></el-input>
          </el-form-item>
          <el-form-item label="价格" prop="price">
            <el-input v-model="form.price"></el-input>
          </el-form-item>
          <el-form-item label="介绍" prop="introduce">
            <el-input type="text" v-model="form.introduce"></el-input>
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
      form:{},
      rules:{
        name: [
          { required: true, message: '请输入设备名称', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入设备编号', trigger: 'blur' }
        ]
      }
    }
  },
  created(){
    this.loadDevices();
    this.loadDeviceTypes();
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
          let url = '/device/saveOrUpdate'
          post(url,this.form).then(response => {
            this.$message({type:"success",message:response.message})
            this.loadDevices();
            this.visible = false;
          })
        } else {
          return false;
        }
      });
    },
    // 去保存
    toSaveHandler(){
      this.title = "新增设备信息";
      this.form = {};
      this.visible = true;
    },
    // 去修改
    toEditHandler(row){
      this.title = "修改设备信息"
      this.form = _.clone(row);
      this.visible = true;
    },
    // 去删除
    toDeleteHandler(row){
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let url = '/device/deleteById'
        get(url,{id:row.id}).then(response =>{
          this.$message({ type: 'success', message: '删除成功!' });
          this.loadDevices();
        })
      })
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