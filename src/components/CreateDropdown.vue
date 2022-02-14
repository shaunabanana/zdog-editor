<template>
  <el-dropdown trigger="click" @command="emitCommand">
    <span class="el-dropdown-link">
      <el-button size="mini" type="text"> <i class="el-icon-plus"></i> </el-button>
    </span>
    <el-dropdown-menu slot="dropdown">
      <el-dropdown-item command="Anchor"> Anchor </el-dropdown-item>
      <el-dropdown-item command="Group"> Group </el-dropdown-item>
      <el-dropdown-item command="Rect"> Rect </el-dropdown-item>
      <el-dropdown-item command="RoundedRect"> RoundedRect </el-dropdown-item>
      <el-dropdown-item command="Ellipse"> Ellipse </el-dropdown-item>
      <el-dropdown-item command="Polygon"> Polygon </el-dropdown-item>
      <el-dropdown-item command="Shape"> Shape </el-dropdown-item>
      <el-dropdown-item command="Box"> Box </el-dropdown-item>
      <el-dropdown-item command="Cylinder"> Cylinder </el-dropdown-item>
      <el-dropdown-item command="Hemisphere"> Hemisphere </el-dropdown-item>
      <el-dropdown-item command="Cone"> Cone </el-dropdown-item>
    </el-dropdown-menu>
  </el-dropdown>
</template>

<script>
export default {
  name: "CreateDropdown",
  props: {},

  methods: {
      emitCommand(command) {
          this.$emit('command', command);
      }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>

