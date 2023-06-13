
## 2023-06-14 更新 ※
1. 为了在 GitHub 内查看或修改文章更符合直觉，文章头部用于 Hexo 解析的 YAML Front Matter 已被移除，文章信息统一存在 `headers.json` 中。
2. 本博客站的文章信息记录形式（文章信息存储在 `headers.json`）与 Hexo 要求 `source` 目录要求的形式（文章信息存储在 YAML Front Matter）的相互转化工具已初步开发完成，当前仓库的转化即使用此工具完成。转化工具项目地址为 [hexo-source-tool](https://github.com/gukaifeng/hexo-source-tool-2023)，此工具未来会加入更多功能。
3. 目前，若某文章在 `headers.json` 缺少信息，则 hexo-source-tool 会在将文章转化为 Hexo 要求的 `source` 形式时，将缺少的文章信息从其 Git 信息中补全。目前支持标题 `title`、发布时间 `date` 和更新时间 `updated` 字段，当 `header.json` 中缺少某文章的这些字段时，`title` 将补全为文件名（`.md` 前的部分），`date` 为此文章第一次 commit 的时间，`updated` 为此文章最后一次 commit 的时间。


---

Hi there 👋，这是我的博客 gukaifeng.cn 的资源仓库。

博客主页 https://gukaifeng.cn  
这个仓库存储了博客站的全部文章以及文章所用到资源。

博客由 [hexo](hexo.io) 驱动，  
此仓库的内容与 hexo 本地根目录下 source 文件夹的内容一致。

博客的评论系统由 [utterances](https://utteranc.es/) 驱动，由于 utterances 依赖 GitHub issues，  
因此你需要确保你的网络可以正常访问 GitHub issues，才可正常使用这个博客的评论功能。  

大部分博客文章都在 \_posts 文件夹内。

目前文章头部有用于 hexo 解析的信息，后面我会自己解析这些信息，使 GitHub 中的文章内容更纯粹。  
（代码已经写好了，因为懒，一直没部署）

<!-- 大部分博客文章都在 \_posts 文件夹内，对应的文章链接是 `https://gukaifeng.cn/posts/[文章名字]`，   -->
<!-- 文章名字就是 markdown 文件去掉后缀 `.md` 剩下的部分，直接用中文就好，空格用横线 `-` 替代。 -->



本站文章的发布与更新流程是，  
我在本地编辑 markdown📝，然后 push 到此仓库🏡，触发 GitHub Webhook ，  
随即博客站服务器拉取此仓库中的最新内容，同步更新♻️。

依据此原理，如果你发现了某篇文章中的错误，你可以在这个仓库中发起一个 pull request，当你的 pull request 被合并进 main 分支时，博客站 gukaifeng.cn 的内容随即会自动更新。
