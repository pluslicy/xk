<template>
  <div class="worker">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <el-col :span="18">
          <el-form :inline="true">
            <el-form-item label="名称">
              <el-input v-model="param.name"></el-input>
            </el-form-item>
            <el-button type="text" size="samll" @click="loadWorker">搜索</el-button>
          </el-form>
        </el-col>
        <!-- <el-col :span="6" style="text-align:right">
          <el-button type="primary" size="small" @click="toSaveHandler">新增</el-button>
          <el-button type="primary" size="small" @click="toDownloadHandler">下载</el-button>
        </el-col> -->
      </el-row>
      
    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table size="small" :data="workerData.list" v-loading="loading">
      <el-table-column type="expand" fixed="left">
        <template slot-scope="props">
          {{props.row}}
        </template>
      </el-table-column>
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
     
      <el-table-column label="操作" width="150" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button type="text" size="mini" :disabled="!(scope.row.status === '已派单')" @click="toDeleteHandler(scope.row)">删除</el-button>
          <el-button type="text" size="mini" :disabled="!(scope.row.status === '已派单')" @click="toEditHandler(scope.row)">更换负责人</el-button>
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
    <!-- 模态框 -->
    <!-- <el-dialog :title="title" :visible.sync="visible" width="50%" @close="clearValidate" class="customer_modal"> -->
    <Briupdrawer 
      @on-change-visible="handlerVisibleChange"
      :drawerVisible="visible" :title="title"  width='40%'>
      <div slot="content">
        <el-form :model="form" ref="workerForm" :rules="rules" label-width="80px">
          <div v-if="form.repairs">
            <el-form-item label="报修地点">
              {{form.repairs.repairsAddress}}
            </el-form-item>
            <el-form-item label="报修时间">
              {{form.repairs.repairsTime | fmtDate}}
            </el-form-item>
          </div>
          <el-form-item label="指派工人" v-model="form.userId">
            <el-select v-model="form.userId">
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
      title:"新增工单信息",
      param:{
        page:1,
        pageSize:15
      },
      workerData:{
        list:[]
      },
      workerTypeData:{
        list:[]
      },
      usersData:{
        list:[]
      },
      form:{},
      rules:{
        name: [
          { required: true, message: '请输入工单名称', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入工单编号', trigger: 'blur' }
        ]
      }
    }
  },
  created(){
    this.loadWorker();
    this.loadUsers();
  },
  methods:{
    //下载
    toDownloadHandler(){
      window.location.href="http://47.102.206.136:8888/worker/download"
    },
    submitHandler(){
      console.log(this.$refs['workerForm']);
      this.$refs['workerForm'].validate((valid) => {
        if (valid) {
          let url = '/worker/saveOrUpdate'
          post(url,this.form).then(response => {
            this.$message({type:"success",message:response.message})
            this.loadWorker();
            this.visible = false;
          })
        } else {
          return false;
        }
      });
    },
    // 去保存
    toSaveHandler(){
      this.title = "新增工单信息";
      this.form = {};
      this.visible = true;
    },
    // 去修改
    toEditHandler(row){
      this.title = "更换工单负责人"
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
        let url = '/worker/deleteById'
        get(url,{id:row.id}).then(response =>{
          this.$message({ type: 'success', message: '删除成功!' });
          this.loadWorker();
        })
      })
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
      this.loadWorker();
    },
    handlerVisibleChange(val){
      this.visible = val;
      this.$refs['workerForm'].clearValidate();
    }
  }
}
</script>