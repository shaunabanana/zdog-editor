<template>
  <div>
    <el-menu
      mode="horizontal"
    >
      <el-menu-item index="1" @click="$emit('save')">
        <i class="el-icon-download"></i>Save
      </el-menu-item>
      <el-menu-item index="2" @click="$emit('load')">
        <i class="el-icon-upload2"></i>Load
      </el-menu-item>
    </el-menu>
    <div class="line"></div>
  </div>
</template>

<script>
export default {
  name: "Menu",
  props: {},

  data() {
    return {
      title: 'Untitled'
    };
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
