# 基于Vue.js的围棋盘组件

## 起源

[Sabaki](https://github.com/SabakiHQ/Sabaki)是一款精美的围棋软件，它的核心棋盘组件[Shudan](https://github.com/SabakiHQ/Shudan)基于Preact编写。我很喜欢，于是用Vue.js重写了这个组件。

## 效果

![Screenshot](./screenshot.png)

## 功能

- 全面: 动态尺寸、落子动画、棋子标记、指示坐标、热度效果、忙碌效果、死子状态、主题更换等
- 灵活: 每部分可单独配置，代码也独立
- 易用: Vue标准组件，直接import即可用

## 构建

确保环境已安装Git/Node.js/npm，然后运行：

```shell
$ git clone git@github.com:roocky-lab/vue-shudan.git
$ cd vue-shudan
$ npm install
```

在浏览器里预览效果：

```shell
$ npm run serve
```

发布最小体积版本：

```shell
$ npm run build
```
