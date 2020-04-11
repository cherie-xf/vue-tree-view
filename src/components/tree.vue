<template>
  <div class="vu-tree">
    <tree-node
      v-for="(item, i) in treeData"
      :key="i"
      :node="item"
      visible
      :multiple="multiple"
      :show-checkbox="showCheckbox"
      :check-directly="checkDirectly"
    ></tree-node>
    <div class="vu-tree-empty" v-if="!treeData.length">{{ emptyText }}</div>
  </div>
</template>
<script>
import treeNode from "./treeNode.vue";
export default {
  name: "vuTree",
  components: {
    treeNode
  },
  props: {
    data: {
      type: Array,
      default() {
        // return [];
        return [
          {
            title: "parent 1",
            expand: true,
            children: [
              {
                title: "parent 1-1",
                expand: true,
                children: [
                  { title: "leaf 1-1-1", checked: true },
                  { title: "leaf 1-1-2" }
                ]
              },
              {
                title: "parent 1-2",
                expand: true,
                children: [
                  { title: "leaf 1-2-1", selected: true },
                  { title: "leaf 1-2-1" }
                ]
              }
            ]
          },
          {
            title: "parent 2",
            children: [{ title: "leaf 2-1" }]
          },
          { title: "parent3", disabled: true }
        ];
      }
    },
    multiple: {
      type: Boolean,
      default: false
      //   default: true
    },
    showCheckbox: {
      type: Boolean,
      //   default: false
      default: true
    },
    checkDirectly:{
      type: Boolean,
      //   default: false
      default: true
    },
    emptyText: {
      type: String,
      default: "No Tree Data"
    }
  },
  data() {
    return {
      treeData: this.data
    };
  },
  watch: {
    data: {
      deep: true,
      handler() {
        this.treeData = this.data;
        this.flatedData = this.flatTree(this.treeData);
        this.rebuildTree();
      }
    }
  },
  methods: {
    flatTree(data) {
      let attachChildren = (node, acc = [], parent) => {
        let nodeKey = acc.length;
        node.nodeKey = nodeKey;
        let res = {
          node: node,
          nodeKey: nodeKey,
          parent: parent !== undefined ? parent.nodeKey : undefined,
          children: node.children ? [] : undefined
        };
        if (parent) {
          acc[parent.nodeKey].children.push(node.nodeKey);
        }
        acc = [...acc, res];
        if (node.children) {
          return node.children.reduce((acc, child) => {
            return attachChildren(child, acc, node);
          }, acc);
        } else {
          return acc;
        }
      };
      return data.reduce((resArr, rootNode) => {
        return attachChildren(rootNode, resArr);
      }, []);
    },
    // only run when prop: data changes
    rebuildTree() {
      const checkedNodes = this.getCheckedNodes(this.flatedData);
      const checkedKeys = checkedNodes.map(node => node.nodeKey);
      checkedKeys.forEach(nodeKey => {
        let node = this.flatedData[nodeKey].node;
        this.updatedTreeDown(node, {
          checked: true
        });
        let parentKey = this.flatedData[nodeKey].parent;
        if (parentKey === undefined) return;
        const parent = this.flatedData[parentKey].node;
        if (parent.checked !== node.checked) {
          this.updatedTreeUp(node.nodeKey);
        }
      });
    },
    // return whole flatedData
    updatedTreeUp(nodeKey = 0) {
      const parentKey = this.flatedData[nodeKey].parent;
      if (parentKey === undefined) return;
      const node = this.flatedData[nodeKey].node;
      const parent = this.flatedData[parentKey].node;
      if (
        node.checked === parent.checked &&
        node.indeterminate === parent.indeterminate
      ) {
        return; // no need to update
      }
      if (node.checked) {
        if (parent.children.every(node => node.checked)) {
          this.$set(parent, "checked", true);
        } else {
          this.$set(parent, "indeterminate", true);
        }
      } else {
        this.$set(parent, "checked", false);
        this.$set(parent, "indeterminate", parent.children.some(node => node.checked));
      }
      return this.updatedTreeUp(parentKey);
    },
    // only return the node
    updatedTreeDown(node, changes = {}) {
      Object.keys(changes).map(key => {
        this.$set(node, key, changes[key]);
      });
      if (node.children) {
        node.children.map(child => {
          this.updatedTreeDown(child, changes);
        });
      }
    },
    handleCheck({ checked, nodeKey }) {
      if (!this.flatedData[nodeKey]) return;
      let node = this.flatedData[nodeKey].node;
      this.$set(node, "checked", checked);
      this.$set(node, "indeterminate", false);
      this.updatedTreeDown(node, {
        checked: checked,
        indeterminate: false
      });
      this.updatedTreeUp(nodeKey);
      console.log("checked node", this.getCheckedNodes(this.flatedData));
    },
    handleSelect(nodeKey) {
      if (!this.flatedData[nodeKey]) return;
      let node = this.flatedData[nodeKey].node;
      if (!this.multiple) {
        // reset previously selected
        const currentSelectedKey = this.flatedData.findIndex(
          obj => obj.node.selected
        );
        if (currentSelectedKey >= 0 && currentSelectedKey !== nodeKey)
          this.$set(
            this.flatedData[currentSelectedKey].node,
            "selected",
            false
          );
      }
      this.$set(node, "selected", !node.selected);
      //TODO: on selected
      this.$emit(
        "on-select-change",
        this.getSelectedNodes(this.flatedData),
        node
      );
      console.log("selected node", this.getSelectedNodes(this.flatedData));
    },
    getCheckedNodes(flatedData) {
      return flatedData.filter(obj => obj.node.checked).map(obj => obj.node);
    },
    getSelectedNodes(flatedData) {
      return flatedData.filter(obj => obj.node.selected).map(obj => obj.node);
    },
    getCheckedAndIndeNodes(flatedData) {
      return flatedData
        .filter(obj => obj.checked || obj.indeterminate)
        .map(obj => obj.node);
    }
  },
  created() {
    this.flatedData = this.flatTree(this.treeData);
    this.flatedData = this.rebuildTree();
  },
  mounted() {
    this.$on("on-check", this.handleCheck);
    this.$on("on-selected", this.handleSelect);
    this.$on("toggle-expand", node => this.$emit("on-toggle-expand", node));
  }
};
</script>
