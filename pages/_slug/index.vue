// $config に環境変数が入っている
<template>
  <main class="main">
    <h1 class="title">{{ title }}</h1>
    <p class="publishedAt">{{ publishedAt }}</p>
    <p class="body">{{ body }}</p>
    <!-- <div class="post" v-html="content"></div> -->
  </main>
</template>

<script>
import axios from 'axios'
export default {
  async asyncData({ params, $config }) {
    const { data } = await axios.get(
      `https://jmkvfdwrd4.microcms.io/api/v1/blog/${params.slug}`,
      { headers: { 'X-MICROCMS-API-KEY': $config.apiKey }
      }
    )
    return data
  },
  // 静的ファイルを出力するのに必要な設定
  generate: {
    async routes( $config ) {
      const pages = await axios.get(
        'https://jmkvfdwrd4.microcms.io/api/v1/blog/',
        { headers: { 'X-MICROCMS-API-KEY': $config.apiKey }}
        )
        .then((res) =>
          res.data.contents.map((content) => ({
            route: `/${content.id}`,
            payload: content
          }))
        )
      return pages
    }
  }
}
</script>

<style lang="scss" scoped>
.main {
  width: 960px;
  margin: 0 auto;
}

.title {
  margin-bottom: 20px;
}

.publishedAt {
  margin-bottom: 40px;
}

.post {
  & > h1 {
    font-size: 30px;
    font-weight: bold;
    margin: 40px 0 20px;
    background-color: #eee;
    padding: 10px 20px;
    border-radius: 5px;
  }

  & > h2 {
    font-size: 24px;
    font-weight: bold;
    margin: 40px 0 16px;
    border-bottom: 1px solid #ddd;
  }

  & > p {
    line-height: 1.8;
    letter-spacing: 0.2px;
  }

  & > ol {
    list-style-type: decimal;
    list-style-position: inside;
  }
}
</style>