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
    <!-- 示意图 -->
    <el-steps :active="3" align-center style="margin:1em 0">
      <el-step title="接单" description="系统派单后需要24小时内完成接单"></el-step>
      <el-step title="反馈" description="工人需要到报修地进行初步检查，确认受损设备，然后在系统中进行反馈，上报故障设备，申请备件，及发送增派请求"></el-step>
      <el-step title="维修" description="工人进行维修，完成后上传完工图片备案"></el-step>
    </el-steps>
    <!-- /示意图 -->
    <!-- 表格 -->
    <el-table size="small" :data="workerData.list" v-loading="loading">
      <el-table-column type="expand" fixed="left">
        <template slot-scope="props">
          <!-- 报修 -->
          <div v-if="props.row.repairs" style="border-bottom:1px solid #f4f4f4">
            <el-form label-width="80px">
              <el-form-item label="报修描述">
                {{props.row.repairs.introduce}}
              </el-form-item>
              <el-form-item label="报修照片">
                <div  style="width:300px;padding:.5em">
                  <img :src="'http://134.175.154.93:8888/group1/'+props.row.repairs.picture" alt="" style="width:100%">
                </div>
              </el-form-item>
            </el-form>
          </div>
          <!-- /报修 -->
          <!-- 维修 -->
          <div v-if="props.row.handlerContent">
            <el-form label-width="80px">
              <el-form-item label="报修描述">
                {{props.row.handlerContent}}
              </el-form-item>
              <el-form-item label="报修照片">
                <div  style="width:300px;padding:.5em">
                  <img :src="'http://134.175.154.93:8888/group1/'+props.row.picture" alt="" style="width:100%">
                </div>
              </el-form-item>
            </el-form>
          </div>
          <!-- 维修 -->
        </template>
      </el-table-column>
      <el-table-column label="序号" type="index" :index="1" fixed="left"></el-table-column>
      <el-table-column label="工单类型" prop="type" width="80" align="center" ></el-table-column>
      <el-table-column label="工单状态"  width="120" align="center">
        <template slot-scope="scope">
          <el-tag type="success">{{scope.row.status}}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="工单负责人" prop="user.realname" width="100" ></el-table-column>
      <el-table-column label="报修地点"  width="140" >
         <template slot-scope="scope">
           <div v-if=" scope.row.repairs">
          {{scope.row.repairs.repairsAddress}}
          </div>
        </template>
      </el-table-column>
       <el-table-column label="报修时间"  width="140" >
         <template slot-scope="scope">
           <div v-if=" scope.row.repairs">
          {{scope.row.repairs.repairsTime | fmtDate}}
          </div>
        </template>
      </el-table-column>
      <el-table-column label="工单处理描述" prop="handlerContent" min-width="100" ></el-table-column>
     
      <el-table-column label="操作" width="150" align="center" fixed="right">
        <template slot-scope="scope">
          <el-button v-if="scope.row.type ==='维修'" type="text" size="mini" :disabled="!(scope.row.status === '已派单')" @click="toAcceptHandler(scope.row)">接单</el-button>
          <el-button v-if="scope.row.type ==='维修'" :disabled="scope.row.status === '已反馈' ||scope.row.status === '已完成'" type="text" size="mini" @click="toCheckHandler(scope.row)">反馈</el-button>
          <el-button  :disabled="scope.row.status === '已完成'"  type="text" size="mini" @click="toRepairHandler(scope.row)">
            {{scope.row.type ==='维修'?'维修':'巡检'}}
          </el-button>
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
        <el-form :model="form" ref="workerForm" :rules="rules" label-width="80px">
          <!-- 反馈 -->
          <div v-if="title === '反馈报修问题'">
            <div class="btns">
              <el-button type="primary" size="mini" @click="toIncreaseWorkerHandler(form)">增派</el-button>
              <el-button type="primary" size="mini" @click="toApplyforSpareHandler(form)">申请备件</el-button>
            </div>

            <el-form-item label="受损设备" prop="deviceId">
              <el-select v-model="form.deviceId">
                <el-option v-for="device in deviceData.list" :key="device.id" :label="device.name" :value="device.id"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="反馈描述" prop="checkContent">
              <el-input type="textarea" v-model="form.checkContent"></el-input>
            </el-form-item>
          </div>
          <!-- /反馈 -->
          <!-- 维修 -->
          <div v-if="title === '提交维修结果'">
            <el-form-item label="维修日志" prop="handlerContent">
              <el-input type="textarea" v-model="form.handlerContent"></el-input>
            </el-form-item>
            <el-form-item label="上传图片" prop="photo">
            <el-upload
              class="upload-demo"
              action="http://localhost:8888/file/upload"
              :on-success="uploadSuccessHandler"
              :file-list="fileList">
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">文件大小不允许超过3M</div>
            </el-upload>
          </el-form-item>
          </div>
          <!-- /维修 -->
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button size="small" @click="visible = false">取消</el-button>
        <el-button size="small" type="primary" @click="submitHandler">确定</el-button>
      </span>
    </Briupdrawer>
    <!-- /抽屉 -->
    <!-- 增派工单模态框 -->
    <el-dialog title="增派工单" :visible.sync="visible_increase" width="50%" class="customer_modal">
      <div class="content">
        <div class="tips">当发现当前工单无法处理完成就需要增派工单，一个工单只能对应一个维修设备，维修多套设备需要有多个工单</div>
        <el-form label-width="100px">
          <el-form-item label="受损设备" prop="deviceId">
            <el-select v-model="form_increase.deviceId">
              <el-option v-for="device in deviceData.list" :key="device.id" :label="device.name" :value="device.id"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="反馈描述" prop="checkContent">
            <el-input type="textarea" v-model="form_increase.checkContent"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button size="small" @click="visible_increase = false">取 消</el-button>
        <el-button size="small" type="primary" @click="submitIncreaseWorkerHandler">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 增派工单模态框 -->
    <!-- 备件申请 -->
    <el-dialog title="申请备件" :visible.sync="visible_spare" width="50%" class="customer_modal">
      <div class="content">
        {{form_spare}}
        <el-form label-width="100px">
          <el-form-item label="申请备件" prop="spareId">
            <el-select v-model="form_spare.spareId">
              <el-option v-for="spare in spareData.list" :key="spare.id" :label="spare.name" :value="spare.id"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="申请数量" prop="spareNum">
            <el-input v-model="form_spare.spareNum"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button size="small" @click="visible_spare = false">取 消</el-button>
        <el-button size="small" type="primary" @click="submitApplyForSpareHandler">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 备件申请 -->
  </div>
