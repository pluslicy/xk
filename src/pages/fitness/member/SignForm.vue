<template>
  <div class="sign_form">
    <el-button type="text" size="small" @click="toBackHandler">返回</el-button>
    <div>
        <el-row>
          <el-col :span="6"><strong>课程名称：</strong></el-col>
          <el-col :span="18">{{ form.courseName }}</el-col>
        </el-row>
        <el-row>
          <el-col :span="6"><strong>课程时间：</strong></el-col>
          <el-col :span="18">{{ form.courseDay }}, {{ form.courseTime }}</el-col>
        </el-row>
        <el-row>
          <el-col :span="6"><strong>经度：</strong></el-col>
          <el-col :span="18">{{ form.longitude }}</el-col>
        </el-row>
        <el-row>
          <el-col :span="6"><strong>维度：</strong></el-col>
          <el-col :span="18">{{ form.latitude }}</el-col>
        </el-row>
        <el-row>
          <el-col :span="6"><strong>地址：</strong></el-col>
          <el-col :span="18">{{ form.address }}</el-col>
        </el-row>
        <!-- 地图 -->
        <Location @loaded="loadedHandler" />
        <div class="btns" style="text-align:center;margin-top:1em;">
          <el-button type="primary" round size="small" @click="submitHandler">签到</el-button>
        </div>
      </div>
  </div>
</template>
<script>
import { get, post } from '@/utils/request'
import Location from '@/components/Fitness/Map/Location'

export default {
  data(){
    return {
      form:{}
    }
  },
  mounted(){
    this.form = this.$route.query;
  },
  components: {
    Location
  },
  methods:{
    toBackHandler(){
      this.$router.go(-1);
    },
    submitHandler() {
      const url = '/sign/in'
      post(url, this.form).then(response => {
        this.$message({ type: 'success', message: response.message })
        this.$router.go(-1)
      })
    },
     // 加载地图回调函数
    loadedHandler(location) {
      this.form.longitude = location.longitude
      this.form.latitude = location.latitude
      this.form.address = location.address
      this.form = _.clone(this.form)
    }
  }
}
</script>