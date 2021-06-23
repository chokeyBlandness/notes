# IE兼容性问题

### 1、IE不支持Object.assign方法

在调用`Object.assign`方法时，IE浏览器报错无法执行。

方法一：在打包时使用相关插件进行兼容性处理，例如babel打包时可使用`@babel/plugin-transform-object-assign`插件。

方法二：可在调用方法前加上如下处理，已达到将其变为可用方法的目的。

```javascript
if (typeof Object.assign != 'function') {
  Object.assign = function(target) {
    'use strict';
    if (target == null) {
      return target;// 可做异常处理
    }
    target = Object(target);
    for (var index = 1; index < arguments.length; index++) {
      var source = arguments[index];
      if (source != null) {
        for (var key in source) {
          if (Object.prototype.hasOwnProperty.call(source, key)) {
            target[key] = source[key];
          }
        }
      }
    }
    return target;
  };
}
```
