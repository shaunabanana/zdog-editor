<template>
  <div id="app">
    <SceneEditor
      class="backdrop"
      :scene="scene"
      @setup="shapes = {}"
      @new-shape="shapes[$event.id] = $event.shape"
    />
    <el-container class="mouse-disable">
      <el-header height="2rem">
        <Menu class="mouse-enable" @save="saveFile" @load="loadFile"/>
        <input ref="load" style="opacity: 0" type="file">
      </el-header>
      <el-main height="calc(100vh - 2rem)">
        <el-col :span="5">
          <TreeEditor class="mouse-enable" :scene="scene" :shapes="shapes" @select="selection = $event" />
        </el-col>
        <el-col :span="12">&nbsp;</el-col>
        <el-col :span="7">
          <PropertyEditor  class="mouse-enable" :shapes="shapes" :selection="selection"/>
        </el-col>
      </el-main>
    </el-container>
  </div>
</template>

<script>

import Menu from "./components/Menu.vue";
import TreeEditor from "./components/TreeEditor.vue";
import SceneEditor from "./components/SceneEditor.vue";
import PropertyEditor from "./components/PropertyEditor.vue";

export default {
  name: "App",
  components: {
    TreeEditor,
    SceneEditor,
    PropertyEditor,
    Menu,
  },

  data() {
    return {
      scene: [{
        id: 'ROOT',
        label: "Root",
        type: 'Anchor',
        props: {},
        children: []
      }],
      selection: null,
      shapes: {},
      title: 'Untitled',
    };
  },

  methods: {
    updateSelection(id, property, value) {
      if (typeof value === 'object') {
        for (let key of Object.keys(value)) {
          this.selection[property][value] = value[key];
        }
      } else {
        this.selection[property] = value;
      }
    },

    saveFile () {
      const data = JSON.stringify(this.scene)
      const blob = new Blob([data], {type: 'text/plain'})
      const e = document.createEvent('MouseEvents'),
      a = document.createElement('a');
      a.download = "zdog.json";
      a.href = window.URL.createObjectURL(blob);
      a.dataset.downloadurl = ['text/json', a.download, a.href].join(':');
      e.initEvent('click', true, false, window, 0, 0, 0, 0, 0, false, false, false, false, 0, null);
      a.dispatchEvent(e);
    },

    loadFile () {
      this.$refs.load.click();
      this.$refs.load.onchange = changeEvent => {
        console.log(changeEvent.target.files[0]);
        if (!changeEvent.target.files[0]) {
          return;
        }
        var reader = new FileReader();
        reader.onload = loadEvent => {
          var content = loadEvent.target.result;
          this.scene = JSON.parse(content);
        };
        reader.readAsText(changeEvent.target.files[0]);
      }
    }
  }
};
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #636;
  height: 100%;
}

.el-header {
  padding: 0px;
  margin-bottom: 2em;
}

.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 0;
}

.el-container, .el-header, .el-main {
  z-index: 100;
}

.mouse-disable {
  pointer-events: none;
}

.mouse-enable {
  pointer-events: auto;
}
</style>
