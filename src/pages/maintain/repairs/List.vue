<template>
  <!-- 设备保修 -->
  <div class="repairs">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <el-col :span="18">
          <el-form :inline="true">
            <el-form-item label="名称">
              <el-input v-model="param.name"></el-input>
            </el-form-item>
            <el-button type="text" size="samll" @click="loadRepairs">搜索</el-button>
          </el-form>
        </el-col>
      </el-row>
      
    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table size="small" :data="repairsData.list" v-loading="loading">
      <el-table-column type="expand" fixed="left">
        <template slot-scope="props">
          {{props}}
        </template>
      </el-table-column>
      <el-table-column label="序号" type="index" :index="1" fixed="left"></el-table-column>
      <el-table-column label="报修地点" prop="repairsAddress" width="140" ></el-table-column>
      <el-table-column label="报修时间" prop="repairsTime" width="160" >
         <template slot-scope="scope">
          {{scope.row.repairsTime | fmtDate}}
        </template>
      </el-table-column>
      <el-table-column label="状态"  width="120" align="center">
        <template slot-scope="scope">
          <el-tag type="success">{{scope.row.status}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="报修人" prop="user.realname" width="80"></el-table-column>
      <el-table-column label="报修信息" prop="introduce" min-width="200"></el-table-column>
      <el-table-column label="操作" width="100" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" size="mini" @click="toHandle(scope.row)" :disabled="!(scope.row.status == '未受理')">受理</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 表格 -->
    <!-- 分页 -->
    <el-pagination background layout="prev, pager, next" small
      :hide-on-single-page="true"
      :total="repairsData.total" 
      :page-size="repairsData.pageSize" 
      :current-page="repairsData.page"
      @current-change="currentChangeHandler">
    </el-pagination>
    <!-- 分页 -->
    <!-- 模态框 -->
    <!-- <el-dialog :title="title" :visible.sync="visible" width="50%" @close="clearValidate" class="customer_modal"> -->
    <Briupdrawer 
      @on-change-visible="handlerVisibleChange"
      :drawerVisible="visible" :title="title"  width='40%'>
      <div slot="content">
        {{worker}}
        <el-form :model="form" ref="workerForm" :rules="rules" label-width="80px">
            <el-form-item label="报修人" v-if="form.user">
              {{form.user.realname}}
            </el-form-item>
            <el-form-item label="报修地点">
              {{form.repairsAddress}}
            </el-form-item>
            <el-form-item label="报修时间">
              {{form.repairsTime | fmtDate}}
            </el-form-item>
            <el-form-item label="故障设备" v-if="form.device">
              <strong>名称：{{form.device.name}}</strong>
              <strong>编号：{{form.device.code}}</strong>
            </el-form-item>
            <el-form-item label="图片">
              <div  style="width:80%;border:1px solid #ccc;border-radius:5px;padding:.5em">
                <img :src="'http://134.175.154.93:8888/group1/'+form.picture" alt="" style="width:100%">
              </div>
            </el-form-item>
            <el-form-item label="指派工人" v-model="form.userId">
              <el-select v-model="worker.userId">
                <el-option v-for="user in usersData.list" :key="user.id" :value="user.id" :label="user.realname"></el-option>
              </el-select>
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
      title:"新增报修信息",
      param:{
        page:1,
        pageSize:15
      },
      repairsData:{
        list:[]
      },
      repairsTypeData:{
        list:[]
      },
      usersData:{ list:[] },
      form:{},
      worker:{},
      rules:{
        name: [
          { required: true, message: '请输入报修名称', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入报修编号', trigger: 'blur' }
        ]
      }
    }
  },
  created(){
    this.loadRepairs();
    this.loadUsers();
  },
  methods:{
    
    //下载
    toDownloadHandler(){
      window.location.href="http://47.102.206.136:8888/repairs/download"
    },
    submitHandler(){
      this.$refs['workerForm'].validate((valid) => {
        if (valid) {
          let url = '/worker/saveOrUpdate'
          post(url,this.worker).then(response => {
            this.$message({type:"success",message:response.message})
            this.loadRepairs();
            this.visible = false;
          })
        } else {
          return false;
        }
      });
    },
    // 去保存
    toSaveHandler(){
      this.title = "新增报修信息";
      this.form = {};
      this.visible = true;
    },
    // 受理报修
    toHandle(row){
      this.title = "受理报修信息"
      this.form = _.clone(row);
      this.worker.repairsId = row.id;
      this.worker.type = '维修';
      this.visible = true;
    },
    // 去删除
    toDeleteHandler(row){
      this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let url = '/repairs/deleteById'
        get(url,{id:row.id}).then(response =>{
          this.$message({ type: 'success', message: '删除成功!' });
          this.loadRepairs();
        })
      })
    },
    // 查询报修
    loadRepairs(){
      this.loading = true;
      let url = "/repairs/pageQuery"
      get(url,this.param).then(response => {
        this.repairsData = response.data;
        this.loading = false;
      })
    },
    // 查询工人
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
    // 当前页发生改变
    currentChangeHandler(page){
      this.param.page = page;
      this.loadRepairs();
    },
    handlerVisibleChange(val){
      this.visible = val;
      this.$refs['workerForm'].clearValidate();
    }
  }
}
</script>