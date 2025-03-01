<template>
  <VueLoading :active="isLoading"/>
  <div class="section">
    <div class="background-image background-image--favorite"></div>
    <div class="background-title">
      <h2 class="fw-bold">收藏清單</h2>
    </div>
  </div>
  <div class="container mt-md-5 mt-3 mb-5">
    <div v-if="favoritesList.length">
      <div class="d-flex justify-content-center align-items-center mb-5">
        <h3 class="section-heading">收藏清單</h3>
      </div>
      <table class="table favorite__table">
        <thead>
          <tr>
            <th scope="col" class="border-0 ps-0">商品名稱</th>
            <th scope="col" class="border-0">價格</th>
            <th scope="col" class="border-0">狀態</th>
            <th scope="col" class="border-0">移除</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in favoritesList" :key="item.id" class="border-bottom border-top">
            <th scope="row" class="border-0 px-0 font-weight-normal py-4">
              <RouterLink :to="`/product/${item.id}`">
                <img class="favorite__thumbnail"
                  :src="item.imageUrl" alt="{{ item.title }}">
                <p class="mb-0 fw-bold ms-3 d-inline-block text-dark">{{ item.title }}</p>
              </RouterLink>
            </th>
            <td class="favorite__price">
              <p class="">NT$ {{ item.price }} / {{ item.unit }}</p>
            </td>
            <td class="border-0 align-middle">
              <button class="btn-primary btn" type="button"
              @click="addToCart(item.id)" :disabled="isLoadingItem === item.id">
                <span class="spinner-border spinner-border-sm"
                  role="status" v-show="isLoadingItem === item.id"></span>
                  加入購物車
              </button>
            </td>
            <td class="border-0 align-middle">
              <i class="bi bi-trash-fill fs-5" role="button" @click="removeFavorite(item.id)"></i>
            </td>
          </tr>
        </tbody>
      </table>
      <ul class="cart__list d-md-none">
        <li
          class="card cart__card"
          v-for="item in favoritesList"
          :key="item.id">
            <div class="d-flex justify-content-between">
              <span class="icon--close icon-close--sp cartNav__close" @click="removeFavorite(item.id)"></span>
              <img class="favorite__thumbnail"
              :src="item.imageUrl" alt="{{ item.title }}">
              <div class="cart__card__content">
                {{ item.title }}
                <p class="mb-0 ms-auto">小計: NT$ {{ item.price }}</p>
                <button class="btn-primary btn w-100 mt-2" type="button"
                @click="addToCart(item.id)" :disabled="isLoadingItem === item.id">
                  <span class="spinner-border spinner-border-sm"
                    role="status" v-show="isLoadingItem === item.id"></span>
                    加入購物車
                </button>
              </div>
            </div>
        </li>
      </ul>
    </div>
    <div v-else class="container--center">
      <h3 class="text-center">目前還沒有收藏行程唷！<br>快去逛逛吧！</h3>
      <RouterLink to="/products"
      class="button--jump mt-5"
      @click="isOpen = !isOpen">開始旅程</RouterLink>
    </div>
  </div>
</template>

<script>
import emitter from '@/libs/emitter'

export default {
  data () {
    return {
      products: [],
      favoritesList: [],
      isDisabled: '',
      isLoading: false,
      isLoadingItem: '',
      favorite: JSON.parse(localStorage.getItem('favorite')) || []
    }
  },
  inject: ['emitter'],
  methods: {
    getProducts () {
      const api = `${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/products/`
      this.isLoading = true
      this.$http.get(api)
        .then((response) => {
          this.products = response.data.products
          this.getFavorites()
          this.isLoading = false
        })
        .catch((error) => {
          this.emitter.emit('push-message', {
            style: 'danger',
            title: '找不到收藏清單',
            content: error.response.data.message
          })
        })
    },
    getFavorites () {
      this.favoritesList = this.products.filter(
        (item) => this.favorite.indexOf(item.id) > -1
      )
    },
    addToCart (id, qty = 1) {
      const data = {
        product_id: id,
        qty
      }
      this.isLoading = true
      this.$http.post(`${process.env.VUE_APP_API}api/${process.env.VUE_APP_PATH}/cart`, { data })
        .then((response) => {
          this.emitter.emit('push-message', {
            style: 'success',
            title: '購物提示',
            content: response.data.message
          })
          this.isLoading = false
          this.isDisabled = ''
          emitter.emit('get-cart')
        })
        .catch((error) => {
          this.emitter.emit('push-message', {
            style: 'danger',
            title: '無法加入購物車',
            content: error.response.data.message
          })
        })
    },
    removeFavorite (id) {
      this.isLoading = true
      const favoriteId = this.favorite.indexOf(id)
      if (favoriteId !== -1) {
        this.favorite.splice(favoriteId, 1)
        localStorage.setItem('favorite', JSON.stringify(this.favorite))
        this.getFavorites()
        this.isLoading = false
        this.emitter.emit('push-message', {
          style: 'success',
          title: '已刪除'
        })
      }
    }
  },
  mounted () {
    this.getProducts()
  }
}
</script>
