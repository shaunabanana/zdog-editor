<template>
  <el-card class="property-editor" v-if="selection !== null">
    <el-form ref="form" label-width="100px" size="mini" :inline="true">
      <el-form-item label="Layer Name">
        <el-input v-model="selection.label" size="mini"> </el-input>
      </el-form-item>

      <el-collapse>
        <el-collapse-item
          v-for="group in properties"
          :key="group.name"
          :title="group.name"
          :name="group.name"
        >
          <el-form-item
            v-for="prop in group.props"
            :key="prop.name"
            :label="prop.name"
          >
            <!-- Number -->
            <el-input-number
              v-if="prop.type === 'number'"
              v-model="shapes[selection.id][prop.name]"
              :controls="false"
              size="mini"
              @change="updateProperty(selection.id, prop.name, $event)"
            >
            </el-input-number>

            <!-- Boolean -->
            <el-switch
              v-if="prop.type === 'boolean'"
              v-model="shapes[selection.id][prop.name]"
              @change="updateProperty(selection.id, prop.name, $event)"
            >
            </el-switch>

            <!-- Color String -->
            <el-color-picker
              v-if="
                prop.type === 'string' &&
                shapes[selection.id][prop.name].slice(0, 1) === '#'
              "
              v-model="shapes[selection.id][prop.name]"
              size="mini"
              @change="updateProperty(selection.id, prop.name, $event)"
            ></el-color-picker>

            <!-- Vector (not rotate!) -->
            <el-input-number
              v-if="prop.type === 'vector' && prop.name !== 'rotate'"
              v-model="shapes[selection.id][prop.name].x"
              :controls="false"
              size="mini"
              @change="updateProperty(selection.id, prop.name, { x: $event })"
            >
            </el-input-number>
            <el-input-number
              v-if="prop.type === 'vector' && prop.name !== 'rotate'"
              v-model="shapes[selection.id][prop.name].y"
              :controls="false"
              size="mini"
              @change="updateProperty(selection.id, prop.name, { y: $event })"
            >
            </el-input-number>
            <el-input-number
              v-if="prop.type === 'vector' && prop.name !== 'rotate'"
              v-model="shapes[selection.id][prop.name].z"
              :controls="false"
              size="mini"
              @change="updateProperty(selection.id, prop.name, { z: $event })"
            >
            </el-input-number>

            <!-- Vector Rotate -->
            <el-input-number
              v-if="prop.type === 'vector' && prop.name === 'rotate'"
              :value="degrees(shapes[selection.id][prop.name].x)"
              :controls="false"
              size="mini"
              @change="updateProperty(selection.id, prop.name, { x: $event })"
            >
            </el-input-number>
            <el-input-number
              v-if="prop.type === 'vector' && prop.name === 'rotate'"
              :value="degrees(shapes[selection.id][prop.name].y)"
              :controls="false"
              size="mini"
              @change="updateProperty(selection.id, prop.name, { y: $event })"
            >
            </el-input-number>
            <el-input-number
              v-if="prop.type === 'vector' && prop.name === 'rotate'"
              :value="degrees(shapes[selection.id][prop.name].z)"
              :controls="false"
              size="mini"
              @change="updateProperty(selection.id, prop.name, { z: $event })"
            >
            </el-input-number>
          </el-form-item>
        </el-collapse-item>
      </el-collapse>
    </el-form>
  </el-card>
</template>

<script>
export default {
  name: "PropertyEditor",
  props: {
    selection: Object,
    shapes: Object,
  },

  methods: {
    updateProperty(id, property, value) {
      this.$emit("update", id, property, value);
      if (typeof value === "object") {
        for (let key of Object.keys(value)) {
          if (!this.selection.props[property])
            this.selection.props[property] = {};
          if (property === "rotate") {
            this.selection.props[property][key] = this.radians(value[key]);
            this.shapes[id][property][key] = this.radians(value[key]);
          } else {
            this.selection.props[property][key] = value[key];
          }
        }
      } else {
        this.selection.props[property] = value;
      }

      if (this.shapes[id].updatePath) {
        this.shapes[id].updatePath();
      }
    },

    radians(value) {
      return (value * Math.PI) / 180;
    },

    degrees(value) {
      return (value * 180) / Math.PI;
    },
  },

  computed: {
    properties() {
      const skip = [
        "renderOrigin",
        "renderFront",
        "renderNormal",
        "sortValue",
        "isFacingBack",
        "renderCentroid",
      ];
      const result = [
        {
          name: "Transform",
          range: ["translate", "rotate", "scale", "origin"],
          props: [],
        },
        {
          name: "Shape and size",
          range: ["width", "height", "diameter", 'length', "quarters"],
          props: [],
        },
        {
          name: "Style",
          range: ["stroke", "color", "visible", "fill", "closed", "backface"],
          props: [],
        },
        { name: "Others", props: [] },
      ];

      if (!this.selection) return [];
      let shape = this.shapes[this.selection.id];
      let properties = [];
      for (let key of Object.keys(shape)) {
        if (skip.includes(key)) continue;
        if (typeof shape[key] === "object") {
          if (
            typeof shape[key].x === "number" &&
            typeof shape[key].y === "number" &&
            typeof shape[key].z === "number"
          ) {
            properties.push({
              name: key,
              type: "vector",
            });
          }
        } else if (typeof shape[key] !== "undefined") {
          if (key.slice(0, 1) === '_') {
            properties.push({
              name: key.slice(1),
              type: typeof shape[key],
            });
          } else {
            properties.push({
              name: key,
              type: typeof shape[key],
            });
          }
        }
      }

      for (let prop of properties) {
        for (let group of result) {
          if (group.range) {
            if (group.range.includes(prop.name)) {
              group.props.push(prop);
              break;
            }
          } else {
            group.props.push(prop);
          }
        }
      }

      for (let group of result) {
        group.props.sort((a, b) => {
          if (a.name === b.name) return 0;
          if (!group.range) {
            if (a.name < b.name) return -1;
            else if (a.name > b.name) return 1;
            else return 0;
          }

          let indexA = group.range.indexOf(a.name);
          let indexB = group.range.indexOf(b.name);
          if (indexA === -1 && indexB === -1) {
            if (a.name < b.name) return -1;
            else if (a.name > b.name) return 1;
            else return 0;
          } else if (indexA > -1 && indexB === -1) {
            return -1;
          } else if (indexA === -1 && indexB > -1) {
            return 1;
          } else {
            return indexA - indexB;
          }
        });
      }
      console.log(result);
      return result;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.property-editor {
  max-height: 80vh;
  overflow-y: scroll;
}

.el-input-number {
  width: 4rem;
}
</style>
