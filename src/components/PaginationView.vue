<template>
  <nav aria-label="Page navigation">
    <ul class="pagination">
      <li class="page-item" :class="{ disabled: !pages.has_pre }">
        <a href="#" class="page-link" aria-label="Previous"
          @click.prevent="emitPages(pages.current_page - 1)">上一頁</a>
      </li>
      <li class="page-item" :class="{ active : page === pages.current_page }"
        v-for="page in pages.total_pages" :key="page" >
        <a class="page-link" href="#" @click.prevent="emitPages(page)">{{ page }}</a>
      </li>
      <li class="page-item" :class="{ disabled: !pages.has_next }">
        <a class="page-link" aria-label="Next" href="#"
          @click.prevent="emitPages(pages.current_page + 1)">下一頁</a>
      </li>
    </ul>
  </nav>
</template>

<script>
export default {
  props: ['pages'],
  methods: {
    emitPages (item) {
      this.$emit('emit-pages', item)
    }
  }
}
</script>
