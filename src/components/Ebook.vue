<template>
  <div class="ebook">
    <title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
    <!-- 主体 -->
    <div class="read-wrapper">
      <div id="read"></div>
      <div class="mask">
        <div
          class="left"
          @click="prePage"
        ></div>
        <div
          class="center"
          @click="toogleMenu"
        ></div>
        <div
          class="right"
          @click="nextPage"
        ></div>
      </div>
    </div>
    <menu-bar
      :ifTitleAndMenuShow="ifTitleAndMenuShow"
      :fontSizeList="fontSizeList"
      :defaultFontSize="defaultFontSize"
      @setFontSize="setFontSize"
      :themeList="themeList"
      :defaultTheme="defaultTheme"
      @selectTheme="selectTheme"
      :bookAvailable="bookAvailable"
      @onProgressChange="onProgressChange"
      :navigation="navigation"
      @jumpTo="jumpTo"
      :bookProgress="bookProgress"
      ref="menuBar"
    ></menu-bar>
  </div>
</template>

<script>
import TitleBar from "@/components/TitleBar";
import MenuBar from "@/components/MenuBar";

import Epub from "epubjs";
// const DOWNLOAD_URL = "../static/深夜书屋.epub";
const DOWNLOAD_URL = "../static/我的僵尸老婆.epub";
// const DOWNLOAD_URL = "../static/2018_Book_AgileProcessesInSoftwareEngine.epub";
export default {
  components: {
    TitleBar,
    MenuBar
  },
  data() {
    return {
      ifTitleAndMenuShow: false,
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 }
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: "默认",
          style: {
            body: {
              // 主题字体颜色
              color: "#000",
              // 主题背景颜色
              background: "#fff"
            }
          }
        },
        {
          name: "护眼",
          style: {
            body: {
              color: "#000",
              background: "#ceeaba"
            }
          }
        },
        {
          name: "夜间",
          style: {
            body: {
              color: "#fff",
              background: "#000"
            }
          }
        },
        {
          name: "金黄",
          style: {
            body: {
              color: "#000",
              background: "#FFD700"
            }
          }
        }
      ],
      defaultTheme: 0,
      bookAvailable: false,
      navigation: null,
      bookProgress: 0
    };
  },
  methods: {
    // 电子书的解析与渲染
    showEpub() {
      // 生成book
      this.book = new Epub(DOWNLOAD_URL);
      // 生成Rendition,通过book的renderTo方法生成
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight
      });
      // 生成Rendition.display渲染电子书
      this.rendition.display();
      // 获取theme对象，用于设置字体
      this.themes = this.rendition.themes;
      this.themes.fontSize(this.defaultFontSize);
      // 注册主题与切换主题
      // this.themes.register(name,styles)
      this.registerTheme();
      // this.themes.select(name)
      this.selectTheme(0);
      // 通过epubJs的钩子函数获取locations
      this.book.ready
        .then(() => {
          return this.book.locations.generate();
        })
        .then(result => {
          this.navigation = this.book.navigation;
          this.locations = this.book.locations;
          this.bookAvailable = true;
        });
    },
    // 上一页
    prePage() {
      // 如果标题和菜单是显示的，先关闭
      if (this.ifTitleAndMenuShow) {
        this.toogleMenu();
      }
      // 调用epubjs的prev方法
      if (this.rendition) {
        this.rendition.prev().then(() => {
          this.updateProgress();
        });
      }
    },
    nextPage() {
      // 如果标题和菜单是显示的，先关闭
      if (this.ifTitleAndMenuShow) {
        this.toogleMenu();
      }
      // 调用epubjs的next方法
      if (this.rendition) {
        this.rendition.next().then(() => {
          this.updateProgress();
        });
      }
    },
    toogleMenu() {
      this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow;
      if (!this.ifTitleAndMenuShow) {
        this.$refs.menuBar.hideSetting();
      }
    },
    setFontSize(fontSize) {
      this.defaultFontSize = fontSize;
      if (this.themes) {
        // 设置字体
        this.themes.fontSize(fontSize + "px");
      }
    },
    // 注册主题
    registerTheme() {
      if (this.themes) {
        this.themeList.forEach(theme => {
          this.themes.register(theme.name, theme.style);
        });
      }
    },
    // 选择主题
    selectTheme(index) {
      this.defaultTheme = index;
      if (this.themes) {
        this.themes.select(this.themeList[index].name);
      }
    },
    // progress：进度条数值 0-100
    onProgressChange(progress) {
      const percentage = progress / 100;
      const location =
        percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0;
      this.rendition.display(location);
    },
    hideTitleAndMenu() {
      this.ifTitleAndMenuShow = false;
      this.$refs.menuBar.hideSetting();
      this.$refs.menuBar.hideContent();
    },
    jumpTo(href) {
      this.hideTitleAndMenu();
      this.rendition.display(href).then(() => {
        this.updateProgress();
      });
    },
    updateProgress() {
      const curLocation = this.rendition.currentLocation();
      const percentage = this.bookAvailable
        ? this.locations.percentageFromCfi(curLocation.start.cfi)
        : 0;
      this.bookProgress = Math.round(percentage * 100);
    }
  },
  mounted() {
    this.showEpub();
  }
};
</script>

<style lang="scss" scoped>
@import "../assets/styles/global";
.ebook {
  position: relative;
  .read-wrapper {
    .mask {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: 100;
      display: flex;
      .left {
        flex: 0 0 px2rem(100);
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 px2rem(100);
      }
    }
  }
}
</style>
