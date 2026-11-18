# 🎨 vue-canvas · 图片标注选区组件

[![Vue](https://img.shields.io/badge/Vue-2.6-4FC08D?style=flat-square&logo=vue.js)](https://v2.vuejs.org/)
[![Element UI](https://img.shields.io/badge/Element%20UI-2.15-409EFF?style=flat-square)](https://element.eleme.io/)
[![Pages](https://img.shields.io/badge/GitHub%20Pages-在线预览-2EA44F?style=flat-square&logo=github)](#在线预览)

在图片上框选任意多边形区域，输出以 **原图分辨率** 为基准的坐标点集。适合做图像标注、OCR 区域圈选、检测框录入这类场景。

## 在线预览

https://fruge365.github.io/vue-canvas/

## 快速开始

```bash
# 安装依赖
npm i

# 启动开发服务器
npm run dev

# 打包生产版本
npm run build
```

## 核心能力

| 能力 | 说明 |
| --- | --- |
| 自由绘制 | 点击「绘制区域」后在图片上依次打点，自动闭合为多边形 |
| 保存结果 | 点击「保存」输出已绘制区域的坐标集合 |
| 批量清除 | 「全部清除」重置所有选区 |
| 坐标还原 | canvas 按容器缩放显示，但输出坐标按原图尺寸换算（`ratio` 处理） |
| 历史保留 | 已完成的区域画在独立 canvas 上，切换绘制状态不会丢失 |

## 三层 canvas 架构

组件用三块 canvas 叠放，各司其职：

```text
┌─────────────────────────────┐
│ #saveCanvas    已完成的区域   │  ← 落笔后迁移到此层，不会被重绘清掉
├─────────────────────────────┤
│ #drawCanvas    进行中的画笔   │  ← 绑定鼠标事件，绘制时实时反馈
├─────────────────────────────┤
│ #imgCanvas     底图           │  ← 只负责绘制原始图片
└─────────────────────────────┘
```

这样把「底图」「临时笔迹」「历史结果」分离，避免每次重绘都把已完成的内容擦掉。

## 输出格式

保存时输出的数据结构如下，每个区域是一个 `polygon`：

```json
[
  {
    "polygon": {
      "x1": 700, "y1": 273,
      "x2": 975, "y2": 278,
      "x3": 1107, "y3": 368,
      "x4": 718, "y4": 354
    }
  }
]
```

坐标基于 `imgWidth` / `imgHeight`（默认 3020 × 1080）的原图尺寸，与页面显示大小无关。**换图片时必须同步修改这两个值**，否则坐标会偏移。

## 项目结构

```text
vue-canvas/
├─ src/
│  ├─ views/come.vue        核心标注页（三 canvas + 打点逻辑）
│  ├─ components/HelloWorld.vue
│  ├─ router/index.js
│  ├─ api/test.js
│  ├─ assets/               reset.scss、tinymce 皮肤、logo
│  ├─ App.vue
│  └─ main.js
├─ public/index.html
├─ lib-flexible.js          移动端 rem 适配
├─ demo.html
├─ babel.config.js
├─ postcss.config.js
└─ package.json
```

`img/code/wechat-code.jpg` 是默认示例底图，替换成自己的图片时路径写在 `come.vue` 的 `imgUrl` 字段。

## 接入到自己的项目

1. 复制 `src/views/come.vue` 到你的项目
2. 确保已引入 Element UI（`el-button` 用到）
3. 改 `data` 里的 `imgUrl`、`imgWidth`、`imgHeight` 为你的图片与真实尺寸
4. 「保存」按钮的 `savePoints()` 里把 `submitData` 提交到后端即可

## 常见问题

**画出来的框位置和鼠标不一致？**
`imgWidth` / `imgHeight` 与实际图片尺寸不匹配。容器尺寸（`wrapWidth`）可以变，原图尺寸必须是真实值。

**点击没反应？**
`#drawCanvas` 必须在最上层且绑定事件，检查 canvas 的 z-index 与容器 `position` 是否被样式覆盖。

**能改成矩形框吗？**
当前只支持依次打点的多边形。要矩形框可以简化为只记录起止两点，或固定四角坐标。

**移动端能用吗？**
项目引入了 `lib-flexible.js`，但鼠标事件未做 touch 适配，触屏设备需自行补 `touchstart` / `touchmove` 处理。

## 许可

仅供学习与参考使用。
