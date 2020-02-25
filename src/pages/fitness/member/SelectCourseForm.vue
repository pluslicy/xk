<template>
  <div class="select_course_form">
    <el-button type="text" @click="toBackHandler" size="small">返回</el-button>
    <CourseTbl @clickNode="clickNodeHandler" />
  </div>
</template>
<script>
import CourseTbl from '@/components/Fitness/CourseTbl'
import { get, post } from '@/utils/request'
import { mapState } from 'vuex'

export default {
  components: {
    CourseTbl
  },
  data(){
    return {

    }
  },
  computed: {
    ...mapState('user', ['userId', 'roles'])
  },
  methods:{
    toBackHandler(){
      this.$router.go(-1);
    },
    // 点击课表中的课程
    async clickNodeHandler(course) {
      const url = '/course/selectUserCourses'
      const response = await get(url, { userId: this.userId, courseId: course.id })

      if (response.data.length > 0) {
        this.$message({ type: 'warning', message: '这门课程你已经选择了' })
      } else {
        this.$alert('课程名称:' + course.name + ' 上课时间:' + course.courseDay + ',' + course.courseTime, '课程信息', {
          confirmButtonText: '确定选课',
          callback: action => {
            if(action === 'confirm'){
               // 选课
              const url_select = '/course/selectCourse'
              post(url_select, { userId: this.userId, courseId: course.id }).then(response => {
                this.$message({ type: 'success', message: response.message })
                // this.loadUserCourses()
              })
            }
          }
        })
      }
    }
  }
}
</script>