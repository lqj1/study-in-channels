<template>
  <div class="address-box">
    <nav-bar class="nav-bar">
      <template v-slot:default>
        地址管理
      </template>
    </nav-bar>

    <div v-show="list.length == 0" style="margin-top:200px;text-align: center;">
      还没有地址信息，去添加吧！
    </div>

    <div class="address-item">
      <van-address-list v-model="chosenAddressId" :list="list" default-tag-text="默认" @add="onAdd" @edit="onEdit" />
    </div>
  </div>
</template>

<script>
import { reactive, toRefs, onMounted } from 'vue'
import NavBar from "components/common/navbar/NavBar";
import { getAddressList } from 'network/address'
import { useRoute, useRouter } from 'vue-router'
export default {
  components: {
    NavBar
  },
  setup () {
    const route = useRoute()
    const router = useRouter()
    const state = reactive({
      chosenAddressId: '1',
      list: []
    })

    onMounted(() => {
      getAddressList().then(res => {
        // console.log(res)
        if (res.data.length === 0) {
          state.list = []
          return
        }
        state.list = res.data.map(item => {
          return {
            id: item.id,
            name: item.name,
            tel: item.phone,
            address: `${item.province} ${item.city} ${item.county} ${item.address}`,
            isDefault: !!item.is_default  // 0/1 转 布尔值，双重取反
          }
        })
      })
    })


    const onAdd = () => {
      router.push({ path: '/addressedit', query: { type: 'add' } })
    }

    const onEdit = (item) => {
      router.push({ path: '/addressedit', query: { type: 'edit', addressId: item.id } })
    }



    return {
      ...toRefs(state),
      onAdd,
      onEdit
    }
  }
}
</script>

<style lang="scss">
  .address-box {
    height:300px;
    .van-radio__icon {
      display: none;
    }
    .address-item {
      margin-top: 45px;
      .van-button {
        background: var(--color-tint);
        border-color: var(--color-tint);
      }
    }
  }
  .van-address-list__bottom {
    bottom: 100px !important;
  }
</style>
