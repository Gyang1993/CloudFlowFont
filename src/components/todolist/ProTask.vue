<template>
  <div class="taskClass">
    <div slot="header">

      <!--<el-input-->
      <!--  v-model="addTodolist.todoTask"-->
      <!--  class="addInput"-->
      <!--  prefix-icon="el-icon-plus"-->
      <!--  clearable-->
      <!--  placeholder="添加待办事项"-->
      <!--  @keydown.enter.native="showAddTodoTask"-->
      <!--/>-->
      <el-button
        icon="el-icon-plus"
        style="float: right; margin-bottom: 10px;margin-top: 10px"
        width="100%"
        @click="showAddTodoTask"
      >
        添加待办事项
      </el-button>
    </div>
    <div>
      <el-table
        :data="todolists"
        style="width: 100%"
        :default-expand-all="isExpand"
      >
        <el-table-column type="expand">
          <template slot-scope="props">
            <el-form label-position="left" class="table-expand">
              <el-form-item label="待办详情">
                <el-divider />&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp
                <span>{{ props.row.taskDetails }}</span>
                <el-divider />
              </el-form-item>
              <el-form-item label="任务分数">
                <span>{{ props.row.taskScore }}</span>
              </el-form-item>
            </el-form>
          </template>
        </el-table-column>
        <el-table-column
          prop="todoTask, planTime"
          label="待办事项"
        >
          <template slot-scope="scope">
            <el-tag
              color="white"
              style="color: #000000"
            >
              {{ scope.row.planTime }}
            </el-tag>
            &nbsp&nbsp&nbsp{{ scope.row.todoTask }}
            <!--<el-button style="border-color: white" @click="displaySubBond">-->
            <!--  {{ scope.row.todoTask }}-->
            <!--</el-button>-->
          </template>
        </el-table-column>
        <el-table-column
          prop="isProcess"
          label="状态"
          width="80"
        >
          <!--:filters="[{ text: '进行中', value: '进行中' }, { text: '逾期', value: '逾期' }]"-->
          <!--:filter-method="filterTag"-->
          <!--filter-placement="bottom-end"-->
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.taskStatusID === '逾期' ? 'danger' : ''"
            >
              {{ scope.row.taskStatusID }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="150"
        >
          <template slot-scope="scope">
            <el-button title="完成" icon="el-icon-check" style="padding:8px" @click="completeTask(scope.row)" />
            <el-button title="编辑" icon="el-icon-edit" style="padding:8px;background: #0e57a2;border-color: #0e57a2; color: #ffffff" @click="showEditTodoTask(scope.row)" />
            <el-button title="删除" icon="el-icon-close" style="padding:8px" type="danger" @click="deleteTask(scope.row)" />
          </template>
        </el-table-column>
      </el-table>
      <div style="margin-top: 10px">
        <el-pagination
          background
          layout="prev, pager, next, jumper, ->, total"
          :total="total"
          @current-change="currentChange"
          @size-change="sizeChange"
        />
      </div>
    </div>
    <div>
      <el-dialog
        class="addDialog"
        title="添加待办"
        style="text-align: left; font-weight: bold"
        :visible.sync="addDialogVisible"
        :close-on-press-escape="isFalse"
        :close-on-click-modal="isFalse"
        :show-close="isFalse"
        width="35%"
        height="100%"
        :before-close="handleClose"
      >
        <el-form ref="taskForm" :model="addTodolist">
          <el-form-item label="待办事项：" prop="addTodolist.todoTask">
            <el-input
              v-model="addTodolist.todoTask"
              placeholder="输入待办事项"
              prefix-icon="el-icon-edit"
              size="mini"
              width="100%"
            />
          </el-form-item>
          <el-form-item label="详细描述：" prop="addTodolist.taskDetails">
            <el-input
              v-model="addTodolist.taskDetails"
              type="textarea"
              :rows="10"
              placeholder="输入待办事项"
              prefix-icon="el-icon-edit"
              size="big"
              width="100%"
            />
          </el-form-item>
          <el-form-item label="执行人：" prop="addTodolist.userID">
            <el-select
              v-model="addTodolist.userID"
              clearable
              filterable
              default-first-option
              placeholder="请选择执行人"
            >
              <el-option
                v-for="item in users"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="计划完成时间:" prop="planTime">
            <el-date-picker
              v-model="addTodolist.planTime"
              size="mini"
              type="date"
              align="right"
              value-format="yyyy-MM-dd"
              :picker-options="pickerOptions"
              placeholder="选择日期"
            />
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="closeAddTask">取 消</el-button>
          <el-button type="primary" style="background: #0e57a2;border-color: #0e57a2" @click="addTodoTask">确 定</el-button>
        </span>
      </el-dialog>
    </div>
  </div>

</template>

<script>
export default {
  name: 'ProTask',
  data() {
    return {
      isFalse: false,
      isTrue: true,
      isExpand: false,
      pickerOptions: {
        disabledDate(time) {
          return time.getTime() < Date.now() - 3600 * 1000 * 24
        },
        shortcuts: [{
          text: '今天',
          onClick(picker) {
            picker.$emit('pick', new Date())
          }
        }, {
          text: '明天',
          onClick(picker) {
            const date = new Date()
            date.setTime(date.getTime() + 3600 * 1000 * 24)
            picker.$emit('pick', date)
          }
        }, {
          text: '一周后',
          onClick(picker) {
            const date = new Date()
            date.setTime(date.getTime() + 3600 * 1000 * 24 * 7)
            picker.$emit('pick', date)
          }
        }, {
          text: '两周后',
          onClick(picker) {
            const date = new Date()
            date.setTime(date.getTime() + 3600 * 1000 * 24 * 14)
            picker.$emit('pick', date)
          }
        }]
      },
      todolists: [],
      addDialogVisible: false,
      type: '',
      addTodolist: {
        todoTask: '',
        taskScore: 0,
        postCount: 0,
        planTime: '',
        tagColor: '',
        taskStatusID: '进行中',
        taskDetails: ''
      },
      users: [],
      total: 0,
      currentPage: 1,
      size: 10
    }
  },
  mounted() {
    this.initTodolist()
    this.initUserID()
  },
  methods: {
    displaySubBond() {
      console.log('进入')
      this.active = !this.active
      if (this.active) {
        console.log('展开')
        this.isExpand = true
      } else {
        console.log('关闭')
        this.isExpand = false
      }
    },
    initUserID() {
      this.getRequest('/system/admin/').then(resp => {
        if (resp) {
          this.users = resp
        }
      })
    },
    closeAddTask() {
      this.addDialogVisible = false
      this.addTodolist = {
        todoTask: '',
        taskScore: 0,
        postCount: '',
        planTime: '',
        tagColor: '',
        taskStatusID: '进行中',
        taskDetails: ''
      }
    },
    open1() {
      this.$message('这是一条消息提示')
    },
    changeStatus() {
      this.visible = true
    },
    initTodolist() {
      this.getRequest('/todolist/?currentPage=' + this.currentPage + '&size=' + this.size).then(resp => {
        if (resp) {
          this.todolists = resp.data
          this.total = resp.total
        }
      })
    },
    completeTask(data) {
      console.log(data.id)
      this.putRequest('/todolist/com/?id=' + data.id).then(resp => {
        if (resp) {
          this.initTodolist()
        }
      })
    },
    deleteTask(data) {
      this.putRequest('/todolist/del/?id=' + data.id).then(resp => {
        if (resp) {
          this.initTodolist()
        }
      })
    },
    showEditTodoTask(data) {
      this.title = '编辑待办信息'
      this.addTodolist = data
      this.addTodolist.taskStatusID = '进行中'
      this.addDialogVisible = true
      this.initTodolist()
    },
    showAddTodoTask() {
      this.initTodolist()
      this.addDialogVisible = true
    },
    addTodoTask() {
      console.log(this.addTodolist)
      if (this.addTodolist.id) {
        this.$refs['taskForm'].validate(valid => {
          if (valid) {
            this.putRequest('/todolist/', this.addTodolist).then(resp => {
              if (resp) {
                this.dialogVisible = false
                // this.addTodolist.todoTask = ''
                this.initTodolist()
                this.addDialogVisible = false
                this.addTodolist = {
                  todoTask: '',
                  taskScore: 0,
                  postCount: '',
                  planTime: '',
                  tagColor: '',
                  taskStatusID: '进行中',
                  taskDetails: ''
                }
              }
            })
          }
        })
      } else {
        this.$refs['taskForm'].validate(valid => {
          if (valid) {
            this.postRequest('/todolist/', this.addTodolist).then(resp => {
              if (resp) {
                this.addDialogVisible = false
                // this.addTodolist.todoTask = ''
                this.initTodolist()
                this.addDialogVisible = false
                this.addTodolist = {
                  todoTask: '',
                  taskScore: 0,
                  postCount: '',
                  planTime: '',
                  tagColor: '',
                  taskStatusID: '进行中',
                  taskDetails: ''
                }
              }
            })
          }
        })
      }
    },
    currentChange(currentPage) {
      this.currentPage = currentPage
      this.initTodolist()
    },
    sizeChange(size) {
      this.size = size
      this.initTodolist()
    },
    handleClose(done) {
      this.$confirm('是否以保存待办内容的方式关闭？如果不希望保存可点击“取消” ')
        .then(_ => {
          done()
        })
        .catch(_ => {})
    },
    filterTag(value, row) {
      return row.taskStatusID === value
    },
    filterHandler(value, row, column) {
      const property = column['property']
      return row[property] === value
    }
  }
}
</script>

<style scoped>
.scoreTab {
  margin-top: 20px;
}

.taskClass {
  /*position: relative;*/
}

.paginationClass {
  display: flex;
  margin-top: 10px;
}

/*.table-expand {*/
/*  !*font-size: 0;*!*/
/*  color: #000000;*/
/*  background: #fafafa;*/
/*  border: 1px solid #dcdfe6;*/
/*  box-shadow: 0 0 1px #cac6c6;*/
/*  border-radius:  4px;*/
/*  box-sizing: border-box;*/
/*}*/
.table-expand label {
  width: 90px;
  font-size: 15px;
  font-color: #99a9bf;
  font-weight: bold;
}
.table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  /*  width: 50%;*/
}

.addDialog /deep/.el-dialog {
  border-radius: 10px;
  /*box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2), 0 6px 20px 0 rgba(0,0,0,0.19);*/
}

.el-pagination.is-background .el-pager li:not(.disabled).active {
  background-color: #0e57a2;
  color: #fff;
}
</style>
