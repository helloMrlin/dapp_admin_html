<template>
  <div class="app-container">
    <el-row :gutter="20">
      <!--侧边部门数据-->
      <el-col :xs="9" :sm="6" :md="4" :lg="4" :xl="4">
        <div class="head-container">
          <el-input v-model="deptName" clearable size="small" placeholder="输入部门名称搜索" prefix-icon="el-icon-search" class="filter-item"
            @input="getDeptDatas" />
        </div>

        <el-tree :data="deptDatas" :props="defaultProps" @node-click="handleNodeClick" />
      </el-col>
      <!--用户数据-->
      <el-col :xs="15" :sm="18" :md="20" :lg="20" :xl="20">
        <!--工具栏-->
        <div class="head-container">
          <div v-if="crud.props.searchToggle">
            <!-- 搜索 -->
            <el-input v-model="query.blurry" clearable size="small" placeholder="输入名称或者邮箱搜索" style="width: 200px;"
              class="filter-item" @keyup.enter.native="crud.toQuery" />
            <el-date-picker v-model="query.gmtCreate" :default-time="['00:00:00','23:59:59']" type="daterange"
              range-separator=":" size="small" class="date-item" value-format="yyyy-MM-dd HH:mm:ss" start-placeholder="开始日期"
              end-placeholder="结束日期" />
            <el-select v-model="query.enabled" clearable size="small" placeholder="状态" class="filter-item" style="width: 90px"
              @change="crud.toQuery">
              <el-option v-for="item in enabledTypeOptions" :key="item.key" :label="item.display_name" :value="item.key" />
            </el-select>
            <rrOperation :crud="crud" />
          </div>
          <crudOperation show="" :permission="permission" />
        </div>
        <!--表单渲染-->
        <el-dialog append-to-body :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0"
          :title="crud.status.title" width="570px">
          <el-form ref="form" :inline="true" :model="form" :rules="rules" size="small" label-width="80px">
            <el-form-item label="用户名" prop="username">
              <el-input v-model="form.username" />
            </el-form-item>
            <el-form-item label="电话" prop="phone">
              <el-input v-model.number="form.phone" />
            </el-form-item>
            <el-form-item label="昵称" prop="nickName">
              <el-input v-model="form.nickName" />
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input v-model="form.email" />
            </el-form-item>
            <el-form-item label="部门" prop="dept.id">
              <treeselect v-model="form.dept.id" :options="depts"   :load-options="getDepts"
               style="width: 172px;" placeholder="选择部门" @select="selectFun" />
            </el-form-item>
            <el-form-item label="岗位" prop="job.id">
              <el-select v-model="form.job.id" style="width: 172px" placeholder="请先选择岗位">
                <el-option v-for="(item, index) in jobs" :key="item.name + index" :label="item.name" :value="item.id" />
              </el-select>
            </el-form-item>
            <el-form-item label="谷歌验证" prop="googleKey" v-if="form.id==null">
              <el-input disabled v-model="form.googleKey" style="width: 172px;" placeholder="请先生成谷歌验证" />
              <el-button v-show="!isGoogle" @click="makeGoogleAuthKey" type="primary">生成</el-button>
            </el-form-item>
            <el-form-item label="性别">
              <el-radio-group v-model="form.sex" style="width: 178px">
                <el-radio label="男">男</el-radio>
                <el-radio label="女">女</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="状态">
              <el-radio-group v-model="form.enabled" :disabled="form.id === user.id">
                <el-radio v-for="item in dict.boolean_status" :key="item.id" :label="item.value">{{ item.label }}</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item style="margin-bottom: 0;" label="角色" prop="roles">
              <el-select v-model="form.roles" style="width: 437px" multiple placeholder="请选择" @remove-tag="deleteTag"
                @change="changeRole">
                <el-option v-for="item in roles" :key="item.name" :disabled="level !== 1 && item.level <= level" :label="item.name"
                  :value="item.id" />
              </el-select>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button type="text" @click="crud.cancelCU">取消</el-button>
            <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
          </div>
        </el-dialog>
        <!--表格渲染-->
        <el-table ref="table" v-loading="crud.loading" :data="crud.data" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
          <el-table-column :selectable="checkboxT" type="selection" width="55" />
          <el-table-column  :show-overflow-tooltip="true" prop="username" label="用户名" />
          <el-table-column  :show-overflow-tooltip="true" prop="nickName" label="昵称" />
          <el-table-column  prop="sex" label="性别" />
          <el-table-column  :show-overflow-tooltip="true" prop="phone" width="100" label="电话" />
          <el-table-column  :show-overflow-tooltip="true" width="125" prop="email" label="邮箱" />
          <el-table-column  :show-overflow-tooltip="true" width="110" prop="dept" label="部门 / 岗位">
            <template slot-scope="scope">
              <div>{{ scope.row.dept.name }} / {{ scope.row.job.name }}</div>
            </template>
          </el-table-column>
          <el-table-column  label="状态" align="center" prop="enabled">
            <template slot-scope="scope">
              <el-switch v-model="scope.row.enabled" :disabled="user.id === scope.row.id" active-color="#409EFF"
                inactive-color="#F56C6C" @change="changeEnabled(scope.row, scope.row.enabled)" />
            </template>
          </el-table-column>
          <el-table-column  :show-overflow-tooltip="true" prop="gmtCreate" width="140"
            label="创建日期">
            <template slot-scope="scope">
              <span>{{ parseTime(scope.row.gmtCreate) }}</span>
            </template>
          </el-table-column>
          <el-table-column v-permission="['admin','user:edit','user:del']" label="操作" width="125" align="center" fixed="right">
            <template slot-scope="scope">
              <udOperation :data="scope.row" :permission="permission" :disabled-dle="scope.row.id === user.id" />
            </template>
          </el-table-column>
        </el-table>
        <!--分页组件-->
        <pagination />
      </el-col>
    </el-row>
  </div>
