<template>
   <div class="ebook">
       <title-bar :ifTitleAndMenuShow = "ifTitleAndMenuShow"></title-bar>
       <div class="read-wrapper">
           <div id="read"></div>
           <div class="mask">
               <div class="left" @click="prevPage"></div>
               <div class="center" @click="toggleTitleAndMenu"></div>
               <div class="right" @click="nextPage"></div>
           </div>
       </div>
       <menu-bar  :ifTitleAndMenuShow = "ifTitleAndMenuShow"
                  :fontSizeList = "fontSizeList"
                  :defaultFontSize = "defaultFontSize"
                  @setFontSize = "setFontSize"
                  ref="menuBar"
                  :themesList = "themesList"
                  :defaultTheme = "defaultTheme"
                  @setTheme = "setTheme"
                  :bookAvailable = "bookAvailable"
                  @onProgressChange = "onProgressChange"
                  :navigation = "navigation"
                  @jumpTo = "jumpTo"
       ></menu-bar>
   </div>
</template>

<script>
import TitleBar from '@/components/TitleBar.vue'
import MenuBar from '@/components/MenuBar.vue'
import Epub from 'epubjs'
global.ePub = Epub
const DOWNLOAD_URL = '/static/计算的本质.epub'
export default {
  components: {
    TitleBar,
    MenuBar
  },
  data () {
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
      themesList: [
        {
          name: 'default',
          style: {
            body: {
              'color': '#000', 'background': '#fff'
            }
          }
        },
        {
          name: 'eye',
          style: {
            body: {
              'color': '#000', 'background': '#ceeaba'
            }
          }
        },
        {
          name: 'gold',
          style: {
            body: {
              'color': '#000', 'background': 'rgb(241, 236, 226)'
            }
          }
        },
        {
          name: 'night',
          style: {
            body: {
              'color': '#fff', 'background': '#000'
            }
          }
        }
      ],
      defaultTheme: 0,
      // 图书是否处于可用状态
      bookAvailable: false,
      navigation: {}
    }
  },
  methods: {
    // 根据链接跳转到指定位置
    jumpTo (href) {
      this.rendition.display(href)
      this.hideTitleAndMenu()
    },
    hideTitleAndMenu () {
      // 隐藏标题栏和菜单栏
      this.ifTitleAndMenuShow = false
      // 隐藏菜单栏弹出的设置栏
      this.$refs.menuBar.hideSetting()
      // 隐藏目录
      this.$refs.menuBar.hideContent()
    },
    // progress 进度条的数值（0-100）
    onProgressChange (progress) {
      const percentage = progress / 100
      const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0
      this.rendition.display(location)
    },
    setTheme (index) {
      this.themes.select(this.themesList[index].name)
      this.defaultTheme = index
    },
    regiseterTheme () {
      this.themesList.forEach(theme => {
        this.themes.register(theme.name, theme.style)
      })
    },
    setFontSize (fontSize) {
      this.defaultFontSize = fontSize
      if (this.themes) {
        this.themes.fontSize(fontSize + 'px')
      }
    },
    toggleTitleAndMenu () {
      this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow
      if (!this.ifTitleAndMenuShow) {
        this.$refs.menuBar.hideSetting()
      }
    },
    prevPage () {
      // Rendtition.prev
      if (this.rendition) {
        this.rendition.prev()
      }
    },
    nextPage () {
      // Rendition.next
      if (this.rendition) {
        this.rendition.next()
      }
    },
    //    电子书解析与渲染
    showEpeub () {
      //    生成book
      this.book = new Epub(DOWNLOAD_URL)
      //    console.log(this.book)
      //    生成Rendition，通过book.renderTo
      this.rendition = this.book.renderTo('read', {
        width: window.innerWidth,
        height: window.innerHeight
      })
      // 通过Rendition.display来渲染电子书
      this.rendition.display()
      // 获取一个Theme对象
      this.themes = this.rendition.themes
      // 设置默认字体
      this.setFontSize(this.defaultFontSize)
      // this.themes.register(name, styles)
      // this.themes.select(name)
      this.regiseterTheme()
      this.setTheme(this.defaultTheme)
      // 获取locations对象
      // console.log(this.book.locations)
      // 通过epubjs的钩子函数来实现
      this.book.ready.then(() => {
        this.navigation = this.book.navigation
        return this.book.locations.generate()
      }).then(result => {
        // 保存locations对象
        this.locations = this.book.locations
        // 标记电子书解析完毕的状态
        this.bookAvailable = true
        // 测试：跳转到书中间部分
        // this.rendition.display(50)
      })
    }
  },
  mounted () {
    this.showEpeub()
  }
}
</script>

<style lang='scss' scoped>
@import 'assets/styles/global';
.ebook {
    position: relative;
    .read-wrapper {
        .mask {
            position: absolute;
            top: 0;
            left: 0;
            z-index: 100;
            display: flex;
            width: 100%;
            height: 100%;
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
