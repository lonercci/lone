# Hugo+Loveit优化记(转载)


![](https://gitee.com/lonercci/picbed/raw/master/img/20210627142221.svg)

## 写在前面

{{< admonition type=info title="注意" open=true >}}
本文章转载于 [八荒山人的博客](https://www.bahuangshanren.tech/hugo-loveit优化记/)  。本博客采用的 Twikoo 评论系统的 Hugo 兼容方案也源自于大佬的文章。
以及Hugo编译不了的问题解决方案。下载 [Hugo-extend](https://github.com/gohugoio/hugo/releases/download/v0.84.1/hugo_extended_0.84.1_Windows-64bit.zip) 成功解决！非常感谢！
{{< /admonition >}}

## 本地调试时加载评论系统

使用 `hugo server` ,会得到终端的提示：

```shell
Current environment is "development". The "comment system", "CDN" and "fingerprint" will be disabled.
当前运行环境是 "development". "评论系统", "CDN" 和 "fingerprint" 不会启用.
```

{{< admonition type=success title="解决方法" open=true >}}
使用 `hugo server -e production` 命令即可运行生产环境进行调试，就能加载评论系统了。
{{< /admonition >}}

## Console报错找不到 site.webmanifest

{{< admonition type=warning title="一定要处理" open=true >}}
如果不处理的话，会影响网站的打开速度。
{{< /admonition >}}

参考LoveIt主题作者的 [方法](https://hugoloveit.com/zh-cn/theme-documentation-basics/#32-%E7%BD%91%E7%AB%99%E5%9B%BE%E6%A0%87-%E6%B5%8F%E8%A7%88%E5%99%A8%E9%85%8D%E7%BD%AE-%E7%BD%91%E7%AB%99%E6%B8%85%E5%8D%95) 
,到 [Favicon Generator](https://realfavicongenerator.net/) 处理自己的网站图标，最后会下载一个压缩包，包括生成的图标和 `browserconfig.xml` 、 `site.webmanifest` 等文件，将这些文件放到 `blog\themes\LoveIt\static` 中即可。

{{< admonition type=note title="顺嘴一提" open=true >}}
`blog\themes\LoveIt\static` 这个目录里的文件，最后会出现在网站根目录中。
{{< /admonition >}}

## LoveIt扩展Shortcodes

更多扩展Shortcodes的应用方法请查看LoveIt主题作者写的 [使用说明](https://hugoloveit.com/zh-cn/theme-documentation-extended-shortcodes/) 。

### admonition

admonition比较常用，有12个样式，但是主题作者并没说明每种样式对应的 `type` 是什么。我从源码中找到了它们的对应关系，在此记录一下。

#### 用法

```markdown
{{</* admonition type=tip title="This is a tip" open=true */>}}
一个"技巧"横幅
{{</* /admonition */>}}

或者

{{</* admonition tip "This is a tip" true */>}}
一个"技巧"横幅
{{</* /admonition */>}}
```

#### 示例

{{< admonition type=note title="注意" open=true >}}
type=note
{{< /admonition >}}

{{< admonition type=abstract title="摘要" open=true >}}
type=abstract
{{< /admonition >}}

{{< admonition type=info title="信息" open=true >}}
type=info
{{< /admonition >}}

{{< admonition type=tip title="技巧" open=true >}}
type=tip
{{< /admonition >}}

{{< admonition type=success title="成功" open=true >}}
type=success
{{< /admonition >}}

{{< admonition type=question title="问题" open=true >}}
type=question
{{< /admonition >}}

{{< admonition type=warning title="警告" open=true >}}
type=warning
{{< /admonition >}}

{{< admonition type=failure title="失败" open=true >}}
type=failure
{{< /admonition >}}

{{< admonition type=danger title="危险" open=true >}}
type=danger
{{< /admonition >}}

{{< admonition type=bug title="Bug" open=true >}}
type=Bug
{{< /admonition >}}

{{< admonition type=example title="示例" open=true >}}
type=example
{{< /admonition >}}

{{< admonition type=quote title="引用" open=true >}}
type=quote
{{< /admonition >}}

## keywords不生效

参考自 [雨临Lewis](https://lewky.cn/posts/hugo-4.html/#%E7%BD%91%E7%AB%99%E9%85%8D%E7%BD%AE%E4%BA%86keywords%E6%B2%A1%E6%9C%89%E7%94%9F%E6%95%88) 的这篇文章。

{{< admonition type=note title="前提配置" open=true >}}
在站点配置文件 `config.toml` 中填好网站关键词：

```toml
  # 网站关键词
  keywords = "keyword1,keyword2"
```

{{< /admonition >}}

虽然已经设置了 `keywords` ，但是F12查看网站源码后发现缺少 `keywords` 这个 `meta` 标签，而且在 [站长工具](https://seo.chinaz.com) 里查询站点时发现页面TDK信息里的关键词 `KeyWords` 为空。

{{< admonition type=bug title="debug" open=true >}}
检查模板文件后发现是LoveIt主题没有引入该标签，需要修改模板。
{{< /admonition >}}

### 解决方法

将 `blog\themes\LoveIt\layouts\partials\head\meta.html` 复制到 `blog\layouts\partials\head\meta.html` ，打开该文件并在

```html
<meta name="Description" content="{{ $params.description | default .Site.Params.description }}">
```

的上方添加如下代码：

```html
<meta name="keywords" content="{{ $params.keywords | default .Site.Params.keywords }}">
```

### 参考链接

更多踩坑记录请参考雨临Lewis的 [这篇文章](https://lewky.cn/posts/hugo-4.html/) 。

更多优化美化指南请参考雨临Lewis的 [这篇文章](https://lewky.cn/posts/hugo-3.html/) 。

{{< admonition type=info title="注意" open=true >}}
上面雨临Lewis的两篇文章中有许多地方对于 [LoveIt_v0.2.10](https://github.com/dillonzq/LoveIt/releases/tag/v0.2.10) 是不必要的。
{{< /admonition >}}

## 换用twikoo评论系统

最开始用的评论系统是 [valine](https://valine.js.org/) ，后来换用了带有后台的 [waline](https://waline.js.org/) ，再之后发现 [twikoo](https://twikoo.js.org/) 后台配置很方便，界面也很好看，于是决定换一波。

但有个问题，twikoo只适配了Hexo的部分主题，而没有适配Hugo主题。

### 解决方法

可以修改评论系统模板文件 `blog\themes\LoveIt\layouts\partials\comment.html` 来手动添加对twikoo的支持，在 `<div id="comments"></div>` 中添加以下代码：

```html
        {{- /* twikoo Comment System */ -}}
        {{- $twikoo := $comment.twikoo}}
        {{- if $twikoo.enable -}}
        <div id="tcomment"></div>
        <script src="https://cdn.jsdelivr.net/npm/twikoo@1.3.1/dist/twikoo.all.min.js"></script>
        <script>
        twikoo.init({
          envId: '',
          // 此处填写您的环境id
          el: '#tcomment',
          // region: 'ap-guangzhou', // 环境地域，默认为 ap-shanghai，如果您的环境地域不是上海，需传此参数
          // path: 'window.location.pathname', // 用于区分不同文章的自定义 js 路径，如果您的文章路径不是 location.pathname，需传此参数
        })
        </script>
        {{- end }}
```

然后在博客配置文件 `blog\config.toml` 中的

```toml
# 评论系统设置
[params.page.comment]
    enable = true
```

下面添加

```toml
# twikoo评论系统
[params.page.comment.twikoo]
    enable = true
```

{{< admonition type=tip title="更多twikoo配置" open=true >}}
云部署及版本更新等信息，请到twikoo [官网](https://twikoo.js.org/) 查看。
{{< /admonition >}}

## 部署方式

### 几种选择

1. GitHub Actions
2. [CircleCI](https://circleci.com/) 、[Netlify](https://www.netlify.com/) 、[Travis CI](https://www.travis-ci.com/) 、[Vercel](https://vercel.com/) 等第三方服务

最后还是选了GitHub Actions，因为不用到另外的网站上再配置一通，使用 [actions-hugo](https://github.com/peaceiris/actions-hugo) 和 [actions-gh-pages](https://github.com/peaceiris/actions-gh-pages) 这两个Action，每次写完博客，push一下，GitHub Actions就会自动构建和部署博客。

### GitHub Actions自动部署Hugo

参考 [actions-hugo](https://github.com/peaceiris/actions-hugo) 和 [actions-gh-pages](https://github.com/peaceiris/actions-gh-pages) 

1. 创建一个私有仓库用来存放博客源码。
2. 创建一个公有仓库用来发布博客。
3. 创建一个个人 [token](https://github.com/settings/tokens) ，名字可以叫做 `blog` 。
4. 此token的访问范围选择 `repo` 和 `workflow` 。
5. 生成token，记住它的值。
6. 到博客源码仓库的 `Settings` → `Secrets` 中新建一个 `Actions secrets` ,名字也叫做 `blog` ,Value填入上一步中的个人token的值。
7. 在博客源码仓库的根目录下创建 `.github/workflows/gh-pages.yml` 文件，写入以下代码，然后提交：

```yaml
name: Deploy Blog #名字随便起

on:
  push:
    branches:
      - master #源码所在分支

jobs:
  deploy:
    runs-on: ubuntu-20.04
    steps:
      - uses: actions/checkout@v2

      - name: Setup Hugo #安装hugo
        uses: peaceiris/actions-hugo@v2.4.13 #使用peaceiris开发的actions-hugo
        with:
          hugo-version: 'latest' #可以指定版本号，也可以使用latest表示最新版
          extended: true #支持hugo的扩展版

      - name: Build #使用hugo构建博客
        run: hugo --gc --minify

      - name: Deploy #部署博客
        uses: peaceiris/actions-gh-pages@v3.7.3 #使用peaceiris开发的actions-gh-pages
        with:
          personal_token: ${{ secrets.blog }} 
          external_repository: #用来发布博客的公有仓库
          publish_branch: master
          publish_dir: ./public
          cname: #填写你的域名
          commit_message: ${{ github.event.head_commit.message }}

```

{{< admonition type=success title="双仓库模式" open=true >}}
本博客即采用上面的 `gh-pages.yml` ，使用私有仓库存放博客源码，将Hugo构建好的 `public` 目录推送到公有仓库来发布。
{{< /admonition >}}

{{< admonition type=warning title="双分支模式" open=true >}}
也可以在一个公有仓库中创建两个分支，一个放源码，一个用来发布，但是那样会暴露源码中一些服务的ID和Key。
{{< /admonition >}}


