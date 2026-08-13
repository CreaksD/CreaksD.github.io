我写 Markdown → 发布 → 全世界能访问 → 长期保存

那么最适合你的就是：

Hugo + GitHub + GitHub Pages

完全可以做到，而且前期基本 0 成本。

一、最终效果

你最终会得到一个：

https://你的用户名.github.io

例如：

https://shenghua.github.io

全世界任何人都可以访问。

你的工作流就是：

写文章
   ↓
Markdown
   ↓
git push
   ↓
GitHub Actions 自动构建
   ↓
GitHub Pages
   ↓
全世界访问
然后让 GitHub 自动发布

这一步使用：

GitHub Actions + GitHub Pages

你的仓库里面创建：

.github/
└── workflows/
    └── hugo.yaml

这个 Workflow 负责：

你 git push
      ↓
GitHub Actions
      ↓
安装 Hugo
      ↓
hugo build
      ↓
生成静态网站
      ↓
部署 GitHub Pages

之后你甚至不用手动执行 Hugo。
以后你写文章就非常简单

你的日常工作流可以变成：

# 创建文章
hugo new posts/redis-cache.md

# 写文章
code content/posts/redis-cache.md

# 本地预览
hugo server -D

# 发布
git add .
git commit -m "add redis cache article"
git push

然后等待 GitHub Actions。

最终：

https://你的用户名.github.io/posts/redis-cache/