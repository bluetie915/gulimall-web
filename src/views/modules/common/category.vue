<template>
  <div>
    <el-tree
      node-key="catId"
      :data="menus"
      :props="defaultProps"
      @node-click="nodeClick"
      ref="menuTree"
    >
    </el-tree>
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
        this.menus = data.data
      })
    },
    // 节点被点击
    nodeClick(data, node, component) {
      // 向父组件发送事件
      this.$emit('tree-node-click', data, node, component)
    },
  },
}
</script>

<style>
</style>