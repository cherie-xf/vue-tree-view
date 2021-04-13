<template>
  <div class="tree-node-wrapper">
    <ul class="tree-node">
      <li>
        <div
          class="tree-node-arrow"
          :class="arrowClass"
          @click="handleExpand"
          v-if="showArrow"
        >
          <i>
            <img alt="Vue logo" src="../assets/arrowDown.png" />
          </i>
        </div>
        <div class="noarrow-placehold" v-if="!showArrow"></div>
        <span
          class="tree-node-checkbox"
          v-if="showCheckbox"
          :class="checkboxClass"
        >
          <label class="checkbox-label"></label>
          <input
            type="checkbox"
            :disabled="node.disabled"
            :checked="node.checked"
            @click="handleCheck"
          />
        </span>
        <span
          class="tree-node-title"
          @click="handleSelect"
          :class="selectedClass"
        >
          <template>{{ node.html || node.title }}</template>
        </span>
        <tree-node
          v-show="node.expand"
          v-for="(item, i) in node.children"
          :key="i"
          :node="item"
          :multiple="multiple"
          :show-checkbox="showCheckbox"
          :check-directly="checkDirectly"
        >
        </tree-node>
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  name: "treeNode",
  props: {
    node: {
      type: Object,
      default() {
        return {};
      }
    },
    multiple: {
      type: Boolean,
      default: false
    },
    showCheckbox: {
      type: Boolean,
      default: false
    },
    checkDirectly:{
      type: Boolean,
      default: false
    },
  },
  data() {
    return {};
  },
  computed: {
    showArrow() {
      return this.node.children && this.node.children.length;
    },
    checkboxClass() {
      if (this.node.disabled) {
        return "checkbox-disabled";
      }
      if (this.node.checked) {
        return "checkbox-checked";
      }
      if (this.node.indeterminate) {
        return "checkbox-inde";
      }
      return "";
    },
    arrowClass() {
      if (this.node.disabled) {
        return "arrow-disabled";
      }
      return this.node.expand ? "arrow-expand" : "";
    },
    selectedClass() {
      return this.node.selected ? "node-selected" : "";
    }
  },
  methods: {
    handleExpand() {
      if (this.node.children && this.node.children.length) {
        this.$set(this.node, "expand", !this.node.expand);
        this.dispatch("vuTree", "toggle-expand", this.node);
      }
    },
    handleSelect() {
      if (this.node.disabled) return;
      this.dispatch("vuTree", "on-selected", this.node.nodeKey);
    },
    handleCheck() {
      if (this.node.disabled) return;
      const changes = {
        checked: !this.node.checked && !this.node.indeterminate,
        nodeKey: this.node.nodeKey
      };
      this.dispatch("vuTree", "on-check", changes);
    },
    // for event pop up
    dispatch(componentName, eventName, params) {
      let parent = this.$parent || this.$root;
      let name = parent.$options.name;

      while (parent && (!name || name !== componentName)) {
        parent = parent.$parent;

        if (parent) {
          name = parent.$options.name;
        }
      }
      if (parent) {
        parent.$emit.apply(parent, [eventName].concat(params));
      }
    }
  }
};
</script>
<style lang="less" scoped>
.tree-node-wrapper {
  ul {
    list-style: none;
    margin: 0;
    padding: 0;
    li {
      list-style: none;
      margin: 8px 0;
      padding: 0;
      white-space: nowrap;
      outline: none;
    }
  }
  li {
    ul {
      margin: 0;
      padding: 0 0 0 18px;
    }
  }
  .tree-node-title {
    display: inline-block;
    margin: 0;
    padding: 0 4px;
    cursor: pointer;
    vertical-align: top;
    transition: all 0.2s ease-in-out;
    &:hover {
      background-color: lightblue;
    }
    &.node-selected,
    &.node-selected:hover {
      background-color: lightblue;
    }
  }
  .noarrow-placehold {
    display: inline-block;
    width: 20px;
  }
  .tree-node-arrow {
    display: inline-block;
    width: 20px;
    i {
      transition: all 0.2s ease-in-out;
      vertical-align: middle;
      //TODO
      img {
        width: 12px;
        transform: rotate(-90deg);
      }
    }
    &.arrow-disabled {
      cursor: default;
    }
    &.arrow-expand {
      i {
        transform: rotate(90deg);
        //TODO
        img {
          transform: rotate(0deg);
        }
      }
    }
  }
  .tree-node-checkbox {
    margin: 0 4px;
    display: inline-block;
    vertical-align: middle;
    cursor: pointer;
    line-height: 1;
    position: relative;
    input {
      position: absolute;
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
      bottom: 0;
      right: 0;
      z-index: 1;
      cursor: pointer;
      opacity: 0;
    }
    .checkbox-label {
      display: inline-block;
      width: 16px;
      height: 16px;
      position: relative;
      top: 0;
      left: 0;
      border: 1px solid #dcdee2;
      border-radius: 2px;
      background-color: #fff;
      transition: all 0.2s ease-in-out;
      &::after {
        box-sizing: border-box;
        content: "";
        display: table;
        height: 8px;
        width: 4px;
        position: absolute;
        top: 1px;
        left: 4px;
        border: 2px solid #fff;
        border-top: 0;
        border-left: 0;
        transform: rotate(45deg) scale(0);
        transition: all 0.2s ease-in-out;
      }
    }
    &.checkbox-disabled {
      cursor: not-allowed;
      input {
        cursor: not-allowed;
      }
      .checkbox-label {
        border-color: #dcdee2;
        background-color: #f3f3f3;
        &::after {
          border-color: #dcdee2;
        }
      }
    }
    &.checkbox-checked {
      .checkbox-label {
        border-color: #2d8cf0;
        background-color: #2d8cf0;
        &::after {
          top: 3px;
          left: 6px;
          transform: rotate(45deg) scale(1);
        }
      }
    }
    &.checkbox-inde {
      .checkbox-label {
        border-color: #2d8cf0;
        background-color: #2d8cf0;
        &::after {
          width: 10px;
          height: 1px;
          top: 7px;
          left: 3px;
          transform: scale(1);
        }
      }
    }
  }
}
</style>
