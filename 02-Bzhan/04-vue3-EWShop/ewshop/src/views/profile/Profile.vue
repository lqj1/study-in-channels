<template>
  <div id="box">
    <nav-bar class="nav-bar">
      <template v-slot:default>
        个人中心
      </template>
    </nav-bar>
    <div class="user-box">
      <div class="user-info">
        <div class="info">
          <img src="~assets/images/user.png" />
          <div class="user-desc">
            <span>昵称：{{user.name}}</span>
            <span>登录名：{{user.email}} </span>
            <span class="name">个性签名：成就自己的只需一套精品</span>
          </div>
        </div>
      </div>
      <ul class="user-list">
        <li class="van-hairline--bottom" v-for="(item,index) in gotoLinkLists" :key="index" @click="goTo(item.path)">
          <span>{{item.title}}</span>
          <van-icon name="arrow" />
        </li>
      </ul>
    </div>
    <div style="margin: 16px;">
      <van-button round block color="#42b983" @click="tologout">退出登录</van-button>
    </div>
  </div>
</template>

<script>
import NavBar from 'components/common/navbar/NavBar'
import { logout, getUser } from 'network/user'
import { Toast } from 'vant'
import { useRouter } from 'vue-router'
import { useStore } from 'vuex'
import { onMounted, reactive, toRefs } from '@vue/runtime-core'
export default {
  name: 'Profile',
  components: {
    NavBar
  },
  setup () {
    const router = useRouter()
    const store = useStore()
    const gotoLinkLists = [
      {
        path: '/collect',
        title: '我的收藏'
      },
      {
        path: '/order',
        title: '我的订单'
      },
      {
        path: '/setting',
        title: '账号管理'
      },
      {
        path: '/address',
        title: '地址管理'
      },
      {
        path: '/about',
        title: '关于我们'
      }
    ]
    const state = reactive({
      user: {}
    })
    onMounted(() => {
      getUser().then(res => {
        state.user = res
      })
    })
    const toLogout = () => {
      logout().then(res => {
        console.log('res', res)
        if (res.status === 204) {
          Toast.success('退出成功')
          // 清除 token
          window.localStorage.setItem('token', '')
          store.commit('setIsLogin', false) // 对应Login.vue中设置的登录状态为 true
          setTimeout(() => {
            router.push({ path: '/login' })
          }, 500);
        }
      })
    }
    // 跳转方法
    const goTo = (path, query) => {
      router.push({ path, query: query || {} })
    }
    return {
      ...toRefs(state),
      gotoLinkLists,
      toLogout,
      goTo
    }
  }
}
</script>

<style lang="scss" scoped>
    #box {
        background:#FCFCFC;
        height:100vh;
    }
    .user-box {
        margin-top:65px;
        .user-header {
            position: fixed;
            top: 0;
            left: 0;
            z-index: 10000;
            width:100%;
            height:44px;
            line-height: 44px;
            padding: 0 10px;
            color: #252525;
            background: #fff;
            border-bottom: 1px solid #dcdcdc;
            .user-name {
                font-size: 14px;
            }
        }
        .user-info {
            width: 94%;
            margin: 10px;
            height: 115px;
            background: linear-gradient(90deg, #31c7A7, #A1c7c7);
            box-shadow: 0 2px 5px #269090;
            border-radius: 6px;
            margin-top: 50px;
            text-align: left;
            .info {
                position: relative;
                display: flex;
                width: 100%;
                height: 100%;
                padding: 25px 20px;

                img {
                   width:60px;
                    height:60px;
                    border-radius: 50%;
                    margin-top: 4px;
                }
                .user-desc {
                    display: flex;
                    flex-direction: column;
                    margin-left: 10px;
                    line-height: 20px;
                    font-size: 14px;
                    color: #fff;
                    span {
                        color: #fff;
                        font-size: 14px;
                        padding: 2px 0;
                    }
                }
                .account-setting {
                    position: absolute;
                    top: 10px;
                    right: 20px;
                    font-size: 13px;
                    color: #fff;
                    .van-icon-setting-o {
                        font-size: 16px;
                        vertical-align: -3px;
                        margin-right: 4px;
                    }
                }
            }
        }
        .user-list {

            padding: 0 8px;
            margin-top: 40px;
            li {
                padding-left:5px;
                padding-right:5px;
                height: 40px;
                line-height: 40px;
                display: flex;
                justify-content: space-between;
                font-size: 14px;
                .van-icon-arrow {
                    margin-top: 13px;
                }
                margin:20px 0 !important;
               background: #FFFFFF;
                border-radius: 3px;
            }
        }
    }
</style>