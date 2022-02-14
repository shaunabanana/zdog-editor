<template>
  <el-card class="box-card">
    <el-input
      placeholder="Filter by name..."
      v-model="filterText"
      style="margin-bottom: 1em"
    >
    </el-input>

    <el-tree
      draggable
      ref="tree"
      :data="scene"
      :props="defaultProps"
      :filter-node-method="filterNode"
      :render-content="renderContent"
      :expand-on-click-node="false"
      :default-expand-all="true"
      :highlight-current="true"
      @node-drop="onNodeDrop"
      @node-click="$emit('select', $event)"
    ></el-tree>
  </el-card>
</template>

<script>
import { v4 as uuidv4 } from "uuid";
import CreateDropdown from "./CreateDropdown.vue"

export default {
  name: "TreeEditor",
  // components: {CreateDropdown},
  props: {
    scene: Array,
    shapes: Object,
  },

  data() {
    return {
      filterText: "",
      defaultProps: {
        id: "id",
        label: "label",
        type: "type",
        props: "props",
        children: "children",
      },
    };
  },

  watch: {
    filterText(val) {
      this.$refs.tree.filter(val);
    },
  },

  methods: {
    filterNode(value, data) {
      if (!value) return true;
      return data.label.indexOf(value) !== -1;
    },

    renderContent(h, { node, data }) {
      if (data.id === "ROOT") {
        return (
          <span class="custom-tree-node">
            <span>{node.label}</span>
            <span>
              <CreateDropdown onCommand={command => this.createNode(data, command)}/>
            </span>
          </span>
        );
      } else {
        return (
          <span class="custom-tree-node">
            <span>{node.label}</span>
            <span>
              <CreateDropdown onCommand={command => this.createNode(data, command)}/>
              <el-button
                size="mini"
                type="text"
                on-click={() => this.duplicateNode(node, data)}
                style="margin-left: 0.3em;"
              >
                <i class="el-icon-copy-document"></i>
              </el-button>
              <el-button
                size="mini"
                type="text"
                on-click={() => this.removeNode(node, data)}
                style="margin-left: 0.3em;"
              >
                <i class="el-icon-delete"></i>
              </el-button>
            </span>
          </span>
        );
      }
    },

    createNode (parent, type) {
      console.log(parent, type);
      parent.children.push({
        id: uuidv4(),
        label: type,
        type: type,
        props: {},
        children: []
      });
    },

    recursiveUpdateId(data) {
      data.id = uuidv4();
      for (let child of data.children) {
        this.recursiveUpdateId(child);
      }
    },

    duplicateNode (node, data) {
        const parent = node.parent;
        const children = parent.data.children || parent.data;
        // console.log(data, parent, children);
        let newChild = JSON.parse(JSON.stringify(data));
        this.recursiveUpdateId(newChild);
        children.push(newChild);
    },

    removeNode(node, data) {
        const parent = node.parent;
        const children = parent.data.children || parent.data;
        const index = children.findIndex(d => d.id === data.id);
        children.splice(index, 1);
    },

    onNodeDrop() {
      // if (location !== 'inner') return;
      // console.log(from.data.label)
      // let toShape = this.shapes[to.data.id];
      // console.log(this.getWorldTransform(from));
    },

    getPath(nodeId, root) {
      if (root.id === nodeId) return [nodeId];
      // let path = [root.id];
      for (let child of root.children) {
        let part = this.getPath(nodeId, child);
        if (part.length > 0) return [root.id].concat(part);
      }
      return [];
    },

    getWorldTransform(node) {
      let path;
      for (let element of this.scene) {
        path = this.getPath(node.data.id, element);
        if (path.length > 0) break;
      }
      if (path.length === 0)
        console.error(
          `Could not find a path to node ${node.data.id}. Is something wrong?`,
          node
        );
      console.log(path);
      // let shape = this.shapes[node.data.id];
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>
