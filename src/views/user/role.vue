<template>
  <div class="dashboard-container">
    <el-form :model="tableData" label-width="80px" :inline="true" size="small">
      <el-form-item label="搜索名称">
        <el-input v-model="tableData.name" placeholder="请输入搜索信息" clearable="" />
      </el-form-item>

      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="getRoleList">搜索</el-button>
        <el-button icon="el-icon-plus" type="primary" size="mini" @click="handleCreateRole">新增</el-button>
      </el-form-item>
    </el-form>

    <el-table
      :data="tableData.list"
      style="width: 100%;"
      @sort-change="handleSortChange"
    >
      <el-table-column type="index" width="60" />
      <el-table-column prop="name" label="角色名称" sortable="custom" />
      <el-table-column prop="description" label="角色描述" sortable="custom" />
      <el-table-column prop="createTime" label="创建时间" sortable="custom" />
      <el-table-column prop="updateTime" label="更新时间" sortable="custom" />
      <el-table-column label="操作">
        <template v-slot="scope">
          <el-button type="text" size="small" icon="el-icon-edit" @click="handleEdit(scope.row)">
            编辑</el-button>
          <el-button type="text" size="small" icon="el-icon-delete" style="color: red;" @click="handleDelete([scope.row.id])">
            删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      class="pagination"
      :current-page.sync="tableData.currentPage"
      :page-sizes="[10,20,30,40]"
      :page-size.sync="tableData.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="tableData.total"
      @size-change="startPagination"
      @current-change="startPagination"
    />

    <el-dialog :title="roleEditForm.id? '新增角色':'角色编辑'" :visible.sync="roleEditDialogVisible" width="50%">
      <el-form ref="roleEditForm" :model="roleEditForm" label-width="80px" :rules="roleEditFormRules">
        <el-form-item label="角色名称" prop="name">
          <el-input v-model="roleEditForm.name" />
        </el-form-item>
        <el-form-item label="角色描述" prop="description">
          <el-input v-model="roleEditForm.description" />
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="roleEditDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="addOrUpdateRole">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Role',
  data() {
    return {
      tableData: {
        searchContent: '',
        list: [
          {
            id: 1,
            description: '描述',
            createTime: '',
            updateTime: ''
          }
        ],
        currentPage: 1,
        pageSize: 10,
        total: 1
      },
      roleEditForm: {
        id: '',
        name: '',
        description: ''
      },
      roleEditFormRules: {
        name: [
          { required: true, trigger: 'blur', validator: this.validateName }
        ]
      },
      roleEditDialogVisible: false
    }
  },
  methods: {
    /**
     * 新增角色
     */
    handleCreateRole() {
      for (const key in this.roleEditForm) {
        this.roleEditForm[key] = ''
      }
      this.openRoleEditDialog()
    },
    /**
     * 打开角色编辑窗口
     */
    openRoleEditDialog() {
      this.roleEditDialogVisible = true
      this.$nextTick(() => { this.$refs.roleEditForm.clearValidate() })
    },
    /**
     * @param {Number} id
     * 删除角色
     */
    handleDelete(id) {
      this.$confirm('此操作将永久删除该角色,是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {

      }).catch(() => {
        this.$message.info('已取消删除')
      })
    },
    /**
     * 编辑角色
     */
    handleEdit(row) {
      // 将编辑项赋值到界面中
      for (const key in this.roleEditForm) {
        this.roleEditForm[key] = row[key]
      }
      this.openRoleEditDialog()
    }
  }
}
</script>
