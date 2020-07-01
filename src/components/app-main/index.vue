<template>
  <multipane @paneResizeStop="handlePaneresize" class="app-main">
    <!-- 左侧菜单栏 -->
    <el-menu @select="handleChangeMenu" :default-active="activeMenu">
      <el-menu-item
        v-for="item of menus"
        :index="item.componentName"
        :key="item.componentName"
      >
        <i :class="item.icon"></i>
      </el-menu-item>
    </el-menu>
    <!-- 左侧菜单栏详情 isShowMenu控制显示隐藏 -->
    <transition name="el-fade-in-linear">
      <component
        :is="'app-main-left-' + activeMenu"
        :style="{ width: leftWidth }"
        key="menu"
        v-show="isShowMenu"
      />
    </transition>
    <!-- 框架的分割线 可拉伸当前模块宽度 -->
    <transition name="el-fade-in-linear">
      <multipane-resizer v-show="isShowMenu" key="resizer"></multipane-resizer>
    </transition>

    <!-- 右侧主要显示框 根据是否选择工程来显示 -->
    <template v-if="isSelectedForm">
      <div class="app-main-container">
        <!-- 功能栏 -->
        <app-main-header />
        <!-- 主题视图区域 -->
        <app-main-center class="app-main-content" />
      </div>
      <!-- 框架的分割线 -->
      <multipane-resizer></multipane-resizer>
      <!-- 最右侧配置项 组件通用配置 组件属性配置 表单配置 -->
      <app-main-right />
    </template>
    <!-- localStorage里面没有选择工程的时候显示 -->
    <template v-else>
      <div class="app-main-container not-selected">
        请先选择表单
      </div>
    </template>
  </multipane>
</template>

<script>
// 主要负责页面的布局和点击显示隐藏

import AppMainHeader from "./components/main-header.vue";
import AppMainCenter from "./components/main-center.vue";
import { Multipane, MultipaneResizer } from "vue-multipane";
import AppMainRight from "./components/main-right/index.vue";
import AppMainLeftProjects from "./components/main-left-projects.vue";
import AppMainLeftComponents from "./components/main-left-components.vue";
import { mapState } from "vuex";

export default {
  name: "AppMain",
  components: {
    AppMainCenter,
    AppMainHeader,
    AppMainRight,
    Multipane,
    MultipaneResizer,
    AppMainLeftComponents,
    AppMainLeftProjects
  },
  data() {
    return {
      leftWidth: "260px",
      // 默认激活
      activeMenu: "projects",
      // 点击切换
      isShowMenu: true,
      // 最左侧的menu数组
      menus: [
        {
          icon: "el-icon-document",
          componentName: "components"
        },
        {
          icon: "el-icon-menu",
          componentName: "projects"
        }
      ]
    };
  },
  computed: {
    ...mapState(["currentProjectIndex", "currentFormIndex"]),
    // 是否有选中的表单
    isSelectedForm() {
      // 如果当前工程索引和表单索引不为空，则表示当前选中某个表单 这两个均在localstorage保存着
      return (
        this.currentProjectIndex !== null && this.currentFormIndex !== null
      );
    }
  },
  methods: {
    // panel 拖拉变化
    handlePaneresize(el) {
      // multipane框架提供的可拖拉功能 因为右边不能放大宽度 所以需要做判断
      // 因为中间部分也可以拖拽，所以需要判断是否为左侧拖拽产生的变化
      if (el.className.startsWith("app-main-left")) {
        localStorage.setItem("menu-width", el.style.width);
      }
    },
    // 改变菜单
    handleChangeMenu(menuName) {
      // 点击菜单和当前菜单相同，且当前为显示模式，则隐藏
      // 否则则显示 isShowMenu控制右侧栏目的显示和隐藏
      if (menuName === this.activeMenu && this.isShowMenu) {
        this.isShowMenu = false;
      } else {
        this.activeMenu = menuName;
        this.isShowMenu = true;
      }

      // 将当前激活的菜单，存储到 localStorage
      localStorage.setItem("active-menu", menuName);
    },

    // 初始化左侧带单宽度
    initMenuWidth() {
      if (localStorage.getItem("menu-width")) {
        this.leftWidth = localStorage.getItem("menu-width");
      }
    },
    // 初始化当前激活的 menu
    initActiveMenu() {
      // 如果没有任何选择的表单，则菜单选择 projects
      if (!this.isSelectedForm) {
        this.activeMenu = "projects";
        // 否则从 localStorage 获取
      } else if (localStorage.getItem("active-menu")) {
        this.activeMenu = localStorage.getItem("active-menu");
      }
    }
  },
  mounted() {
    // 根据localstorage初始化左侧的框宽度
    this.initMenuWidth();
    // 根据localstorage初始化左侧的框默认勾选状态
    this.initActiveMenu();
  }
};
</script>

<style lang="scss">
.app-main {
  height: calc(100vh - 60px);
  overflow: hidden;
  margin-top: 2px;
  .el-menu-item {
    padding: 0 15px;
  }
  .app-main-content {
    height: calc(100% - 60px);
    overflow: scroll;
    padding-bottom: 20px;
    box-sizing: border-box;
    &::-webkit-scrollbar {
      width: 0;
    }
  }

  .app-main-container {
    flex-grow: 1;
    flex: 1;
    &.not-selected {
      text-align: center;
      color: #666;
      font-size: 14px;
      margin-top: 15%;
    }
  }

  & > .multipane-resizer {
    margin: 0;
    left: 0;
    position: relative;
    border-left: 1px solid #ebeef5;
    border-right: 1px solid #ebeef5;
    width: 7px;
    &::before {
      display: block;
      content: "";
      width: 1px;
      height: 40px;
      position: absolute;
      top: 50%;
      left: 50%;
      margin-top: -20px;
      margin-left: -1.5px;
      border-left: 1px solid #dcdfe6;
      border-right: 1px solid #dcdfe6;
      &::before {
        border-color: #999;
      }
    }
  }
}
</style>
