# Vue切换页面时中断axios请求

### 一、概述

​	在Vue单页面开发过程中，遇到这样的情况，当我切换页面时，由于上一页面请求执行时间长，切换到该页面时，还未执行完，这时那个请求仍会继续执行直到请求结束，此时将会影响页面性能，并且可能对现在页面的数据显示造成一定影响。所以我们应该在切换页面前中断前面所有请求。

### 二、解决方法

main.js中，重新封装axios请求，在`router.beforeEach`中强制中断请求。

##### 方法一：

采用axios原生方法，在axios的拦截器中添加config，config中配置axios的cancelToken

~~~javascript
const CancelToken = axios.CancelToken;
Vue.$axiosRequestCancelList = [];

axios.interceptors.request.use(config => {
    let globalConfig = {
        cancelToken: new CancelToken(cancel => {
            Vue.$axiosRequestCancelList.push(cancel)
        })
    }
    return Object.assign(globalConfig, config)
}, error => {
    console.log('stop')
    console.log(error)
    return Promise.reject(error)
})

router.beforeEach((to, from, next) => {   //路由切换检测是否强行中断，
    if (Vue.$axiosRequestCancelList.length > 0) {//强行中断时才向下执行
        console.log(Vue.$axiosRequestCancelList[0])
        Vue.$axiosRequestCancelList.forEach(item => {
            item('interrupt');//给个标志，中断请求
        })
    }
    next();
});
~~~

>注意：使用方法一时须在调用axios时补充完整catch方法，不能只有then方法。

##### 方法二：

将axios方法封装为Promise

~~~javascript
Vue.prototype.$http= axios;//Vue函数添加一个原型属性$axios 指向axios,这样vue实例或组件中不用再去重复引用Axios 直接用this.$axios就能执行axios 方法const CancelToken = axios.CancelToken;
Vue.$httpRequestList=[];

Vue.prototype.$ajax = (type, url, data) => {
    return new Promise((resolve, reject) => {   //封装ajax
        var aa = {
            method: type,
            url: url,
            cancelToken: new CancelToken(c => {  //强行中断请求要用到的
                Vue.$httpRequestList.push(c);
            })
        }
        var json = (type == 'get') ? Object.assign(aa, { params: data }) : Object.assign(aa, { data: data });
        var ajax = Vue.prototype.$http(json).then(res => {
            resolve(res);
        }).catch(error => {   //中断请求和请求出错的处理
                if (error.message == "interrupt") {
                    console.log('已中断请求');
                    return;
                } else {
                    reject(error);
                }
            })
        return ajax;
    })
};

router.beforeEach((to, from, next) => {   //路由切换检测是否强行中断，
    if(Vue.$httpRequestList.length>0){        //强行中断时才向下执行
        Vue.$httpRequestList.forEach(item=>{
            item('interrupt');//给个标志，中断请求
        })  
    }
    next();    
});
~~~


