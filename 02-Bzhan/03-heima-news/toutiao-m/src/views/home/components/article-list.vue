<template>
  <div class="article-list" ref="article-list">
    <van-pull-refresh v-model="isRefreshLoading" :success-text="refreshSuccessText" :success-duration="1500" @refresh="onRefresh">
      <van-list
        v-model="loading"
        :finished="finished"
        finished-text="没有更多了"
        @load="onLoad"
      >
        <article-item v-for="(article, index) in articles" :key="index" :article="article"></article-item>
        <!-- <van-cell v-for="(article,index) in articles" :key="index" :title="article.title" /> -->
      </van-list>
    </van-pull-refresh>
  </div>
</template>

<script>
import { getArticles } from '@/api/article'
import ArticleItem from '@/components/article-item'
import { debounce } from 'lodash'
export default {
  name: 'ArticleList',
  props: {
    channel: {
      required: true,
      type: Object
    }
  },
  components: {
    ArticleItem
  },
  data () {
    return {
      articles: [], // 数据列表
      loading: false, // 控制加载中的 loading 状态
      finished: false, // 控制加载结束的状态，当加载结束，不再加载更多
      timestamp: null, // 时间戳
      isRefreshLoading: false, // 下拉刷新的loading状态
      refreshSuccessText: '', // 下拉刷新成功的提示文本
      scrollTop: 0 // 列表滚动到顶部的距离
    }
  },
  methods: {
    async onLoad () {
      // 异步更新数据
      // 1. 请求获取数据
      const { data } = await getArticles({
        channel_id: this.channel.id, // 频道id
        timestamp: this.timestamp || Date.now(), // 时间戳，请求新的推荐数据传当前的时间戳，请求历史推荐传指定的时间戳，timestamp相当于页码，请求最新数据使用当前最新时间戳，下一页数据使用上一次返回的数据中的时间戳
        with_top: 1 // 是否包含置顶项，进入页面第一次请求时要包含置顶文章，1-包含置顶项，0-不包含
      })
      // 2. 把数据放到 list 数组中
      const { results } = data.data
      this.articles.push(...results)
      // 3. 设置本次加载状态结束，它才可以判断是否需要加载下一次，否则就会永远地停留在这里
      this.loading = false
      // 4. 数据全部加载完成
      if (results.length) {
        // 更新获取下一页的页码
        this.timestamp = data.data.pre_timestamp
      } else {
        // 没有数据，把加载状态设置结束，不再触发加载更多
        this.finished = true
      }
    },
    async onRefresh () {
      // 下拉刷新，组件自己就会展示 loading 状态
      // 1. 请求获取数据
      const { data } = await getArticles({
        channel_id: this.channel.id,
        timestamp: Date.now(), // 只要传递不同的时间戳，就会返回不一样的数据，且数据不为空
        with_top: 1
      })
      // 2. 把数据放到数据列表中（往顶部追加）
      const { results } = data.data
      this.articles.unshift(...results)
      // 3. 关闭刷新的状态 loading
      this.isRefreshLoading = false
      this.refreshSuccessText = `更新了${results.length}条数据`
      // console.log('onRefresh')
    }
  },
  mounted () {
    // 监听首页滚动位置
    const articleList = this.$refs['article-list']
    articleList.onscroll = debounce(() => {
      this.scrollTop = articleList.scrollTop
    }, 50)
  },
  activated () {
    this.$refs['article-list'].scrollTop = this.scrollTop
  },
  deactivated () {
    console.log('deactivated')
  }
}
</script>

<style lang="less" scoped>
.article-list {
  position: fixed;
  left: 0;
  right: 0;
  top: 90px;
  bottom: 50px;
  overflow-y: auto;
  z-index: 99;
}
</style>
