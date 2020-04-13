<template>
  <div class="personal_comment">
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
        <!-- <el-col :span="6" style="text-align:right">
          <el-button type="primary" size="small" @click="toSaveHandler">报修</el-button>
        </el-col> -->
      </el-row>
      
    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table size="small" :data="repairsData.list" v-loading="loading">
      <el-table-column type="expand" fixed="left">
        <template slot-scope="props">
          <el-form label-width="100px" size="mini">
            <el-form-item label="报修地点">
              {{props.row.repairsAddress}}
            </el-form-item>
            <el-form-item label="报修时间">
              {{props.row.repairsTime | fmtDate}}
            </el-form-item>
            <el-form-item label="状态">
              <el-tag type="success">{{props.row.status}}</el-tag>
            </el-form-item>
            <el-form-item label="故障设备" v-if="props.row.device">
              <strong>名称：{{props.row.device.name}}</strong>
              <strong>编号：{{props.row.device.code}}</strong>
            </el-form-item>
            <el-form-item label="图片" style="position:absolute;right:1em;top:1em;width:400px">
              <img :src="'http://134.175.154.93:8888/group1/'+props.row.picture" alt="" style="width:100%">
            </el-form-item>
            <el-form-item label="报修人" v-if="props.row.user">
              {{props.row.user.realname}}
            </el-form-item>
            <div>
              <el-form-item label="处理日志"  > </el-form-item>
                <el-table :data="props.row.workers" v-if="props.row.workers" >
                  <el-table-column label="序号" type="index" :index="1"></el-table-column>
                  <el-table-column label="记录" prop="handlerContent"></el-table-column>
                  <el-table-column label="时间" prop="handlerTime">
                    <template slot-scope="s">
                      {{s.row.handlerTime | fmtDate}}
                    </template>
                  </el-table-column>
              </el-table>
            </div>
          </el-form>
        </template>
      </el-table-column>
      <el-table-column label="序号" type="index" :index="1" fixed="left"></el-table-column>
      <!-- <el-table-column label="报修地点" prop="repairsAddress" width="140" ></el-table-column>
      <el-table-column label="报修时间" prop="repairsTime" width="160" >
         <template slot-scope="scope">
          {{scope.row.repairsTime | fmtDate}}
        </template>
      </el-table-column> -->
      <el-table-column label="报修信息" prop="introduce" min-width="200"></el-table-column>
      <el-table-column label="报修状态"  width="120" align="center">
        <template slot-scope="scope">
          <el-tag type="success">{{scope.row.status}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="评论" prop="comment" min-width="280"></el-table-column>
      <el-table-column label="评论时间" prop="commentTime" width="160" >
         <template slot-scope="scope">
          {{scope.row.commentTime | fmtDate}}
        </template>
      </el-table-column>
      <el-table-column label="操作" width="100" align="center" fixed="right">
        <template slot-scope="scope">
          <!-- <el-button type="text" size="mini" @click="toDeleteHandler(scope.row)">删除</el-button> -->
          <el-button type="text" size="mini" :disabled="!(scope.row.status === '已完成')" @click="toCommentHandler(scope.row)">评论</el-button>
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
    <Briupdrawer 
      @on-change-visible="handlerVisibleChange"
      :drawerVisible="visible" :title="title"  width='40%'>
      <div slot="content">
        <el-form :model="form" ref="repairsForm" :rules="rules" label-width="80px">
          <el-form-item label="评论信息" prop="introduce">
            <el-input type="textarea" v-model="form.comment"></el-input>
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
import { mapGetters } from 'vuex'
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
      form:{},
      fileList:[],
      rules:{
        name: [
          { required: true, message: '请输入地址信息', trigger: 'blur' }
        ],
        introduce: [
          { required: true, message: '请输入描述信息', trigger: 'blur' }
        ]
      }
    }
  },
  created(){
    this.loadRepairs();
  },
  computed:{
    ...mapGetters(["user"])
  },
  methods:{
    //下载
    submitHandler(){
      this.form.userId = this.user.id;
      this.$refs['repairsForm'].validate((valid) => {
        if (valid) {
          let url = '/repairs/comment'
          post(url,this.form).then(response => {
            this.$message({type:"success",message:response.message})
            this.loadRepairs();
            this.visible = false;
          })
        } else {
          return false;
        }
      });
    },
    // 上传成功
    uploadSuccessHandler(response){
      if(response.status == 200){
        this.$set(this.form,'picture',response.data.id);
      } else {
        this.$message({type:'error',message:'附件服务器异常！'})
        this.visible = false;
      }
    },
    // 去修改
    toCommentHandler(row){
      this.title = "评论"
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
    // 当前页发生改变
    currentChangeHandler(page){
      this.param.page = page;
      this.loadRepairs();
    },
    handlerVisibleChange(val){
      this.visible = val;
      this.$refs['repairsForm'].clearValidate();
    }
  }
}
</script>