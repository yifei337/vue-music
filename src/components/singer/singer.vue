<template>
  <div class="singer">
    <listview @select="selectSinger" :data="singers"></listview>
    <router-view></router-view>
  </div>
</template>

<script type="text/ecmascript-6">
  import {getSingerList} from 'api/singer'
  import {ERR_OK} from 'api/config'
  import Singer from 'common/js/singer'
  import Listview from 'base/listview/listview'
  import {mapMutations} from 'vuex' // 语法糖

  const HOT_NAME = '热门'
  const HOT_SINGER_LEN = 10
  export default {
    data() {
      return {
        singers: []
      }
    },
    created() {
      this._getSingerList()
    },
    methods: {
      // 根据当前点击的歌手id跳转到对应的页面
      selectSinger(singer) {
        this.$router.push({
          path: `/singer/${singer.id}`
        })
        this.setSinger(singer) // 设置setSinger的值为歌手id
      },
      // 获取歌手数据
      _getSingerList () {
        getSingerList().then((res) => {
          if (res.code === ERR_OK) {
            this.singers = this._normalizeSinger(res.data.list)
          }
        })
      },
      _normalizeSinger(list) {
        let map = {
          hot: {
            title: HOT_NAME,
            items: []
          }
        }
        // 获取热门歌手数据
        list.forEach((item, index) => {
          if (index < HOT_SINGER_LEN) {
            map.hot.items.push(new Singer({
              id: item.Fsinger_mid,
              name: item.Fsinger_name
            }))
          }
          const key = item.Findex // 得到歌手开头拼音字母
          if (!map[key]) { // 没有首字母对象就创建
            map[key] = {
              title: key,
              items: []
            }
          }
          map[key].items.push(new Singer({ // 利用封装的Singer方法得到歌手id,名称,头像地址
            id: item.Fsinger_mid,
            name: item.Fsinger_name
          }))
        })

        // 得到有序列表,处理map
        let hot = []
        let ret = []
        for (let key in map) {
          let val = map[key]
          // 过滤排序
          if (val.title.match(/[a-zA-Z]/)) {
            ret.push(val)
          } else if (val.title === HOT_NAME) {
            hot.push(val)
          }
        }
        // 排序
        ret.sort((a, b) => {
          return a.title.charCodeAt(0) - b.title.charCodeAt(0)
        })
        return hot.concat(ret) // 拼接数组 得到一维数组
      },
      ...mapMutations({
        setSinger: 'SET_SINGER' // 通过语法糖配置要设置的常量
      })
    },
    components: {
      Listview
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .singer
    position: fixed
    top: 88px
    bottom: 0
    width: 100%
</style>
