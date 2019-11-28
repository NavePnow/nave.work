---
title: Hexo主题折腾日记(一) 从cactus到icarus
date: 2019-11-16 10:59:11
tags: [JavaScript]
categories:
thumbnail: https://images.unsplash.com/photo-1486312338219-ce68d2c6f44d?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=900&q=60
recommend: 1
top: 100
toc: true
---

# 前言
从Hexo建站开始，一直是使用的cactus主题，很喜欢那种简约的风格，主页文章预览的都没有的那一种。<!--more-->
<center>
<img src="https://raw.githubusercontent.com/NavePnow/blog_photo/master/IMAGE 2019-11-16 11:11:11.jpg" height="60%" width="60%">
</center>

直到昨天心血来潮想搞一个基于ghost的动态博客，同时爱上了他的默认主题casper，心想这不就是我梦寐已久的主题么，结果搜了很多的资料，发现个人维护的建站工具都是至少1年以上的，有一个基于 python 可以通过ghost部署到github pags上的工具还是基于2.7,我直接裂开，最终直接放弃了这套方案，回过头来想能不能把casper移植到hexo上呢，在github上也搜到了相应的项目，但效果emmm，不是特别能让我满意，贴一张demo供大家参考。
<center>
<img src="https://raw.githubusercontent.com/NavePnow/blog_photo/master/IMAGE 2019-11-16 11:10:20.jpg" height="60%" width="60%">
</center>

主要还是插件的支持度没有成熟主题的高，然后就又开始google: hexo主题推荐2019类似的关键词，终于发现了这个模版，icarus，没有cactus那么朴素，插件支持度和commit活跃度也比hexo-casper高，所以昨天花了很长很长的时间调整网页布局和文章渲染问题，最终效果我打9分吧，因为还有一点问题没有解决，等写完这个博客我再搞，下面主要把我基于别人修改的模版和自己修改的内容做一下总结，以免以后git pull之后不知道自己做了哪些修改。

# icarus主题之上主要改动
1. 主页显示两栏widget，文章只显示左边widget
2. 文章图片居中
3. 增加profile下面的 bio, 可以放一点自己想说的话
4. 置顶文章
5. 文章底部文章详细信息显示以及推荐文章模块配置
6. 页脚访问人数显示修改

