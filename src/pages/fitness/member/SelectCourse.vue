<template>
  <!-- 选课 -->
  <div class="member_select_course">
    <!-- 按钮 -->
    <div class="btns">
      <el-row>
        <el-col :span="12">
          <van-button size="mini" @click="toSelectCourseHandler">选课</van-button>
        </el-col>
        <el-col :span="12" style="text-align:right">
          <el-link type="primary">{{ view.title }}</el-link>
          <el-switch v-model="view.type" @change="switchChangeHandler" />
        </el-col>
      </el-row>

    </div>
    <!-- /按钮 -->
    <!-- 表格 -->
    <div v-if="view.type">
      <van-card
        v-for="c in courses"
        :key="c.id"
        :desc="c.description"
        :title="c.name"
      >
        <div slot="tags">
          <van-tag plain type="danger">{{ c.courseDay }}</van-tag>
          <van-tag plain type="danger">{{ c.courseTime }}</van-tag>
        </div>
        <div slot="footer">
          <van-button size="mini" type="danger" @click.prevent="deleteHandler(c)">删除</van-button>
        </div>
      </van-card>
    </div>
    <!-- /表格 -->
    <!-- 课表 -->
    <BaseCourseTbl v-else :data="courses" />
    <!-- /课表 -->
  </div>
</template>

<script>
import { get, post } from '@/utils/request'
import BaseCourseTbl from '@/components/Fitness/BaseCourseTbl'
import { mapState } from 'vuex'

export default {
  components: {
    BaseCourseTbl
  },
  data() {
    return {
      title: '课表选课',
      courses: [],
      view: {
        type: true,
        title: '表格视图'
      }
    }
  },
  computed: {
    ...mapState('user', ['userId', 'roles'])
  },
  created() {
    this.loadUserCourses()
  },
  
  methods: {
    loadUserCourses() {
      const url = '/course/selectUserCourses'
      get(url, { userId: this.userId }).then(response => {
        this.courses = response.data
      })
    },
    deleteHandler(row) {
      this.$confirm('该操作将永久删除这条数据，你确认删除？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const url = '/course/deleteUserCourse'
        get(url, { userId: this.userId, courseId: row.id }).then(response => {
          this.$message({ type: 'success', message: response.message })
          this.loadUserCourses()
        })
      })
    },
    toSelectCourseHandler() {
      this.$router.push({path:'/fitness/member/SelectCourseForm'})
    },
    switchChangeHandler(now) {
      if (now) {
        this.view.title = '表格视图'
      } else {
        this.view.title = '课表视图'
      }
    }
  }
}
</script>
<style scoped>
.btns {
  margin-bottom: .5em;
}
</style>