</template>

<script>
  import crudUser from '@/api/system/user'
  import {
    makeGoogleAuthKey
  } from '@/api/system/user'
  import {
    isvalidPhone
  } from '@/utils/validate'
  import {
    getDepts
  } from '@/api/system/dept'
  import {
    getAll,
    getLevel
  } from '@/api/system/role'
  import {
    getAllJob
  } from '@/api/system/job'
  import CRUD, {
    presenter,
    header,
    form,
    crud
  } from '@crud/crud'
  import rrOperation from '@crud/RR.operation'
  import crudOperation from '@crud/CRUD.operation'
  import udOperation from '@crud/UD.operation'
  import pagination from '@crud/Pagination'
  import Treeselect from '@riophae/vue-treeselect'
  import {
    mapGetters
  } from 'vuex'
  import '@riophae/vue-treeselect/dist/vue-treeselect.css'

  let userRoles = []
  // crud交由presenter持有
  const defaultCrud = CRUD({
    title: '用户',
    url: 'api/users',
    crudMethod: { ...crudUser
    }
  })
  const defaultForm = {
    id:null,
    username: null,
    nickName: null,
    sex: '男',
    email: null,
    googleKey: null,
    enabled: 'false',
    roles: [],
    job: {
      id: null
    },
    dept: {
      id: null
    },
    phone: null
  }
  export default {
    name: 'User',
    components: {
      Treeselect,
      crudOperation,
      rrOperation,
      udOperation,
      pagination
    },
    mixins: [presenter(defaultCrud), header(), form(defaultForm), crud()],
    // 数据字典
    dicts: ['boolean_status'],
    data() {
      // 自定义验证
      const validPhone = (rule, value, callback) => {
        if (!value) {
          callback(new Error('请输入电话号码'))
        } else if (!isvalidPhone(value)) {
          callback(new Error('请输入正确的11位手机号码'))
        } else {
          callback()
        }
      }
      return {
        height: document.documentElement.clientHeight - 180 + 'px;',
        deptName: '',
        isGoogle: false,
        depts: [],
        deptDatas: [],
        jobs: [],
        level: 3,
        roles: [],
        defaultProps: {
          children: 'children',
          label: 'name'
        },
        permission: {
          add: ['admin', 'user:add'],
          edit: ['admin', 'user:edit'],
          del: ['admin', 'user:del']
        },
        enabledTypeOptions: [{
            key: 'true',
            display_name: '激活'
          },
          {
            key: 'false',
            display_name: '锁定'
          }
        ],
        rules: {
          username: [{
              required: true,
              message: '请输入用户名',
              trigger: 'blur'
            },
            {
              min: 2,
              max: 20,
              message: '长度在 2 到 20 个字符',
              trigger: 'blur'
            }
          ],
          nickName: [{
              required: true,
              message: '请输入用户昵称',
              trigger: 'blur'
            },
            {
              min: 2,
              max: 20,
              message: '长度在 2 到 20 个字符',
              trigger: 'blur'
            }
          ],
          email: [{
              required: true,
              message: '请输入邮箱地址',
              trigger: 'blur'
            },
            {
              type: 'email',
              message: '请输入正确的邮箱地址',
              trigger: 'blur'
            }
          ],
          googleKey: [{
            required: true,
            message: '谷歌验证不能为空',
            trigger: 'blur'
          }, ],
          roles: [{
            required: true,
            message: '请选择角色',
            trigger: 'blur'
          }, ],
          phone: [{
            required: true,
            trigger: 'blur',
            validator: validPhone
          }]
        }
      }
    },
    computed: {
      ...mapGetters([
        'user'
      ])
    },
    created() {
      this.$nextTick(() => {
        this.getDeptDatas()
        this.crud.toQuery()
        this.crud.msg.add = '新增成功，默认密码：123456'
      })
    },
    mounted: function() {
      const that = this
      window.onresize = function temp() {
        that.height = document.documentElement.clientHeight - 180 + 'px;'
      }
    },
    methods: {
      //获取谷歌验证码
      makeGoogleAuthKey() {
        makeGoogleAuthKey().then(res => {
          this.form.googleKey=res;
          this.isGoogle = true;
          this.$notify({
            title: '谷歌验证key获取成功',
            type: 'success',
            duration: 1500
          })
        })
      },
      changeRole(value) {
        userRoles = []
        value.forEach(function(data, index) {
          const role = {
            id: data
          }
          userRoles.push(role)
        })
      },
      [CRUD.HOOK.afterAddError](crud) {
        this.afterErrorMethod(crud)
      },
      [CRUD.HOOK.afterEditError](crud) {
        this.afterErrorMethod(crud)
      },
      afterErrorMethod(crud) {
        // 恢复select
        const initRoles = []
        userRoles.forEach(function(role, index) {
          initRoles.push(role.id)
        })
        crud.form.roles = initRoles
      },
      deleteTag(value) {
        userRoles.forEach(function(data, index) {
          if (data.id === value) {
            userRoles.splice(index, value)
          }
        })
      },
      beforeInit() {

        this.url = 'api/users'
        return true
      },
      // 新增与编辑前做的操作
      [CRUD.HOOK.afterToCU](crud, form) {
      this.isGoogle = false;
        this.getDepts()
        this.getRoles()
        this.getRoleLevel()
        this.getJobs()
        form.enabled = form.enabled.toString()
      },
      // 打开编辑弹窗前做的操作
      [CRUD.HOOK.beforeToEdit](crud, form) {
        this.getJobs(this.form.dept.id)
        userRoles = []
        const roles = []
        form.roles.forEach(function(role, index) {
          roles.push(role.id)
          // 初始化编辑时候的角色
          const rol = {
            id: role.id
          }
          userRoles.push(rol)
        })
        form.roles = roles
      },
      // 提交前做的操作
      [CRUD.HOOK.afterValidateCU](crud) {
        if (!crud.form.dept.id) {
          this.$message({
            message: '部门不能为空',
            type: 'warning'
          })
          return false
        } else if (!crud.form.job.id) {
          this.$message({
            message: '岗位不能为空',
            type: 'warning'
          })
          return false
        } else if (this.roles.length === 0) {
          this.$message({
            message: '角色不能为空',
            type: 'warning'
          })
          return false
        }
        crud.form.roles = userRoles
        return true
      },
      // 获取左侧部门数据
      getDeptDatas() {
        const sort = 'id,desc'
        const params = {
          sort: sort
        }
        if (this.deptName) {
          params['name'] = this.deptName
        }
        getDepts(params).then(res => {
          this.deptDatas = res.content
        })
      },
      // 获取弹窗内部门数据
      getDepts() {
        getDepts({
          enabled: true
        }).then(res => {
          this.depts = res.content
        })
      },

      // 切换部门
      handleNodeClick(data) {
        if (data.pid === 0) {
          this.query.deptId = null
        } else {
          this.query.deptId = data.id
        }
        this.crud.toQuery()
      },
      // 改变状态
      changeEnabled(data, val) {
        this.$confirm('此操作将 "' + this.dict.label.boolean_status[val] + '" ' + data.username + ', 是否继续？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          crudUser.edit(data).then(res => {
            this.crud.notify(this.dict.label.boolean_status[val] + '成功', CRUD.NOTIFICATION_TYPE.SUCCESS)
          }).catch(() => {
            data.enabled = !data.enabled
          })
        }).catch(() => {
          data.enabled = !data.enabled
        })
      },
      // 获取弹窗内角色数据
      getRoles() {
        getAll().then(res => {
          this.roles = res
        }).catch(() => {})
      },
      // 获取弹窗内岗位数据
      getJobs(id) {
        getAllJob(id).then(res => {
          this.jobs = res.content
        }).catch(() => {})
      },
      // 点击部门搜索对应的岗位
      selectFun(node, instanceId) {
        this.getJobs(node.id)
        this.form.job.id = null
      },
      // 获取权限级别
      getRoleLevel() {
        getLevel().then(res => {
          this.level = res.level
        }).catch(() => {})
      },
      checkboxT(row, rowIndex) {
        return row.id !== this.user.id
      }
    }
  }
</script>

<style scoped>
</style>
