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
export default {}
</script>

<style>
</style>
