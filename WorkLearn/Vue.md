# Vue小知识
## Vue3 ref和reactive的区别
- 对于响应式而言，ref支持对象类型和基本类型，而reactive只支持对象类型
- 当ref是对象类型的时候，本质上也是一个reactive
- 对象类型的ref和reactive，其响应式底层原理都是Proxy