## 主页显示两栏widget，文章只显示左边widget
主要参考 [水寒blog](https://dp2px.com/2019/06/04/icarus-theme/)
### 配置
1. 修改 /includes/helpers/layout.js

    ```diff
        hexo.extend.helper.register('column_count', function () {
         let columns = 1;
        +        if (this.page.__post === true || this.page.__page === true) {
        +            return 2;
        +        }
        const hasColumn = hexo.extend.helper.get('has_column').bind(this);
        columns += hasColumn('left') ? 1 : 0;
        columns += hasColumn('right') ? 1 : 0;
    ```
    
2. 修改 /layout/common/widget.ejs

    ```diff
        <%- partial('widget/' + widget.type, { widget, post: page }) %>
        <% }) %>
        <% if (position === 'left') { %>
        -        <div class="column-right-shadow is-hidden-widescreen <%= sticky_class('right') %>">
        +        <div class="column-right-shadow <%= (page.__page !== true && page.__post !== true) ? 'is-hidden-widescreen' : '' %> <%= sticky_class('right') %>">
         <% get_widgets('right').forEach(widget => {%>
             <%- partial('widget/' + widget.type, { widget, post: page }) %>
         <% }) %>
    ```
3. 修改 /layout/layout.ejs

    ```diff
         <div class="columns">
                 <div class="column <%= main_column_class() %> has-order-2 column-main"><%- body %></div>
                 <%- partial('common/widget', { position: 'left' }) %>
        +                <% if (page.__page !== true && page.__post !== true) { %>
                 <%- partial('common/widget', { position: 'right' }) %>
        +                <% } %>
             </div>
         </div>
         </section>
    ```
    
4. 修改 /source/css/style.styl 
    
    ```
    @media screen and (min-width: screen-widescreen)
        .is-1-column .container
        .is-2-column .container
            max-width: screen-widescreen - 2 * gap
            width: screen-widescreen - 2 * gap
    @media screen and (min-width: screen-fullhd)
        .is-2-column .container
            max-width: screen-fullhd - 2 * gap
            width: screen-fullhd - 2 * gap
        .is-1-column .container
            max-width: screen-desktop - 2 * gap
            width: screen-desktop - 2 * gapp
    ```

1. 修改 /layout/layout.ejs
    ```
         <% function main_column_class() {
        switch (column_count()) {
            case 1:
                return 'is-12';
            case 2:
                return 'is-8-tablet is-9-desktop is-9-widescreen';
            case 3:
                return 'is-8-tablet is-8-desktop is-6-widescreen'
        }
        return '';
        } %>
    ```
6. 修改 /layout/common/widget.ejs
    ```
        <% function side_column_class() {
        switch (column_count()) {
        case 2:
            return 'is-4-tablet is-3-desktop is-3-widescreen';
        case 3:
            return 'is-4-tablet is-4-desktop is-3-widescreen';
        }
        return '';
        } %>
    ```

## 文章图片居中
最开始尝试了修改 source/js/main.js 和 layout/css/style.styl , 修改的内容也是基于水寒的博客，本地 hexo server没有问题，但是 hexo g -d 之后就总是出问题，最后还是老老实实
    ```
    <center> 
    </center>
    ```

## 增加profile下面的 bio, 可以放一点自己想说的话
### 实现效果
<center>
<img src="https://raw.githubusercontent.com/NavePnow/blog_photo/master/screenshot 2019-11-16 at 11.47.59.png" height="40%" width="40%">
</center>

### 配置
修改 /layout/widget/profile.ejs, 在最后加上你想说的话
    ```diff
            <% } %>
        </div>
        <% } %>
    +        <hr>
    +        <p id="evan">修子也好，远野也好，对于情感世界发生的事，很难简单以对和错来衡量，在这样的世界里沉浮，飘落的是情感，不败的总是每年盛开的樱花。    --《情人》</p>
    </div>
    </div>
    ```

## 置顶文章
参考文章: `https://removeif.github.io/2019/09/19/%E5%8D%9A%E5%AE%A2%E6%BA%90%E7%A0%81%E5%88%86%E4%BA%AB.html#more`

[github commit history](https://github.com/removeif/hexo-theme-icarus-removeif/commit/a924e02916607ed351904e4833c541199807482d)

### 实现效果
<center>
<img src="https://raw.githubusercontent.com/NavePnow/blog_photo/master/screenshot 2019-11-16 at 13.30.30.png" height="40%" width="40%">
</center>

### 使用说明
在每篇文章的top comment部分配置top字段，初始值是100，如果要置顶，需要设置为大于100的值，值越大越靠前。相等时，根据时间降序。具体设置如下  
    ```
    title: 一亩三分地自动签到脚本
    top: 102
    toc: true
    recommend: 1 
    date: 2019-09-19 22:10:43
    thumbnail: https://cdn.jsdelivr.net/gh/removeif/blog_image/img/2019/20190919221611.png
    tags: 
    categories: 
    ```

### 配置
修改 /layout/common/article.ejs
    ```diff
        <% if (post.layout != 'page') { %>
        <div class="level article-meta is-size-7 is-uppercase is-mobile is-overflow-x-auto">
            <div class="level-left">
        +       <% if(post.top > 100) { %>
        +       <div class="level-item tag is-danger" style="background-color: #3273dc;">Pin</div>
        +       <%} %>
                <time class="level-item has-text-grey" datetime="<%= date_xml(post.date) %>"><%= date(post.date) %></time>
                <% if (post.categories && post.categories.length) { %>
                <div class="level-item">
    ```

## 文章底部文章详细信息显示以及推荐文章模块配置
参考文章: `https://removeif.github.io/2019/09/19/%E5%8D%9A%E5%AE%A2%E6%BA%90%E7%A0%81%E5%88%86%E4%BA%AB.html#more`

[github commit history](https://github.com/removeif/hexo-theme-icarus-removeif/commit/8fb8c23b8e3861fd56aa983f3eac8b0dbe18162d)

### 实现效果
<center>
<img src="https://raw.githubusercontent.com/NavePnow/blog_photo/master/screenshot 2019-11-16 at 11.59.42.png" height="80%" width="80%">
</center>

### 使用说明
在每篇文章的top comment部分配置recommend值（必须大于0），越大越靠前，相等取最新的，最多取5条。具体设置如下  
    ```
    title: 一亩三分地自动签到脚本
    top: 102
    toc: true
    recommend: 1 
    date: 2019-09-19 22:10:43
    thumbnail: https://cdn.jsdelivr.net/gh/removeif/blog_image/img/2019/20190919221611.png
    tags: 
    categories: 
    ```

### 配置
1. 在 languages/xx.yml 中插入 recommend_posts，具体如下
    ```diff
        widget:
            follow: 'Follow'
            recents: 'Recent'
        +   recommend_posts: 'Recommend Posts'
            links: 'Links'
            tag_cloud: 'Tag Cloud'
            catalogue: 'Catalogue'
    ```
注意所使用的语言所对应的文件

1. 修改 /layout/common/article.ejs 
    ```diff
        <div class="level-start">
            <div class="level-item">
        -        <span class="is-size-6 has-text-grey has-mr-7">#</span>
        +        <i class="fas fa-tags has-text-grey"></i>&nbsp;
                <%- list_tags(post.tags, {
                    class: 'has-link-grey ',
                    show_count: false,
                    
        ..........

            </div>
        </div>
        <% } %>
        + <!-- 部分参考自https://www.alphalxy.com/2019/03/customize-icarus/ -->
        + <% if (!index && post.layout === 'post' && post.copyright !== false) { %>
            + <ul class="post-copyright">
            + <li><strong>本文标题：</strong><a href="<%= post.permalink %>"><%= page.title %></a></li>
            + <li><strong>本文作者：</strong><a href="<%= theme.url %>"><%= theme.author %></a></li>
            + <li><strong>本文链接：</strong><a href="<%= post.permalink %>"><%= post.permalink %></a></li>
            + <li><strong>版权声明：</strong>本博客所有文章除特别声明外，均采用 <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh" rel="external nofollow" target="_blank">CC BY-NC-SA 4.0</a> 许可协议。转载请注明出处！
            + </li>
            + </ul>
            + <br>
            + <%- _partial('widget/recommend_posts') %>
           + <br>
        + br>
         + %>
        <% if (!index && has_config('share.type')) { %>
        <%- _partial('share/' + get_config('share.type')) %>
        <% } %>
    ```

3. /layout/widget 目录下添加 recommend_posts.ejs 
    ```
        <span class="is-size-6 has-text-grey has-mr-7">#&nbsp;<%= __('widget.recommend_posts') %></span>
        <br>
        <% var i = 1;posts.forEach(post => { %>
        &nbsp;<%=i %>.<a href="<%- url_for((post.link?post.link:post.path)) %>" class="is-size-6" target="_blank"><%= post.title %></a><br>
        <% i++;}) %> 
    ```

4. /layout/widget 目录下添加 recommend_posts.locals.js
    ```
        module.exports = (ctx, locals) => {
        const { has_config, get_config, get_thumbnail } = ctx;
        const { posts } = ctx.site;
        if (!posts.length) {
        return null;
        }
        const thumbnail = !has_config('article.thumbnail') || get_config('article.thumbnail') !== false;
        const _posts = posts.filter((item, index, arr) => item.recommend != undefined && item.recommend > 0).sort('recommend',-1).sort('recommend',-1).limit(5).map(post => ({
        link: post.link,
        path: post.path,
        title: post.title,
        date: post.date,
        thumbnail: thumbnail ? get_thumbnail(post) : null,
        // fix circular JSON serialization issue
        categories: () => post.categories
        }));
        return Object.assign(locals, { thumbnail, posts: _posts });
        } 
    ```

## 页脚访问人数显示修改
### 实现效果  
<center>
<img src="https://raw.githubusercontent.com/NavePnow/blog_photo/master/screenshot 2019-11-17 at 18.25.58.png">
</center>

### 配置
修改 /layout/common/footer.ejs 卜算子部分.
```
<% if (busuanzi) { %>
                <br>
                <span id="busuanzi_container_site_uv">
                <!-- <%- _p('plugin.visitor', '<span id="busuanzi_value_site_uv">0</span>') %>
                </span>
                <br> -->
                <span id="busuanzi_container_site_pv">
                    Visited by <span id="busuanzi_value_site_uv"></span> users with <span id="busuanzi_value_site_pv"></span> times
                </span>
                </span>
                <% } %>
```