---
title: 博客园和Hexo 加入 Live2D
date: 2019-07-06 16:49:00
tags: 
  - live2d
  - hexo
categories: Hexo
---

今天在查资料时，在这篇博客 [Unity FSM 有限状态机](https://www.cnblogs.com/Firepad-magic/p/6185201.html) 看到了一个有趣的东西 ，屏幕右下角有一个二次元的模型，而且鼠标移到不同位置，模型会跟着动，点击还会播放音频。通过截图使用 google 的图片搜索，原来这个叫做 [Live 2D](https://www.live2d.com/en/) ，最终找到了添加方式，可以在博客园添加，后来发现原来 hexo 也可以添加。



### Live2D简介

Live2D 是一种应用于电子游戏的绘图渲染技术，由日本 Cybernoids 公司开发。

Live2D共有两个分支：**Cubism**（主要）和**Euclid**（已停止开发）。若无特殊说明，Live2D均指Cubism分支。

**工作原理**

`Live2D Cubism` 的工作原理是通过将一系列的 2D 图像进行平移、旋转和变形等操作，生成一个具有自然动画效果的可动人物模型。

<!--more-->

**![img](https://img.moegirl.org/common/2/2b/Live2d02.gif)**





### 博客园添加 Live2D

**起源**

这个做法的发源地是在 [博客美化—给你博客添加一个萌萌的看板娘吧](https://www.cnblogs.com/yjlaugus/p/8724881.html) 这里

似乎需要上传多个文件内容： `waifu.css` 、`waifu-tips.js` 、`live2d.js` 和 `flat-ui.min.css` （若不加菜单可不引入此文件）。

**配置**

后来被简化了许多，下面是精简版的配置方法：

首先，需要申请博客园的 `js 权限` ，步骤是：管理--》设置》--》js权限申请

然后，在 【页面html代码】编辑器中插入如下内容：

- 引入 live2d 的 js：

  ``` html
  <script src="https://eqcn.ajz.miesnfu.com/wp-content/plugins/wp-3d-pony/live2dw/lib/L2Dwidget.min.js"></script>
  ```

- 初始化 js ，加载模型：

  ``` html
  <script>
      L2Dwidget.init({
          "model": {
  　　　　　　　//jsonpath控制显示那个小萝莉模型，下面这个就是我觉得最可爱的小萝莉模型
              jsonPath: "https://unpkg.com/live2d-widget-model-chitose@1.0.5/assets/chitose.model.json",
              "scale": 1
          },
          "display": {
              "position": "right", //看板娘的表现位置
              "width": 150,  //小萝莉的宽度
              "height": 300, //小萝莉的高度
              "hOffset": 0,
              "vOffset": -20
          },
          "mobile": {
              "show": true,
              "scale": 0.5
          },
          "react": {
              "opacityDefault": 0.7,
              "opacityOnHover": 0.2
          }
      });
  </script>
  ```

最后，保存上面修改然后刷新页面就能看到可爱的模型了。

**换模型**

假如希望换成其他的模型，可以修改 `jsonPath` 的路径，格式为：`https://unpkg.com/2D模型全名称@1.0.5/assets/模型.model.json` ，可选的模型名称有：

- live2d-widget-model-chitose
- live2d-widget-model-epsilon2_1
- live2d-widget-model-gf
- live2d-widget-model-haru/01 (use npm install --save live2d-widget-model-haru)
- live2d-widget-model-haru/02 (use npm install --save live2d-widget-model-haru)
- live2d-widget-model-haruto
- live2d-widget-model-hibiki
- live2d-widget-model-hijiki
- live2d-widget-model-izumi
- live2d-widget-model-koharu
- live2d-widget-model-miku
- live2d-widget-model-ni-j
- live2d-widget-model-nico
- live2d-widget-model-nietzsche
- live2d-widget-model-nipsilon
- live2d-widget-model-nito
- live2d-widget-model-shizuku
- live2d-widget-model-tororo
- live2d-widget-model-tsumiki
- live2d-widget-model-unitychan
- live2d-widget-model-wanko
- live2d-widget-model-z16

在这里可以预览各个模型的样子：[截图预览](https://huaji8.top/post/live2d-plugin-2.0/)



### hexo 添加 Live2D

参考 hexo 官方文档 [hexo-helper-live2d/README/中文](<https://github.com/EYHN/hexo-helper-live2d/blob/master/README.zh-CN.md>) 中的操作，大致步骤如下：

- 安装模块：

  ``` shell
  $ npm install --save hexo-helper-live2d
  ```

- 配置：

  向Hexo的 `_config.yml` 文件或主题的 `_config.yml` 文件中添加配置

  ``` yaml
  live2d:
    enable: true
    scriptFrom: local
    pluginRootPath: live2dw/
    pluginJsPath: lib/
    pluginModelPath: assets/
    tagMode: false
    debug: false
    model:
      use: live2d-widget-model-wanko
    display:
      position: right
      width: 150
      height: 300
    mobile:
      show: true
    react:
      opacity: 0.7
  ```

- 模型

  按照官方的说明，可以将模型放在博客工程根目录中，也可以通过 npm install 已经发布到 npm 上的模型。使用第二种方式的话，假如需要添加自定义模型，需要自己先制作发布到 npm ，在 npm install 。因此我还是选择使用第一种方式，步骤如下：

  - 下载模型资源：

    可以在这里 [live2dDemo](https://github.com/summerscar/live2dDemo) 的 assets 目录下获取自己喜欢的模型，可以在这个 [页面](https://summerscar.me/live2dDemo/) 通过修改 `modelName` 然后点击 `GO!` 按钮预览模型。

  - 在博客根目录下创建目录 `live2d_models` ；
  - 进入该目录，新建一个子目录（名称可自定义），并将模型复制到子目录下；
  - 将子目录的名称配置到上面的 `_config.yml` 的 `module.use` 中。



我直接选了一个模型，并集成到了我的 hexo 博客上，可以在这里查看效果 [linshuhe1.github.io](https://linshuhe1.github.io/) 或 [linshuhe1.coding.me](http://linshuhe1.coding.me/)，由于模型资源有点大（2M 左右），而且是从 github （.me 是从 Coding.net 上拉取，会快一些）上获取资源，因此会有点慢。



### 小结

由于我本身就是做游戏客户端开发的，看到 Live 2D 就想到了 Spine 技术，都是使用少量资源的 2D 动画技术，不难看出 Spine 的表现力没有 Live 2D 强，但 Live 2D 似乎是比较耗 CPU 的方式。Live 2D 在很多日系游戏中有被使用到，因为 Live2D 适用于与玩家有交互性的游戏，点击某个区域有特定的反馈。当然，用于制作卡牌游戏的 2D 动画其实也是可行的方案。



### 参考

- [博客美化—给你博客添加一个萌萌的看板娘吧](https://www.cnblogs.com/yjlaugus/p/8724881.html)

- [博客园主页上添加Live 2D模型](https://www.cnblogs.com/dxdblog/p/10255503.html)
- [五种技术选择：2D手游美术实现方案分析](http://www.gamelook.com.cn/2015/04/212359)
- [hexo中next主题添加里lived看板娘（会说话，会换装）](https://blog.csdn.net/dataiyangu/article/details/83021854)