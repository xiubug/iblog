---
title: Nuxtjs 开发笔记
tags:
  - note
categories:
  - Nuxtjs
date: 2018-11-03 21:13:56
---

## 开发用法
### 按需引入UI组件库
这边以 element-ui 组件库为例：
#### 使用 babel-plugin-import 
`babel-plugin-import`是一款 babel 插件，它会在编译过程中将 import 的写法自动转换为按需引入的方式
``` bash
# 安装 babel-plugin-import 插件
npm i babel-plugin-import -D
```
#### 配置nuxt.config.js：
``` js
module.exports = {
  mode: 'universal',

  /*
  ** Headers of the page
  */
  head: {
    title: pkg.name,
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { hid: 'description', name: 'description', content: pkg.description }
    ],
    link: [
      { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
    ]
  },

  /*
  ** Customize the progress-bar color
  */
  loading: { color: '#fff' },

  /*
  ** Global CSS
  */
  css: [
    // 自定义公共 CSS 文件
    '@/assets/css/common.css'
    // 全部引用的时候需要用到
    // 'element-ui/lib/theme-chalk/index.css'
  ],

  /*
  ** Plugins to load before mounting the App
  */
  plugins: [
    { src: '@/plugins/element-ui', ssr: true }
  ],

  /*
  ** Nuxt.js modules
  */
  modules: [
  ],

  /*
  ** Build configuration
  */
  build: {
    analyze: true,
    vendor: ['element-ui'],
    maxChunkSize: 300000,
    babel: {
      plugins: [
        [
          'component',
          {
            'libraryName': 'element-ui',
            'styleLibraryName': 'theme-chalk'
          }
        ]
      ]
    },
    /*
    ** You can extend webpack config here
    */
    extend(config, ctx) {
      // Run ESLint on save
      if (ctx.isDev && ctx.isClient) {
        config.module.rules.push({
          enforce: 'pre',
          test: /\.(js|vue)$/,
          loader: 'eslint-loader',
          exclude: /(node_modules)/
        })
      }
    }
  }
}
```
#### 修改plugins/element-ui.js：
``` js
import Vue from 'vue'
import { Button } from 'element-ui'
Vue.component(Button.name, Button)
```
#### 最后使用组件
``` html
<el-button type="primary">主要按钮</el-button>
```

## 常见问题

