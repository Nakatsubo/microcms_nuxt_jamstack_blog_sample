// $config に環境変数が入っている
<template>
  <ul>
    <li v-for="content in contents" :key="content.id">
      <nuxt-link :to="`/${content.id}`">
        {{ content.title }}
      </nuxt-link>
      <!-- {{ content.content }} -->
    </li>
  </ul>
</template>

<script>
import axios from 'axios'
export default {
  async asyncData({ $config }) {
    const { data } = await axios
      .get(
        'https://jmkvfdwrd4.microcms.io/api/v1/blog/',
        { headers: { 'X-MICROCMS-API-KEY': $config.apiKey }}
    )
    return data;
  },
}
</script>