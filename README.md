# microCMS + NuxtでJamstackブログを作ってみよう
Referenced by [microCMS Tutorials](https://blog.microcms.io/microcms-nuxt-jamstack-blog/).

# 補足

## 環境変数を設定する

- dotenvを設定しなくても環境変数を .envファイルから読み込むことができる
- nuxt.config.js に下記コードを追記し .envファイルを読み込む

### .env

```
API_KEY=your_api_key
```

### nuxt.config.js

```javascript
const { API_KEY, API_URL } = process.env;

export default {

// ...

  privateRuntimeConfig: {
    apiKey: API_KEY,
    // apiUrL: API_URL
  },
  publicRuntimeConfig: {
    apiKey: process.env.NODE_ENV !== 'production' ? API_KEY : undefined
  },

}
```

## axios と sass をインストールする

- node-sassをインストールしてしまうと、デプロイ時にエラーが発生する
- sass-loaderのバージョン依存を解消しないと、デプロイ時にエラーが発生する

[プリプロセッサ](https://nuxtjs.org/ja/docs/features/configuration/#%E3%83%97%E3%83%AA%E3%83%97%E3%83%AD%E3%82%BB%E3%83%83%E3%82%B5)

```
// axios
$ npm install -D @nuxtjs/axios

// sass
$ npm install -D sass sass-loader@10.1.1
```

### package.json

```javascript
{

  // ...

  "dependencies": {
    // @nuxtjs/axios は　dependencies にインストールする
    "@nuxtjs/axios": "^5.13.6",
    "core-js": "^3.15.1",
    "nuxt": "^2.15.7"
  },
  "devDependencies": {
    "sass": "^1.49.7",
    "sass-loader": "^10.1.1"
  }
}
```

### nuxt.config.js

```javascript
export default {

  // ...
  modules: [
    "@nuxtjs/axios",
  ],

}
```

## テンプレートファイルを作成
APIから値が返ってくればOK!

### index.vue

```html
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
```
