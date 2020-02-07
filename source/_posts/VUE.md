# VUE

**介绍：vue相对于react，使用起来会更简单，因为vuex, vue router都是官方在维护，比react第三方维护要好很多**

## 目标

1. **了解vue**
2. **搭建vue工程**
3. **webpack相关**



## 快速开始

http://vuejs-templates.github.io/webpack/

## 原理

### 一、VUE实例

挂载vue方法

1. new 方法构造参数里自行制定
2. 通过template模板

### 二、VUE实例的生命周期方法

1. **beforeCreate：初始化vue前**
2. **create：初始化vue**
3. **beforeMount：挂载vue前**
4. **mounted：挂载vue后**
5. 改变前
6. 改变后
7. 销毁前
8. 销毁后

### 三、数据绑定

vue实例和template进行数据交换，template能绑定的数据一定是vue实例里定义的

总结：vue挂载后（mounted）才可以访问当前挂载组件里实例对象，最后完成双向绑定

### 四、computed和watch

1. computed是vue里的一个对象，里面所有的方法都是可以直接在当前vue内使用
   - 只生成新值，不做修改动作
   - 依赖值发生变化，触发
2. watch是监听变化做指定操作

### 五、原生指令

> v-xxx:原生指令,部分指令有缩写方式，例如:id = v-bing:id="xxx"
>
> v-text="xxx" 等价于 {{text}}
>
> V-html：可以解析html标签
>
> V-show：display：none，作用是显示或隐藏节点
>
> v-if：功能类似v-show，但是会触发动态增删节点
>
> v-for：遍历数组（元素，索引），集合（val，key，index）
>
> v-model：数据绑定

### 六、组件

定义组件

1. new VUE()-->把template的内容挂载到el上，
2. .vue文件原理：export出去vue对象(组件)
3. prop属性：定义组件可配置行为，使用组件时要求或当前定义的变量有其作用，外部组件用来约束定义prop组件行为 

注意事项：组件内data要在当前组件内初始化，不可使用全局data。

### 七、组件继承





### VUE-router

使用：导入vue-router，new router定义路由，默认hash路由

## 结构

```bash
.
├── build/                      # webpack config files
│   └── ...
├── config/
│   ├── index.js                # main project config
│   └── ...
├── src/
│   ├── main.js                 # app entry file
│   ├── App.vue                 # main app component
│   ├── components/             # ui components
│   │   └── ...
│   └── assets/                 # module assets (processed by webpack)
│       └── ...
├── static/                     # pure static assets (directly copied)
├── test/
│   └── unit/                   # unit tests
│   │   ├── specs/              # test spec files
│   │   ├── eslintrc            # config file for eslint with extra settings only for unit tests
│   │   ├── index.js            # test build entry file
│   │   ├── jest.conf.js        # Config file when using Jest for unit tests
│   │   └── karma.conf.js       # test runner config file when using Karma for unit tests
│   │   ├── setup.js            # file that runs before Jest runs your unit tests
│   └── e2e/                    # e2e tests
│   │   ├── specs/              # test spec files
│   │   ├── custom-assertions/  # custom assertions for e2e tests
│   │   ├── runner.js           # test runner script
│   │   └── nightwatch.conf.js  # test runner config file
├── .babelrc                    # babel config
├── .editorconfig               # indentation, spaces/tabs and similar settings for your editor
├── .eslintrc.js                # eslint config
├── .eslintignore               # eslint ignore rules
├── .gitignore                  # sensible defaults for gitignore
├── .postcssrc.js               # postcss config
├── index.html                  # index.html template
├── package.json                # build scripts and dependencies
└── README.md                   # Default README file
```

## API













## 扩展

### webpack

####  [WebPack](https://webpack.js.org/concepts/)原理

webpack 用于打包前端资源, 前端资源有很多不同的类型 js, css, img, font ，通过http请求加载，开发webapp时都是一整个js加载到浏览器端之后再把所有的内容渲染出来，无法做到按需加载导致性能低下，通过webpack可以做到提升加载效率



#### webpack功能

配置

插件 