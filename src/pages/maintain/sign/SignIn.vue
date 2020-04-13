<template>
  <div class="sign_form">
    <div class="sign_info">
      <el-form label-width="90px" size="small">
        <el-row>
          <el-col :span="12">
            <el-form-item label="经度：">
              {{ form.longitude }}
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="经度：">
              {{ form.latitude }}
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item v-if="visible" label="当前地址：">
          {{ form.signLocation }}
        </el-form-item>
        <el-form-item v-else label="当前地址：">
          <el-input v-model="form.signLocation" placeholder="请手动输入地址" />
        </el-form-item>
      </el-form>
    </div>
    <!-- 百度地图 -->
    <BriupMap @loaded="loadedHandler" @onError="errorHandler" />
    <div class="btns" style="text-align:center;margin-top:1em;">
      <el-button type="primary" round size="small" @click="submitHandler">签到</el-button>
    </div>
  </div>
</template>
<script>
import { get, post } from '@/utils/request'
import _ from 'lodash'
import { mapGetters } from 'vuex'
export default {
  data() {
    return {
      form: {},
      visible: false
    }
  },
  mounted() {
    this.form = this.$route.query
  },
  computed: {
    ...mapGetters(['user'])
  },
  methods: {
    submitHandler() {
      if (this.form.signLocation) {
        const url = '/sign/saveOrUpdate'
        post(url, this.form).then(response => {
          this.$message({ type: 'success', message: response.message })
          this.$router.go(-1)
        })
      } else {
        this.$message({ type: 'error', message: '请手动输入地址' })
      }
    },
    // 加载地图回调函数
    loadedHandler(location) {
      this.visible = true
      this.form.userId = this.user.id
      this.form.longitude = location.longitude
      this.form.latitude = location.latitude
      this.$set(this.form, 'signLocation', location.address)
      this.form = _.clone(this.form)
    },
    errorHandler() {
      this.visible = false
    }
  }
}
</script>