</template>
<script>
import {get,post} from '@/utils/request'
import _ from 'lodash'
export default {
  data(){
    return {
      visible:false,
      visible_increase:false,
      visible_spare:false,
      form:{},
      form_increase:{},
      form_spare:{},
      workerData:{ list:[] },
      workerTypeData:{ list:[] },
      deviceData:{ list:[] },
      spareData:{list:[]},
      loading:false,
      title:"新增工单信息",
      param:{
        page:1,
        pageSize:15
      },
      rules:{
        name: [
          { required: true, message: '请输入工单名称', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入工单编号', trigger: 'blur' }
        ]
      },
    }
  },
  created(){
    this.loadWorker();
    this.loadDevices();
    this.loadSpares();
  },
  methods:{
    // 提交申请备件
    submitApplyForSpareHandler(){
      let url = '/spareUse/saveOrUpdate';
      post(url,this.form_spare).then(response => {
        this.$message({type:"success",message:response.message})
        this.visible_spare = false;
      })
    },
    // 申请备件
    toApplyforSpareHandler(worker){
      this.form_spare.userId = worker.userId;
      this.form_spare.workerId = worker.id;
      this.form_spare.userRealname = worker.user.realname;
      
      this.visible_spare = true;
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
    // 提交增派工单
    submitIncreaseWorkerHandler(){
      let url = '/worker/saveOrUpdate';
      post(url,this.form_increase).then(response => {
        this.$message({type:"success",message:response.message})
        this.visible_increase = false;
      })
    },
    // 增派工单
    toIncreaseWorkerHandler(form){
      this.form_increase = {};
      this.form_increase.repairsId = form.repairsId;
      this.form_increase.userId = form.userId;
      this.form_increase.parentId = form.id;
      this.visible_increase = true;
    },
    // 提交工单反馈及维修
    submitHandler(){
      this.$refs['workerForm'].validate((valid) => {
        if (valid) {
          let url = '/worker/saveOrUpdate';
          if(this.title === '反馈报修问题'){
            url = '/worker/check'
          } else if(this.title === '提交维修结果') {
            url = '/worker/repair'
          }
          
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
    // 维修
    toRepairHandler(row){
      this.title = "提交维修结果"
      this.form = _.clone(row);
      this.visible = true;
    },
    // 去反馈
    toCheckHandler(row){
      this.title = "反馈报修问题"
      this.form = _.clone(row);
      this.visible = true;
    },
    // 确认接单
    toAcceptHandler(row){
      this.$confirm('是否确认接单?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let url = '/worker/accept'
        get(url,{id:row.id}).then(response =>{
          this.$message({type:'success',message:response.message})
          this.visible = false;
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
     // 查询备件
    loadSpares(){
      let url = "/spare/pageQuery"
      get(url,{
        page:1,
        pageSize:100
      }).then(response => {
        this.spareData = response.data;
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