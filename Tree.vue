<template>
  <ul class="tree">
    <li
      class="tree-node"
      v-for="(node, index) in data"
      :key="node[defaultProps.label]"
    >
      <i
        v-if="node[defaultProps.children]"
        class="iconfont"
        :class="{
          'icon-right': showChildren[index],
          'icon-down': !showChildren[index],
        }"
      />
      <span class="node-label" @click="handleClick(index)">{{
        node[defaultProps.label]
      }}</span>
      <!-- 使用keep-alive保持子组件之前的展开/隐藏状态 -->
      <keep-alive>
        <base-tree
          v-if="showChildren[index] && node[defaultProps.children]"
          :data="node[defaultProps.children]"
        />
      </keep-alive>
    </li>
  </ul>
</template>
<script>
export default {
  name: "base-tree", // 在子组件中加了name选项，便可以在自身递归使用
  props: {
    data: {
      type: Array,
      require: true,
    },
    defaultProps: {
      // 提高组件传值的健壮性，若父组件传值非预想字段名，则按照此特性来传递一个对象告诉子组件
      type: Object,
      default: () => ({
        label: "label",
        children: "children",
      }),
    },
  },
  data() {
    return {
      showChildren: [],
    };
  },
  methods: {
    handleClick(index) {
      this.$set(this.showChildren, index, !this.showChildren[index]);
    },
  },
  created() {},
};
</script>
<style scoped>
@import url("./assets/font.css");
li {
  list-style: none;
}

.tree-node {
  cursor: pointer;
}

.tree-node .iconfont {
  color: #c0c4cc;
  font-size: 12px;
  margin-right: 5px;
  vertical-align: middle;
}

.tree-node .node-label {
  font-size: 14px;
  color: #606266;
  vertical-align: middle;
}
</style>