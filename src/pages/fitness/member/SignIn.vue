<template>
  <!-- 选课 -->
  <div class="sign_id">
    <!-- /按钮 -->
    <!-- 表格 -->
    <van-card
      v-for="c in courses"
      :key="c.id"
      :desc="c.description"
      :title="c.name" >
      <div slot="tags">
        <van-tag plain type="danger">{{ c.courseDay }}</van-tag>
        <van-tag plain type="danger">{{ c.courseTime }}</van-tag>
      </div>
      <div slot="footer">
        <van-button size="mini" type="primary" 
        :disabled="c.courseDay !=getWeekDay() "
          @click="toSignIn(c)">签到</van-button>
      </div>
    </van-card>
    <!-- /表格 -->
  </div>
</template>

<script>
import { get, post } from '@/utils/request'
import { mapState } from 'vuex'
import moment from 'moment'
import { getWeekDay } from '@/utils/week'
import _ from 'lodash'

export default {
  data() {
    return {
      title: '',
      courses: [],
      form:{}
    }
  },
  created() {
    this.loadUserCourses()
  },
  mounted() { },
  computed: {
    ...mapState('user', ['userId', 'roles', 'name']),
    test() {
      const day = moment().format('YYYY-MM-DD')
      return moment(day + ' 10:00').diff(moment(day + ' 10:39'), 'minutes')
    }
  },
 
  methods: {
    moment,
    // 签到
    toSignIn(record) {
      const day = moment().format('YYYY-MM-DD')
      const course_start_time = record.courseTime.split('-')[0]
      const course_end_time = record.courseTime.split('-')[1]
      // 判断是否在签到时间范围内
      // if((10 > moment(day+" "+course_start_time).diff(moment().format("YYYY-MM-DD hh:mm"),'minutes')) && (0 < moment(day+" "+course_end_time).diff(moment().format("YYYY-MM-DD hh:mm"),'minutes'))){
      this.title = '签到'
      this.form.type = '签到'
      this.form.userId = this.userId
      this.form.userName = this.name
      this.form.courseId = record.id
      this.form.courseName = record.courseName
      this.form.courseName = record.name
      this.form.courseDay = record.courseDay
      this.form.courseTime = record.courseTime
      // this.visible = true
      this.$router.push({path:'/fitness/member/SignForm',query:this.form})
      // } else {
      //   this.$message({type:"warning",message:"不在签到时间范围内"})
      // }
    },
    // // 签退
    // toSignOut(record){
    //   this.title = "签退";
    //   this.visible = true;

    // },
    getWeekDay,
    loadUserCourses() {
      const url = '/course/selectUserCourses'
      get(url, { userId: this.userId }).then(response => {
        this.courses = response.data
      })
    }
  }
}
</script>
<style scoped>
.el-col{
  line-height: 1.5em
}
</style>
