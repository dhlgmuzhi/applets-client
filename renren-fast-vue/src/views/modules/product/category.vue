<template>
  <div class="container">
    <el-tree
      :data="menu"
      node-key="catId"
      :props="defaultProps"
      :default-expanded-keys="expandedKey"
      :highlight-current="true"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button type="text" size="mini" @click="() => append(data)">
            Add
          </el-button>
          <el-button type="text" size="mini" @click="() => remove(node, data)">
            Delete
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: ``,
      dialogType: '', // edit,add
      dialogVisible: false,
      menu: [],
      category: {
        name: ``,
        icon: ``,
        productUnit: ``,
        parentCid: 0,
        catLevel: 0,
        sort: 0,
        showStatus: 1,
        catId: null
      },
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      currentNodeKey: ``,
      expandedKey: []
    }
  },
  created: function() {
    this.getMenus()
  },
  methods: {
    getMenus: function() {
      this.$http({
        url: this.$http.adornUrl(`/product/category/list/tree`),
        method: `get`
      }).then(({ data }) => {
        console.log('成功获取到菜单数据...', data)
        this.menu = data.data
      })
    },
    // 点击节点事件
    handleNodeClick(data) {
      console.log(data)
    },
    // 点击添加事件
    append(data) {
      this.dialogType = 'add'
      // 清空数据
      this.category.name = ``
      this.category.icon = ``
      this.category.productUnit = ``
      this.dialogVisible = true
      this.title = '添加分类'
      // 设置Pid
      this.category.parentCid = data.catId
    },
    // 点击修改事件
    edit(data) {
      this.dialogVisible = true
      this.dialogType = 'edit'
      this.title = '修改'
      this.category.name = data.name
      this.category.icon = data.icon
      this.category.productUnit = data.productUnit
      this.category.catId = data.catId
      this.category.parentCid = data.parentCid
    },
    // 点击删除事件
    remove(node, data) {
      var catIds = [data.catId]
      var expandedKey = data.parentCid
      this.$confirm(`是否删除【${data.name}】菜单?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          // 发送请求
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(catIds, false)
          }).then(({ data }) => {
            this.$message({
              message: '删除成功', // 提示内容
              type: 'success' // 消息类型
            })
            // 刷新列表
            this.getMenus()
            // 展开节点
            this.expandedKey = [expandedKey]
          })
        })
        .catch(err => {
          // 捕获异常
          console.log(err)
        })
    },
    // 保存操作
    addCategory() {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        // 保存之后刷新菜单列表，然后默认展开该节点
        this.getMenus()
        this.expandedKey = [this.category.parentCid]
      })
      this.dialogVisible = false
    },
    // 修改分类
    editCategory() {
      // 只提交需要修改的数据
      var { name, catId, productUnit, icon } = this.category

      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ name, catId, productUnit, icon }, false)
      }).then(({ data }) => {
        // 刷新列表
        this.getMenus()
        // 关闭对话框
        this.dialogVisible = false
        // 展开列表
        this.expandedKey = [this.category.parentCid]
        this.$message({
          message: '修改成功', // 提示内容
          type: 'success' // 消息类型
        })
      })
    },
    // 提交数据
    submitData() {
      if (this.dialogType === `add`) {
        this.addCategory()
      } else if (this.dialogType === `edit`) {
        this.editCategory()
      }
    }
  }
}
</script>

<style></style>
