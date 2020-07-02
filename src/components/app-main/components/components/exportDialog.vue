<template>
  <el-dialog
    :visible="visible"
    append-to-body
    title="数据"
    @update:visible="$emit('update:visible', $event)"
    v-if="visible"
    width="600px"
  >
    <ele-form :formData="{ code: codeData }" :formDesc="codeFormDesc">
      <template v-slot:form-btn>
        <div style="text-align: center;">
          <el-button @click="handleCopyData" type="primary">复制数据</el-button>
        </div>
      </template>
    </ele-form>
  </el-dialog>
</template>

<script>
import copy from "clipboard-copy";
import serialize from "serialize-javascript";

export default {
  name: "exportDialog",
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    formDesc: {
      type: Object,
      default: () => ({})
    },
    formAttr: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      codeFormData: {},
      codeFormDesc: {
        code: {
          type: "json-editor",
          attrs: {
            options: {
              mode: "preview"
            }
          }
        }
      },
      codeData: Object.assign({}, this.formAttr, {
        formDesc: this.formDesc
      })
    };
  },

  methods: {
    handleCopyData() {
      copy(serialize(this.codeData, { space: 2 }));
      this.$message.success("复制成功!");
    },
    // 对数据进行转义 转成su-form的配置项
    setNewConfig(data) {
      let configList = [];
      Object.keys(Object.assign({}, data)).forEach(key => {
        let item = data[key];
        item.field = key;
        item.span = item.layout;
        delete item.layout;
        // 设置必填项
        if (this.checkProperty(item, "required")) {
          item.validate = "required";
          item.rules = [
            { required: true, message: `请输入${item.label}!`, trigger: "blur" }
          ];
          delete item.required;
        }
        // 设置组件属性配置
        if (this.checkProperty(item, "attrs")) {
          item.maxlength = item.attrs.maxlength;
          item.minlength = item.attrs.minlength;
          item.placeholder = item.attrs.placeholder;
          item.type = item.attrs.type || item.type || "input";
          item.min = Number(item.attrs.min) || null;
          item.max = Number(item.attrs.max) || null;
          item.precision = Number(item.attrs.precision) || 0;
          delete item.attrs;
        }

        // 设置select值
        if (this.checkProperty(item, "options")) {
          item.options.forEach(obj => {
            obj.label = obj.text;
            delete obj.text;
          });
        }
        configList.push(item);
      });
      this.codeData = configList;
    },
    //检查是否包含属性
    checkProperty(data, key) {
      return Object.prototype.hasOwnProperty.call(data, key);
    }
  },
  watch: {
    formDesc: {
      handler(newVal) {
        this.setNewConfig(newVal);
      },
      immediate: true
    }
  }
};
</script>
