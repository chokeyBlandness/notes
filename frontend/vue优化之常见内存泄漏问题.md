# vue优化之常见内存泄漏问题

1、监听window/body等事件没有解绑。

2、接口调用不规范。碰到一个项目中调用查询接口时少传参数，否则会造成内存激增。

3、绑定在EventBus上的事件没有解绑。

4、setTimeout和setInterval要关闭。尽量避免频繁使用，会导致生命周期的混乱，可寻找替代方案。

5、模块形成的闭包内部变量使用完后没有置成null。

6、使用$on后需要$off解绑。

7、使用的addEventListener需要解绑。

8、全局变量引起的内存泄漏，在window下声明的变量不会被回收，需要在beforeDestroy或者destroy中置null。

9、使用第三方库，没有在beforeDestroy中做对应的销毁处理。尤其是echarts容易造成页面卡死。

10、递归函数调用未返回。

11、数据量多而引起的内存占用严重。减少数据，简化vue实例对象的数据量，减少内存开销。

12、vue是单页面应用，页面路由切换后，内存未释放。

13、如果在mounted/created 钩子中绑定了DOM/BOM 对象中的事件，需要在beforeDestroy 中做对应解绑处理。

14、js实例用完后未被清理。

15、js内存泄漏：意外的全局变量、被遗忘的计时器或回调函数、脱离DOM的引用。

