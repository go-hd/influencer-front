<template>
  <div>
    <div class="container" :style="containerStyle">
      <div
        v-for="medium in filteredMedia"
        :key="medium.id"
        class="post"
        :style="{ backgroundImage: `url(${medium.media_url})` }"
      >
        <a v-if="hasLink" :href="medium.permalink" class="caption">
          <!-- eslint-disable-next-line vue/no-v-html -->
          <div v-html="medium.caption.replace(/\uFEFF?\n/g, '<br>')"></div>
        </a>
        <div v-else-if="hasCaption" class="caption">
          <!-- eslint-disable-next-line vue/no-v-html -->
          <div v-html="medium.caption.replace(/\uFEFF?\n/g, '<br>')"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  validate({ query }) {
    return (
      (!query.rows || /^\d+$/.test(query.rows)) &&
      (!query.link || /^[01]$/.test(query.link)) &&
      query.access_token
    )
  },

  data() {
    return {
      width: 0
    }
  },

  computed: {
    rows() {
      return this.$route.query.rows || 3
    },

    columns() {
      const width = this.width
      const columnWidth = width > 768 ? 250 : width > 544 ? 150 : 80

      return parseInt(width / columnWidth)
    },

    limit() {
      return this.rows * this.columns
    },

    filteredMedia() {
      return this.media.slice(0, this.limit)
    },

    hasLink() {
      return this.$route.query.link !== '0' && this.hasCaption
    },

    hasCaption() {
      return this.width > 768
    },

    containerStyle() {
      const { columns, rows } = this

      return {
        gridTemplateColumns: `repeat(${columns}, calc(100vw / ${columns}))`,
        gridTemplateRows: `repeat(${rows}, calc(100vw / ${columns}))`
      }
    }
  },

  asyncData({ $axios, params, query, error }) {
    $axios.setToken(query.access_token, 'Bearer')
    return $axios.$get(`/api/${params.brand}/media`).catch(() =>
      error({
        statusCode: 404,
        message: 'This page could not be found.'
      })
    )
  },

  mounted() {
    this.width = window.innerWidth
    window.addEventListener('resize', this.handleResize)
  },

  beforeDestroy() {
    window.removeEventListener('resize', this.handleResize)
  },

  methods: {
    handleResize() {
      this.width = window.innerWidth
    }
  }
}
</script>

<style lang="scss">
.container {
  display: grid;

  > .post {
    background-size: cover;
    background-position: center;
    overflow: hidden;

    .caption {
      display: block;
      width: 100%;
      height: 100%;
      visibility: hidden;
      opacity: 0;
      background: rgba(0, 0, 0, 0.8);
      color: white;
      padding: 1em;
      font-size: 12px;
      overflow: scroll;
      transition: all 0.3s ease-out;
      text-decoration: none;

      > div {
        overflow-y: scroll;
      }
    }

    &:hover .caption {
      visibility: visible;
      opacity: 1;
    }
  }
}
</style>
