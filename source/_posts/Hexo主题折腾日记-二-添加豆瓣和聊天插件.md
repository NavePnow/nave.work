---
title: Hexo主题折腾日记(二) 添加豆瓣和聊天插件
date: 2019-11-18 10:59:11
tags: []
categories:
thumbnail: https://images.unsplash.com/photo-1486312338219-ce68d2c6f44d?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=900&q=60
recommend: 1
top: 100
toc: true
---

# 前言
今上午翻看别人的博客，看到了Next主题相关优化的帖子，有关于豆瓣主页和博客聊天插件的内容，<!--more-->于是就想在 icarus 主题内也实现这个功能，折腾了一下午，终于搞好了。

## 豆瓣主页插件
### 实现效果
<!--more-->
<center>
<img src="https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/screenshot 2019-11-17 at 12.18.22.png" height="60%" width="60%">
</center>

### 配置
1. 安装模块依赖
    ` $ npm install hexo-douban --save`
2. 在站点配置文件中添加配置
    在 `_config.xml` 文件中添加
    ```
douban:
  user: mythsman
  builtin: false
  book:
    title: 'This is my book title'
    quote: 'This is my book quote'
  movie:
    title: 'This is my movie title'
    quote: 'This is my movie quote'
  game:
    title: 'This is my game title'
    quote: 'This is my game quote'
  timeout: 10000
    ```
    
说明:

- user: 你的豆瓣ID.打开豆瓣，登入账户，然后在右上角点击 "个人主页" ，这时候地址栏的URL大概是这样："https://www.douban.com/people/xxxxxx/" ，其中的"xxxxxx"就是你的个人ID了。
- builtin: 是否将生成页面的功能嵌入hexo s和hexo g中，默认是false,另一可选项为true, 如果豆瓣更新频率不高建议选择false，没有必要再每一次部署的时候重新生成豆瓣的页面
- title: 该页面的标题.
- quote: 写在页面开头的一段话,支持html语法.
- timeout: 爬取数据的超时时间，默认是 10000ms ,如果在使用时发现报了超时的错(ETIMEOUT)可以把这个数据设置的大一点。
如果只想显示某一个页面(比如movie)，那就把其他的配置项注释掉即可。

3. 修改/layout/common/article.ejs
 ```diff
<%- list_categories(post.categories, {
                    class: 'has-link-grey ',
                    show_count: false,
                    style: 'none',
                    separator: '&nbsp;/&nbsp;'
                }) %>
                </div>
                <% } %>
+               <% if (post._content && word_count(post._content)) { %>
                    <% if (!has_config('article.readtime') || get_config('article.readtime') === true) { %>
                    <span class="level-item has-text-grey">
                        <% const words = word_count(post._content); %>
                        <% const time = duration((words / 150.0) * 60, 'seconds') %>
                        <%= `${ time.locale(get_config('language', 'en')).humanize() } ${ __('article.read')} (${ __('article.about') } ${ words } ${ __('article.words') })` %>
                    </span>
                <% } %>
+               <% } %>
                <% if (!index && (has_config('plugins.busuanzi') ? get_config('plugins.busuanzi') : false)) { %>
                <span class="level-item has-text-grey" id="busuanzi_container_page_pv">
                    <i class="far fa-eye"></i>
                    <%- _p('plugin.visit', '<span id="busuanzi_value_page_pv">0</span>') %>
 ```
4. 测试并发布
 `hexo douban -bgm && hexo server`
 如果终端没有报错且网页 `http://localhost:4000/books`, `http://localhost:4000/movies`, `http://localhost:4000/movies` 没有问题，就可以直接发布了，这里注意 bgm = books+ games+ movies, 如果前面的配置中只选择了 books，那这里只需要 `hexo douban -b` 即可，其他同理。
 
5. 主题文件修改
在对应主题的 `_config.xml` 文件 menu 模块添加对应的配置，示例如下。
    ```
    menu:
        Home: /
        About: /about
        Articles: /archives
        Gallery: /gallery
        Books: /books
    ```
最后测试发布

## 博客聊天插件
### 实现效果
<center>
<img src="https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/screenshot 2019-11-17 at 20.33.21.png" height="40%" width="40%">
</center>

### 配置
1. 首先需要注册 [Tidio](https://www.tidio.com/) 账号，根据引导填写应用信息。
2. 在个人主页中选择 `Channels -> Live Chat -> Integration` ,复制 JS 代码
<center>
<img src="https://cdn.jsdelivr.net/gh/NavePnow/blog_photo@private/screenshot 2019-11-17 at 20.37.55.png" height="60%" width="60%">
</center>
3. 修改 /layout/layout.ejs, 在文件最后插入对应的代码.
```diff
    <% } %>
+    <script src="//code.tidio.co/token.js" async></script>
</body>
</html>
```
其中将`token`替换成你对应的token即可，接下来可以在 Tidio 控制台的 `Channel -> Live chat -> Appearance` 中根据提示定制聊天对话框的主题外观和语言包，以适应自己的需求。