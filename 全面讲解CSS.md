# 全面讲解CSS

## 第2章 HTML基础强化

### 2-1 HTML常见元素和理解

* head中：

  * 页面使utf-8编码的字符集

    ```html
    <meta charset="utf-8">
    ```

  * 把屏幕当成一个viewport，就是指视口宽度等于设备宽度，以iphone5为例（宽度为320），就是指定页面以320的宽度来显示，用来适配移动端。

    ```html
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no"
    ```

  * 指定一个基准路径，所有路径都是以这个为基准

    ```html
    <base href="/">
    ```

HTML5新增内容

* 新的区块标签
  * section
  * article
  * nav
  * aside
* 表单增强
  * 日期、时间、搜索
  * 表单验证
  * Placeholder自动聚焦
* 新增语义
  * header/footer 头尾
  * section/article 区域
  * nav导航
  * aside 不重要内容
  * em/strong 斜体/强调
  * i 图标icon

面试题：

1. doctype的意义是什么？

   让浏览器以标准模式渲染

   让浏览器直到元素的合法性



