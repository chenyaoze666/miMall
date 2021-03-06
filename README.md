# MiMall
高仿小米商城

## 跨域
* CORS跨域
  * 服务器设置 后端允许前端某个站点进行访问
* JSONP跨域
  * 前端适配, 后台配合 前后台同时改造
  * 前端jsonp插件
```
jsonp(url, (err,res)=>{

})
```
* 代理跨域
  * 接口代理 通过修改nginx服务配置来实现
  * 前端修改, 后端不动
```
新建 vue.config.js

module.exports = {
  devServer:{
    host:'localhost',
    port:3000,
    proxy:{
      '/api':{
        target:'代理地址(http://localhost:8080)',
        changeOrigin:false//主机头url
      }
    }
  }
}


module.exports = {
  devServer:{
    host:'localhost',
    port:8080,
    proxy:{
      '/api':{
        target:'代理地址',
        changeOrigin:true,//主机头url
        pathRewrite:{//两者配合表示请求地址上有/api 但实际上他表示''
          '/api':''
        }
      }
    }
  }
}

```
## 依赖
```
npm i vue-lazyload element-ui node-sass sass-loader vue-awesome-swiper vue-axios vue-cookie --save-dev
```
## 路由封装
```
import Vue from 'vue'
import Router from 'vue-router'

import Home from './pages/home';
import Index from './pages/index';
import Product from './pages/product';
import Detail from './pages/detail';
import Cart from './pages/cart';
import Order from './pages/order';
import OrderConfirm from './pages/orderConfirm';
import OrderList from './pages/orderList';
import OrderPay from './pages/orderPay';
import Alipay from './pages/alipay';

Vue.use(Router);

export default new Router({
  mode:"history",
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home,
      redirect:'/index',
      children: [
        {
          path: '/index',
          name: 'index',
          component: Index,
        },
        {
          path: '/product/:id',
          name: 'product',
          component: Product,
        },
        {
          path: '/detail/:id',
          name: 'detail',
          component: Detail,
        },
      ]
    },
    {
      path: '/cart',
      name: 'cart',
      component: Cart,
    },
    {
      path: '/order',
      name: 'order',
      component: Order,
      children: [
        {
          path: 'list',
          name: 'ordor-list',
          component: OrderList,
        },
        {
          path: 'confirm',
          name: 'Order-confirm',
          component: OrderConfirm,
        },
        {
          path: 'pay',
          name: 'order-pay',
          component: OrderPay,
        },
        {
          path: 'alipay',
          name: 'alipay',
          component: Alipay,
        }
      ]
    }
  ]
})
```
## Storage封装
* Cookie, localStorage, sessionStorage三者的区别
```
存储大小:  C  4K,       S  5M
有效期:    C  有效期    S  永久
C 会发送到服务器 存储在内存中  S值存在浏览器
路径:  C 有路径限制  S只存在域名下  
API   C 没有特定的API   S   有对应的API  

localStorage    不随浏览器关闭而删除

sessionStorage 随浏览器关闭而删除
```
```
/**
 * Storage封装
 */

const STORAGE_KEY = 'mall';

export default{
  // 存储值
  setItem(key,value,module_name){//user val
    if(module_name){
      let val = this.getItem(module_name);
      val[key] = value;
      this.setItem(module_name,val)
    }else{
      let val = this.getStorage();
      val[key] = value
      window.sessionStorage.setItem(STORAGE_KEY,JSON.stringify(val));
    }
   
  },
  // 获取某一个模块的属性
  getItem(key,module_name){
    if(module_name){
      let val = this.getItem(module_name);
      if(val) return val[key]
    }
    return this.getStorage()[key];
  },
  getStorage(){
   return JSON.parse( window.sessionStorage.getItem(STORAGE_KEY) || '{}' );
  }, 
  clear(key,module_name){
    let val = this.getStorage();
    if(module_name){
      if(!val[module_name][key]) return;
      delete val[module_name][key];
    }else{
      delete val[key];
    }
    window.sessionStorage.setItem(STORAGE_KEY,JSON.stringify(val));
  }
}
```
## 接口拦截
```
import Vue from 'vue'
import App from './App.vue'
import axios from 'axios'
import VueAxios from 'vue-axios'
import router from './router'

//根据前端的跨域方式做调整
axios.defaults.baseURL = '/api';
axios.defaults.timeout = 8000;

//接口错误拦截
axios.interceptors.response.use(function(response) {
  let res = response.data;
  if(res.status == 0){
    return res.data;
  }else if(res.status == 10){//10代表未登录
    window.location.href = '/login'
  }else{
    alert(res.msg);
  }
})

Vue.use(VueAxios,axios);

Vue.config.productionTip = false

new Vue({
  router,
  render: h => h(App),
}).$mount('#app')

```
## 接口环境设置
```
配置环境变量
 "serve": "vue-cli-service serve --mode=development",
 "test": "vue-cli-service serve --mode=test",
 "build": "vue-cli-service build --mode=production"


 如需添加自定义环境 如prev

 新建.evn.prev
 ```
 NODE_ENV='prev'
 ```

新建evn.js


let baseURL;

/**
 * 查询进程的环境变量
 */
switch (process.env.NODE_ENV) {
  case 'development':
    baseURL = 'http://dev-mall-pre.springboot.cn/api';
    break;
  case 'test':
    baseURL = 'http://test-mall-pre.springboot.cn/api';
    break;
  case 'production':
    baseURL = 'http://mall-pre.springboot.cn/api';
    break;
  default:
    baseURL = 'http:/mall-pre.springboot.cn/api';
    break;
}

export default {
  baseURL
}

```

## Mock设置 
* 本地创建json
```
public/mock/user/login.json

{
  "status": 0,
  "data": {
    "id": 12,
    "username": "admin",
    "email": "admin@51purse.com",
    "phone": null,
    "role": 0,
    "createTIme": 147904832500,
    "updateTIme": 147904832500
  }
}

  this.axios.get('/mock/user/login.json').then((res) => {
      this.res = res;
    })

```
* easy-mock平台
```
axios.defaults.baseURL = '写上easy-mock的项目生成的地址';

  this.axios.get('/user/login').then((res) => {
      this.res = res;
    })
```
* 集成Mock API
```
安装
cnpm i mockjs --save-dev

//mock开关
const mock = true;
if(mock){//import 预编译就加载  require 需要时加载
  require('./mock/api');
}


app.js

import Mock from 'mockjs'
Mock.mock('/api/user/login',{
  "status": 0,
  "data": {
    "id": 13,
    "username": "admin",
    "email": "admin@51purse.com",
    "phone": null,
    "role": 0,
    "createTIme": 147904832500,
    "updateTIme": 147904832500
  }
})


//第三种方式 集成Mockjs
      this.axios.get('/user/login').then((res) => {
      this.res = res;
    })


    不会发起请求 而是直接就给数据了

```
## 基础css抽离
* 引入 @import './assets/scss/reset.scss';