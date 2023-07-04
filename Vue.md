# Vue基础

## package.config

```C#
{
  "name": "demo",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build"
  },
  "dependencies": {
    "core-js": "^3.8.3",
    "vue": "^3.2.13",
    "vue-router": "^4.0.3",
    "vuex": "^4.0.0"
  },
  "devDependencies": {
    "@vue/cli-plugin-babel": "~5.0.0",
    "@vue/cli-plugin-router": "~5.0.0",
    "@vue/cli-plugin-vuex": "~5.0.0",
    "@vue/cli-service": "~5.0.0"
  },
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not dead",
    "not ie 11"
  ]
}
```

^：表示大版本不小于3，小版本不小于8.3

~:不能修改大版本号

## view.config.js

```C#
const { defineConfig } = require('@vue/cli-service')
module.exports = defineConfig({
  transpileDependencies: true,
  devServer:{
    port:8081,
    host:"localhost",
    open:true
  }
})
```

可配置端口



@表示根目录

. 本级

.. 上一级

## 可用的Css

```css
* {
    margin: 0;
    padding: 0;
}

.n-card {
    max-width: 300px;
}

ul li {
    list-style: none;
}

a {
    text-decoration: none;
    color: #333;
}

body {
    font-size: 14px;
}

.login {
    width: 100%;
    height: 100vh;
    background-size: cover;
}

.login-content {
    background-color: white;
    width: 45vh;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 3px 2px 2px rgb(0, 0, 0, .3);
    position: relative;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
}
.login .login-title{
    font-size: 20px;
    text-align: center;
    margin: 20px 0;
}
.login .login-button{
    text-align: center;
}

#module .n-layout-sider{
    height: 100vh;
    background-color:	rgb(121, 116, 126);;
}
#module .n-layout-header{
    height: 5vh;
    background-color:	rgb(201, 198, 204);;
}
#module .n-layout-content{
    height: 90vh;
}
#module .n-layout-footer{
    background-color:#d9d9d9;
    height: 5vh;
}
.content-top {
    box-shadow: 0 3px 2px 2px rgba(235, 232, 232, 0.3);
    height: 3vh;
    padding-top: 5px;
    padding-bottom: 5px;
}
#header .header-right{
    text-align:right;
}
#header .header-right .header-right-dot-button{
    padding: 15px 10px 10px  10px;
}
#header .header-right .header-right-span-drop{
    
}

```

## 全局配置



```C@
app.config.globalProperties.$layer = layer;
```

## 路由

```C#
const router = new VueRouter({
    routes: [
        {
            path: '/notes',
            name: 'notes',
            component: () => import('@/views/Notes')
	},
	{
            path: '/login',
            name: 'login',
            component: () => import('@/views/Login')
	},
    ]
})

//全局守卫
router.beforeEach((to, from, next) => {
  //用户访问的是'/notes'
  if(to.path === '/notes') {
    //查看一下用户是否保存了登录状态信息
    let user = 
    JSON.parse(localStorage.getItem('user'))
    if(user) {
      //如果有，直接放行
      next();
    }else {
      //如果没有，用户跳转登录页面登录
      next('/login')
    }
  }else {
    next();
  }
})
```



