<template>
  <div class="main">
    <div class="insights">
      <div class="left">
        <div>
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span>通知</span>
            </div>
            <div>
              <el-input
                v-model="billboard.noticeContent"
                class="billboardText"
                type="textarea"
                :rows="2"
                overflow:hidden
                :autosize="{ minRows: 10, maxRows: 20}"
                show-word-limit
                clearable
                size="medium"
                style="font-size: 20px; text-align:left; font-family: 新宋体"
              />
            </div>
          </el-card>
        </div>
      </div>
      <div class="right">
        <el-tabs
          v-model="activeName"
          class="todolistTab"
          stretch
          type="border-card"
          @tab-click="handleClick"
        >
          <el-tab-pane label="任务清单" name="ProTask">
            <ProTask v-if="isProTaskUpdate" />
          </el-tab-pane>
          <el-tab-pane label="完成任务" name="ComTask">
            <ComTask v-if="isComTaskUpdate" />
          </el-tab-pane>
          <el-tab-pane label="删除任务" name="DelTask">
            <DelTask v-if="isDelTaskUpdate" />
          </el-tab-pane>
        </el-tabs>
      </div>
    </div>
  </div>

</template>

<script>
import ComTask from '../../components/todolist/ComTask'
import ProTask from '../../components/todolist/ProTask'
import DelTask from '../../components/todolist/DelTask'

export default {
  name: 'HomePage',
  components: {
    ComTask,
    ProTask,
    DelTask
  },
  data() {
    return {
      isFalse: false,
      isTrue: true,
      defaultOpen: 'preview',
      // toolbars: {
      //   bold: true,
      //   italic: true,
      //   header: true,
      //   underline: true,
      //   strikethrough: true,
      //   mark: true,
      //   superscript: true,
      //   subscript: true,
      //   quote: true,
      //   ol: true,
      //   ul: true,
      //   table: true,
      //   fullscreen: true,
      //   readmodel: true,
      //   help: true,
      //   undo: true,
      //   redo: true,
      //   alignleft: true,
      //   aligncenter: true,
      //   alignright: true,
      //   subfield: true
      // },
      isProTaskUpdate: true,
      isComTaskUpdate: false,
      isDelTaskUpdate: false,
      calendarDate: new Date(),
      activeName: 'ProTask',
      addDialogVisible: false,
      repoDialogVisible: false,
      finishDialogVisible: false,
      billboard: {
        noticeContent: '',
        createDate: ''
      }
    }
  },
  mounted() {
    this.notice()
  },
  methods: {
    notice() {
      this.getRequest('/billboard/').then(resp => {
        if (resp) {
          this.billboard.noticeContent = resp.noticeContent
          this.billboard.createDate = resp.createDate
        }
      })
    },
    handleClose(done) {
      this.$confirm('确认关闭？')
        .then(_ => {
          done()
        })
        .catch(_ => {})
    },
    handleClick(tab) {
      if (tab.name === 'ProTask') {
        this.isProTaskUpdate = true
        this.isComTaskUpdate = false
        this.isDelTaskUpdate = false
      } else if (tab.name === 'ComTask') {
        this.isProTaskUpdate = false
        this.isComTaskUpdate = true
        this.isDelTaskUpdate = false
      } else if (tab.name === 'DelTask') {
        this.isProTaskUpdate = false
        this.isComTaskUpdate = false
        this.isDelTaskUpdate = true
      }
    }
  }
}
</script>

<style scoped>
*{
  margin: 0;
  /*padding: 0;*/
  box-sizing: border-box;
}

.main {
  margin-top: 0.4rem
}

.main .insights {
  display: grid;
  grid-template-columns: 48% 50%;
  gap: 1rem
}

/* 如果你的 el-input type 设置成textarea ，就要用这个了 */
.billboardText>>>.el-textarea__inner {
  border: 0;
  resize: none;/* 这个是去掉 textarea 下面拉伸的那个标志，如下图 */
}

</style>
