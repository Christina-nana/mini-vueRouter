# 手写mini vue-router
> 目前只实现了、<router-link>、<router-view>组件、createRouter、createWebHashHistory函数、useRouter功能 

新建src/router/mini-router/index.js文件，主要用来写底层的mini vue-router逻辑：
```javascript
import { inject,ref } from "vue"
import RouterLink from './RouterLink.vue'
import RouterView from './RouterView.vue'

const ROUTER_KEY = '__router__'

function createRouter(options){
    return new Router(options)
}

function useRouter(){
    return inject(ROUTER_KEY)
}

function createWebHashHistory(){
    function bindEvent(fn){
        window.addEventListener('hashchange',fn)
    }
    return {
        bindEvent,
        url: window.location.hash.slice(1)
    }
   
}

class Router{
    constructor(options){
        this.routes = options.routes
        this.history = options.history
        this.current = ref(this.history.url)

        this.history.bindEvent(()=>{
            this.current.value = window.location.hash.slice(1)
        })
    }
    install(app){
        app.provide(ROUTER_KEY,this)
        app.component('router-link',RouterLink)
        app.component('router-view',RouterView)
    }
}
export {createRouter,createWebHashHistory,useRouter}
``` 
新建src/router/mini-router/RouterView.vue，配置router的<router-view>内置组件：
```vue
<template>
    <component :is="component"></component>
</template>
<script setup>
import { computed } from 'vue';
import { useRouter } from './index';
const router = useRouter()
const component = computed(()=>{
    const route =  router.routes.find(
        (route)=>route.path === router.current.value
    )
    return route ? route.component : null
})
</script>
```

新建src/router/mini-router/RouterLink.vue，配置router的<router-link>内置组件：
```vue
<template>
    <a :href="'#' + props.to">
        <slot/>
    </a>
</template>

<script setup>
import { defineProps } from 'vue';
const props = defineProps({
    to:{
        type:String,
        require:true
    }
})
</script>
``` 

使用mini vue-router：在router配置文件src/router/index.js中引入
```javascript 
import {createRouter,createWebHashHistory} from './mini-router/index'
import Home from '../pages/home.vue'
import About from '../pages/about.vue'

const routes = [
    {
        path:'/',
        name:'home',
        component:Home
    },
    {
        path:'/about',
        name:'about',
        component:About
    }
]
const router = createRouter({
    history:createWebHashHistory(),
    routes
})

export default router
```

在main.js入口文件注册mini vue-router组件：

```javascript 
import { createApp } from 'vue'
import App from './App.vue'
import router from './router/index'

createApp(App)
.use(router)
.mount('#app')
``` 

在App.vue根组件中使用mini vue-router配置的<router-link>和<router-view>组件:
```vue 
<template>
    <div>
        <router-link to="/">首页</router-link> |
        <router-link to="/about">关于我们</router-link>
    </div>
    <router-view></router-view>
</template>
```
