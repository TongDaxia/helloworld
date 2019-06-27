新建一个 Static Web 项目；

```
npm i -g cnpm --registry=https://registry.npm.taobao.org 

npm i -g vue-cli

vue -V

cnpm install -g webpack

vue init webpack demo


```

到这里，描写项目描述时，出错，卡顿没有反应。

解决：npm 8.11 降级为 npm 8.0.

不对。应该是没安装 webpack，安装一下！

`npm install -g webpack`

