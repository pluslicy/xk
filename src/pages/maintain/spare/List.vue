<template>
  <div class="spare">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <el-col :span="18">
          <el-form :inline="true">
            <el-form-item label="名称">
              <el-input v-model="param.name"></el-input>
            </el-form-item>
            <el-button type="text" size="samll" @click="loadSpares">搜索</el-button>
          </el-form>
        </el-col>
        <el-col :span="6" style="text-align:right">
          <el-button type="primary" size="small" @click="toSaveHandler">新增</el-button>
        </el-col>
      </el-row>
      
    </div>
    <!-- 按钮 -->
    <!-- 表格 -->
    <el-table size="small" :data="spareData.list" v-loading="loading">
      <el-table-column label="序号" type="index" :index="1" fixed="left"></el-table-column>
      <el-table-column label="名称" prop="name" width="140" ></el-table-column>
      <el-table-column label="价格" prop="price" width="80"></el-table-column>
      <el-table-column label="参数" prop="parameters"  width="160"></el-table-column>
      <el-table-column label="库存" min-width="200">
        <template slot-scope="scope">
          <div v-if="scope.row.stock < 10">
             <el-tag type="danger" size="small">
              {{scope.row.stock}}
            </el-tag>
            <span style="color:red;font-size:12px">库存告急，请及时采购！</span>
          </div>
          <div v-else>
             {{scope.row.stock}}
          </div>
        </template>
      </el-table-column>
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
      :total="spareData.total" 
      :page-size="spareData.pageSize" 
      :current-page="spareData.page"
      @current-change="currentChangeHandler">
    </el-pagination>
    <!-- 分页 -->
    <!-- 模态框 -->
    <!-- <el-dialog :title="title" :visible.sync="visible" width="50%" @close="clearValidate" class="customer_modal"> -->
    <Briupdrawer 
      @on-change-visible="handlerVisibleChange"
      :drawerVisible="visible" :title="title"  width='40%'>
      <div slot="content">
        <el-form :model="form" ref="spareForm" :rules="rules" label-width="80px">
          <el-form-item label="名称" prop="name">
            <el-input v-model="form.name"></el-input>
          </el-form-item>
          <el-form-item label="参数" prop="parameters">
            <el-input v-model="form.parameters"></el-input>
          </el-form-item>
          <el-form-item label="价格" prop="price">
            <el-input v-model="form.price"></el-input>
          </el-form-item>
          <el-form-item label="数量" prop="stock">
            <el-input v-model="form.stock"></el-input>
          </el-form-item>
          <el-form-item label="介绍" prop="introduce">
            <el-input type="textarea" v-model="form.introduce"></el-input>
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
      title:"新增备件信息",
      param:{
        page:1,
        pageSize:15
      },
      spareData:{
        list:[]
      },
      spareTypeData:{
        list:[]
      },
      form:{},
      rules:{
        name: [
          { required: true, message: '请输入备件名称', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入备件编号', trigger: 'blur' }
        ]
      }
    }
  },
  created(){
    this.loadSpares();
  },
  methods:{
    submitHandler(){
      console.log(this.$refs['spareForm']);
      this.$refs['spareForm'].validate((valid) => {
        if (valid) {
          let url = '/spare/saveOrUpdate'
          post(url,this.form).then(response => {
            this.$message({type:"success",message:response.message})
            this.loadSpares();
            this.visible = false;
          })
        } else {
          return false;
        }
      });
    },
    // 去保存
    toSaveHandler(){
      this.title = "新增备件信息";
      this.form = {};
      this.visible = true;
    },
    // 去修改
    toEditHandler(row){
      this.title = "修改备件信息"
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
        let url = '/spare/deleteById'
        get(url,{id:row.id}).then(response =>{
          this.$message({ type: 'success', message: '删除成功!' });
          this.loadSpares();
        })
      })
    },
    // 查询备件
    loadSpares(){
      this.loading = true;
      let url = "/spare/pageQuery"
      get(url,this.param).then(response => {
        this.spareData = response.data;
        this.loading = false;
      })
    },
    // 当前页发生改变
    currentChangeHandler(page){
      this.param.page = page;
      this.loadSpares();
    },
    handlerVisibleChange(val){
      this.visible = val;
      this.$refs['spareForm'].clearValidate();
    }
  }
}
</script>