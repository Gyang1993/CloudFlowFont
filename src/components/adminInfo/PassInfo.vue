<template>
  <div>
    <div>
      <el-menu class="userMenu" mode="horizontal" router @select="handleSelect">
        <el-menu-item index="/userInfo">个人信息</el-menu-item>
        <el-menu-item index="/passInfo">密码保护</el-menu-item>
      </el-menu>
    </div>
    <div class="passWordMenu">
      <el-form ref="ruleForm" :model="ruleForm" status-icon :rules="rules" label-width="100px" class="ruleForm">
        <el-form-item label="密保问题" prop="oldPass">
          <el-input v-model="ruleForm.passQuestion" autocomplete="off" />
        </el-form-item>
        <el-form-item label="密保答案" prop="pass">
          <el-input v-model="ruleForm.passAnswer" autocomplete="off" />
          <el-button
            type="primary"
            style="background: #0e57a2; border-color: #0e57a2; width: 100%; margin-top: 30px"
            @click="updatePasswordInfo('ruleForm')"
          >提交</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PassInfo',
  data() {
    var validatePassInfo = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请输入密保信息'))
      }
    }
    return {
      admin: null,
      ruleForm: {
        passQuestion: '',
        passAnswer: ''
      },
      rules: {
        passQuestion: [
          { validator: validatePassInfo, trigger: 'blur' }
        ],
        passAnswer: [
          { validator: validatePassInfo, trigger: 'blur' }
        ]
      }
    }
  },
  mounted() {
    this.initAdmin()
  },
  methods: {
    updatePasswordInfo(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.ruleForm.adminId = this.admin.id
          console.log(this.ruleForm)
          this.putRequest('/system/admin/passPro', this.ruleForm).then(resp => {
            if (resp) {
              console.log('设置密保成功')
            }
          })
        } else {
          console.log('设置密保失败')
          return false
        }
      })
    },
    handleSelect(index) {
      this.$router.push(index)
    },
    initAdmin() {
      this.getRequest('/admin/info/').then(resp => {
        this.admin = resp
        this.ruleForm.passQuestion = resp.passQuestion
        this.ruleForm.passAnswer = resp.passAnswer
        // 拷贝admin信息用于展示
        this.adminDisplay = Object.assign({}, this.admin)
      })
    }
  }
}
</script>

<style scoped>
.userMenu {
  display: flex;
  justify-content: flex-end;
}

.passWordMenu {
  margin-top: 20px;
  display: flex;
  justify-content: left;
}

.ruleForm {
  width: 30%;
}

/*.passWordMenu /deep/.el-form-item {*/
/*  border-radius: 10px;*/
/*  font-weight: bold;*/
/*  !*box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2), 0 6px 20px 0 rgba(0,0,0,0.19);*!*/
/*}*/

</style>
