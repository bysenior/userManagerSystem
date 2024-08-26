<template>
  <div class="dashboard-container">
    <el-form :model="tableData" label-width="80px" :inline="true" size="small">
      <el-form-item label="用户名称">
        <el-input v-model="tableData.username" placeholder="请输入用户信息" />
      </el-form-item>
      <el-form-item label="选择时间">
        <el-date-picker
          v-model="tableData.minCreateTime"
          type="datetime"
          placeholder="起始时间"
          class="date-picker"
        />
        <el-date-picker
          v-model="tableData.maxCreateTime"
          type="datetime"
          placeholder="截止时间"
          class="date-picker"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="getUserList">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <div>
      <el-button type="primary" plain icon="el-icon-plus" size="mini" @click="handleCreateUser">新增</el-button>
      <el-button type="danger" plain icon="el-icon-delete" size="mini" @click="handleBatchDelete">删除</el-button>
      <el-button type="info" plain icon="el-icon-upload2" size="mini" @click="handleImportUser">导入</el-button>
    </div>

    <ul>
      <li v-for="user in filteredUsers" :key="user.id">
        {{ user.userName }} - {{ user.trueName }}
      </li>
    </ul>

    <el-table
      :data="tableData.paginatedData"
      style="width: 100%;"
      @selection-change="val => tableData.selection = val"
      @sort-change="handleSortChange"
    >
      <el-table-column type="index" width="60" />
      <el-table-column type="selection" width="50" />
      <el-table-column width="50">
        <template v-slot="scope">
          <img :id="'avatar-' + scope.row.id" class="table-avatar">
        </template>
      </el-table-column>

      <el-table-column prop="userName" label="用户名" sortable="custom" width="180" />
      <el-table-column prop="trueName" label="真实姓名" sortable="custom" width="180" />
      <el-table-column prop="roleList" label="角色" sortable="custom" width="180" />
      <el-table-column prop="createTime" label="创建时间" sortable="custom" width="200" />

      <el-table-column prop="status" label="是否激活" sortable="custom" width="100">
        <template v-slot="scope">
          <el-switch v-model="scope.row.status" :active-value="1" :inactive-value="0" @change="handleSwitch(scope.row)" />
        </template>
      </el-table-column>

      <el-table-column label="操作">
        <template v-slot="scope">
          <el-button type="text" size="small" icon="el-icon-edit" @click="handleEdit(scope.row)">
            编辑</el-button>
          <el-button type="text" size="small" icon="el-icon-delete" style="color: red;" @click="handleDelete(scope.row.id)">
            删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      class="pagination"
      :current-page.sync="tableData.currentPage"
      :page-sizes="[5,10,15,20]"
      :page-size.sync="tableData.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="tableData.list.length"
      @size-change="getUserList"
      @current-change="handlePageChange"
    />

    <el-dialog
      class="user-edit-dialog"
      :title="mode === 'edit'? '用户编辑':'新增用户'"
      :visible.sync="userEditDialogVisible"
      width="50%"
      top="6vh"
    >
      <el-form ref="userEditForm" status-icon :model="userEditForm" label-width="80px">
        <el-form-item label="用户名" prop="userName">
          <el-input v-model="userEditForm.userName" @input="handleInputUserName" />
        </el-form-item>
        <el-form-item label="真实姓名" prop="trueName">
          <el-input v-model="userEditForm.trueName" @input="handleInputTrueName" />
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="userEditForm.password" />
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="userEditForm.email" />
        </el-form-item>
        <el-form-item label="性别">
          <el-radio-group v-model="userEditForm.gender">
            <el-radio :label="0">男</el-radio>
            <el-radio :label="1">女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="地址">
          <el-input v-model="userEditForm.address" />
        </el-form-item>
        <el-form-item label="简介">
          <el-input v-model="userEditForm.introduction" />
        </el-form-item>
        <el-form-item label="电话">
          <el-input v-model="userEditForm.phone" />
        </el-form-item>

        <el-form-item label="角色" prop="roleIds">
          <el-select v-model="userEditForm.roleIds" multiple placeholder="请选择角色">
            <el-option v-for="role in allRoles" :key="role.id" :label="role.name" :value="role.id" />
          </el-select>
        </el-form-item>

        <el-form-item label="头像">
          <el-upload class="avatar-uploader" action="" :auto-upload="false" :show-file-list="false">
            <img v-if="avatarUploadData.url" :src="avatarUploadData.url" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon" />
          </el-upload>
          <el-button v-if="avatarUploadData.raw" size="mini">重置</el-button>
        </el-form-item>

      </el-form>

      <span slot="footer" class="dialog-footer">

        <el-button @click="userEditDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="addOrUpdateUser">确定</el-button>

      </span>
    </el-dialog>

  </div>
</template>

<script lang="ts">

