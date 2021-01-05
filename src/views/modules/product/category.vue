<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button type="success" @click="batchSave" size="mini"
      >批量保存</el-button
    >
    <el-button type="danger" @click="batchDelete" size="mini"
      >批量删除</el-button
    >
    <el-tree
      node-key="catId"
      :data="menus"
      show-checkbox
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedkey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
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
      // 开启批量保存时要展开的父节点id
      pCid: [],
      // 是否开启拖拽
      draggable: false,
      // 修改的节点
      updateNodes: [],
      // 拖拽最大层级
      maxLevel: 0,
      // dialog标题
      title: '',
      // dialog类型 edit, append
      dialogType: '',
      // 表单绑定对象
      category: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: '',
        catId: null,
      },
      // 对话框显示属性
      dialogVisible: false,
      // 刷新之后默认打开列表
      expandedkey: [],
      // 三级分类属性名字
      defaultProps: {
        children: 'children',
        label: 'name',
      },
      // 菜单列表
      menus: [],
    }
  },
  created() {
    this.getMenus()
  },
  methods: {
    // 获取所有菜单
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
      }).then(({ data }) => {
        this.menus = data.entities
      })
    },
    batchDelete() {
      let catIds = []
      let checkedNodes = this.$refs.menuTree.getCheckedNodes()
      for (let i = 0; i < checkedNodes.length; i++)
        catIds.push(checkedNodes[i].catId)
      this.$confirm(`是否删除【${catIds}】菜单`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl(`/product/category/delete`),
            method: 'post',
            data: this.$http.adornData(catIds, false),
          }).then(({ data }) => {
            this.$message({
              type: 'success',
              message: '删除成功!',
            })
            // 刷新出新菜单
            this.getMenus()
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除',
          })
        })
    },
    batchSave() {
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false),
      }).then(({ data }) => {
        this.$message({
          type: 'success',
          message: '修改成功!',
        })
        // 刷新菜单
        this.getMenus()
        // 打开默认展开菜单
        this.expandedkey = this.pCid
        // 修改默认值
        this.updateNodes = []
        this.maxLevel = 0
      })
    },
    // 拖拽操作数据库
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // 1.获取当前节点最新的父节点id
      let pCid = 0
      let siblings = null
      // 如果是放在节点里面，则被放置节点的id即为最新父节点id
      if (dropType == 'inner') {
        pCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }
      // 如果是放在前后，则被放置节点与放置节点是兄弟关系，即同一个父节点
      else {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      }
      this.pCid.push(pCid)
      // 2.获取当前节点的最新顺序
      for (let i = 0; i < siblings.length; i++) {
        // 如果遍历的是当前正在拖拽的节点
        if (siblings[i].data.catId == draggingNode.data.catId) {
          let catLevel = draggingNode.level
          // 当前节点的层级发生变化
          if (siblings[i].level != draggingNode.level) {
            // 修改当前节点的层级
            catLevel = siblings[i].level
            // 修改子节点的层级
            this.updateChildNodeLevel(siblings[i])
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          })
        } else this.updateNodes.push({ catId: siblings[i].data.catId, sort: i })
      }
      // 3.获取当前节点的最新层级
    },

    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0)
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          })
          this.updateChildNodeLevel(node.childNodes[i])
        }
    },

    // 拖拽之后是否允许被放置
    allowDrop(draggingNode, dropNode, type) {
      // 被拖拽的当前节点以及要放置位置的父节点总层数不能大于3
      // 被拖拽的当前节点总层数
      this.countNodeLevel(draggingNode)
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1
      // 如果要放在目标节点里面，则判断目标节点所在位置的等级
      if (type == 'inner') return deep + dropNode.level <= 3
      // 如果要放在目标节点前后，则判断目标节点父节点所在位置的等级
      else return deep + dropNode.parent.level <= 3
    },
    countNodeLevel(node) {
      // 找到所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel)
            this.maxLevel = node.childNodes[i].level
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },
    // 提交dialog
    submitData() {
      if (this.dialogType == 'append') this.addCategory()
      if (this.dialogType == 'edit') this.editCategory()
    },
    edit(data) {
      console.log(data)
      this.dialogType = 'edit'
      this.title = '修改分类'
      this.dialogVisible = true
      // 此时应该发送请求获取最新数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
      }).then(({ data }) => {
        this.category.name = data.data.name
        this.category.catId = data.data.catId
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
        this.category.parentCid = data.data.parentCid
      })
    },
    editCategory() {
      var { catId, name, icon, productUnit } = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(({ data }) => {
        this.$message({
          type: 'success',
          message: '修改成功!',
        })
        // 关闭对话框
        this.dialogVisible = false
        // 刷新出菜单
        this.getMenus()
        // 设置默认展开的菜单
        this.expandedkey = [this.category.parentCid]
      })
    },
    append(data) {
      this.dialogType = 'append'
      this.title = '添加分类'
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      // 防止修改时表单留存
      this.category.name = ''
      this.category.catId = null
      this.category.icon = ''
      this.category.productUnit = ''
      this.category.sort = 0
      this.category.showStatus = 1
    },
    // 添加三级分类的方法
    addCategory() {
      this.$http({
        url: this.$http.adornUrl(`/product/category/save`),
        method: 'post',
        data: this.$http.adornData(this.category, false),
      })
        .then(() => {
          this.$message({
            type: 'success',
            message: '保存成功!',
          })
          // 关闭对话框
          this.dialogVisible = false
          // 刷新出菜单
          this.getMenus()
          // 设置默认展开的菜单
          this.expandedkey = [this.category.parentCid]
        })
        .catch(() => {})
    },
    // 删除三级分类的方法
    remove(node, data) {
      var ids = [data.catId]
      this.$confirm(`是否删除【${data.name}】`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl(`/product/category/delete`),
            method: 'post',
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              type: 'success',
              message: '删除成功!',
            })
            // 刷新出新菜单
            this.getMenus()
            // 删除之后当前菜单不关闭（设置默认展开菜单）
            this.expandedkey = [node.parent.data.catId]
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除',
          })
        })
    },
  },
}
</script>

<style>
</style>