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

  // 環境変数を読み込む
  // サーバーからのみ使用する環境変数
  privateRuntimeConfig: {
    apiKey: API_KEY,
    apiUrL: API_URL
  },
  // クライアントとサーバーの両方で使用する環境変数
  publicRuntimeConfig: {
    apiKey: process.env.NODE_ENV !== 'production' ? API_KEY : undefined,
    apiUrl: process.env.NODE_ENV !== 'production' ? API_URL : undefined
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

### pages/index.vue

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

### pages/_slug/index.vue
microCMSの管理画面上で設定したコンテンツIDがURLとなる

**axios のget URLを環境変数にしているとバグる問題**😖

```html
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
```

## Vueファイルを静的ファイルにビルドする
~/dist/ 配下に静的ファイルが生成される

```bash
$ npm run generate
$ npm start

-> http://localhost:3000/
```

## Netlify でホスティングする
あらかじめNetlifyに登録しGitHubとアカウント連携する

## microCMSとNetlifyを連携する