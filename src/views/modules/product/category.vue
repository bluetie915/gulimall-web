<template>
  <div>
    <el-tree
      node-key="catId"
      :data="menus"
      show-checkbox
      :props="defaultProps"
      :expand-on-click-node="false"
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
  </div>
</template>

<script>
export default {
  data() {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name',
      },
    }
  },
  created() {
    this.getMenus()
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
      }).then(({ data }) => {
        this.menus = data.entities
      })
    },
    append(data) {},

    remove(node, data) {
      var ids = [data.catId]
      this.$http({
        url: this.$http.adornUrl(`/product/category/delete`),
        method: 'post',
        data: this.$http.adornData(ids, false),
      }).then(({ data }) => {
        console.log('删除成功')
      })
    },
  },
}
</script>

<style>
</style>