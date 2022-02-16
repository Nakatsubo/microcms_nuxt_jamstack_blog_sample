# microCMS + NuxtでJamstackブログを作ってみよう
Referenced by [microCMS Tutorials](https://blog.microcms.io/microcms-nuxt-jamstack-blog/).

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