export default {
  name: 'User',
  data() {
    return {
      tableData: {
        username: [],
        minCreateTime: '',
        maxCreateTime: '',
        list: [{
          id: 1,
          userName: 'ZhangSan',
          trueName: '张三',
          roleList: ['admin'],
          createTime: this.getCurrentDate(),
          status: true
        }
        ],
        // 存储分页后的数据
        paginatedData: [],
        // 拷贝当前的分页数据，以便做搜索
        resetContent: [],
        // 存储搜索数据
        searchContent: [],
        selection: '',
        currentPage: 1,
        pageSize: 10,
        total: 1
      },
      userEditForm: {
        id: '',
        userName: '',
        trueName: '',
        password: '',
        email: '',
        gender: '',
        address: '',
        introduction: '',
        phone: '',
        roleIds: []
      },
      currentEditRow: null,
      userEditDialogVisible: false,
      editForm: {},
      nowId: 3,
      mode: 'add',
      buttonType: 'edit',
      allRoles: [],
      avatarUploadData: {
        url: '',
        raw: false
      }
    }
  },
  created() {
    this.setPaginatedData()
  },
  methods: {
    /**
     * 获取当前时间
     */
    getCurrentDate() {
      const now = new Date()
      const year = now.getFullYear()
      const month = String(now.getMonth() + 1).padStart(2, '0')
      const date = String(now.getDate()).padStart(2, '0')
      const hours = String(now.getHours()).padStart(2, '0')
      const minutes = String(now.getMinutes()).padStart(2, '0')
      return `${year}-${month}-${date} ${hours}:${minutes}`
    },
    /**
     * 更改编辑用户名
     * @param value
     */
    handleInputUserName(value) {
      this.mode = 'edit'
      this.userEditForm.userName = value
      let index = -1
      for (var i = 0; i < this.tableData.list.length; i++) {
        if (this.tableData.list[i].trueName === this.userEditForm.trueName) {
          index = i
          break
        }
      }
      this.tableData.list[index].userName = this.userEditForm.userName
    },
    /**
     * 更改编辑真实姓名
     * @param value
     */
    handleInputTrueName(value) {
      this.userEditForm.trueName = value
      this.mode = 'edit'
      let index = -1
      for (var i = 0; i < this.tableData.list.length; i++) {
        if (this.tableData.list[i].userName === this.userEditForm.userName) {
          index = i
          break
        }
      }
      this.tableData.list[index].trueName = this.userEditForm.trueName
    },
    /**
     * 搜索指定用户名
     */
    filteredUsers() {
      for (var i = 0; i < this.tableData.list.length; i++) {
        if (this.tableData.list[i].userName === this.tableData.username) {
          this.tableData.searchContent.push(this.tableData.list[i])
        }
      }
    },
    /**
     *页面转换
     * @param newPage 新的页码
     */
    handlePageChange(newPage) {
      this.currentPage = newPage
      this.setPaginatedData()
    },
    /**
     * 分页
     */
    setPaginatedData() {
      const startIndex = (this.tableData.currentPage - 1) * this.tableData.pageSize
      const endIndex = startIndex + this.tableData.pageSize
      this.tableData.paginatedData = this.tableData.list.slice(startIndex, endIndex)
    },
    /**
     * 获取搜素结果列表
     */
    getUserList() {
      this.filteredUsers()
      this.tableData.resetContent = this.tableData.paginatedData
      this.tableData.paginatedData = this.tableData.searchContent
    },
    /**
     * 重置搜索内容
     */
    resetQuery() {
      this.tableData.userName = []
      this.tableData.minCreateTime = ''
      this.tableData.maxCreateTime = ''
      this.tableData.paginatedData = this.tableData.resetContent
      this.setPaginatedData()
      this.tableData.resetContent = []
      this.tableData.searchContent = []
    },
    /**
     * 创建用户
     */
    handleCreateUser() {
      this.buttonType = 'add'
      this.userEditDialogVisible = true
    },

    // 编辑用户信息
    handleEdit(row) {
      for (const key in this.userEditForm) {
        this.userEditForm[key] = row[key]
      }
      this.userEditDialogVisible = true
    },
    /**
     * 删除数据
     * @param userIds 行号
     */
    handleDelete(userIds) {
      this.$confirm('此操作将永久删除该用户,是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {

      }).finally(() => {
        const index = this.tableData.list.findIndex(item => item.id === userIds)
        if (index > -1) {
          this.tableData.list.splice(index, 1)
        }
        this.setPaginatedData()
      }).catch(() => {
        this.$message.info('操作成功')
      })
    },
    /**
     * 新增用户信息
     */
    addOrUpdateUser() {
      this.userEditDialogVisible = false
      if (this.buttonType === 'add') {
        this.tableData.list.push({
          userName: this.userEditForm.userName,
          trueName: this.userEditForm.trueName,
          createTime: this.getCurrentDate(),
          roleList: this.userEditForm.roleIds,
          status: true,
          id: this.nowId
        })
        this.nowId = this.nowId + 1
      }
      this.buttonType = 'edit'
      this.setPaginatedData()
    },
    handleSwitch(row) {

    },
    handleSortChange() {

    },
    handleImportUser() {

    },
    handleBatchDelete() {

    }

  }
}
</script>
<style scoped>
.date-picker{
  margin-right: 10px;
  width:160px;
}
</style>
