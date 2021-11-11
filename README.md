Hi there 👋，这是我的博客 gukaifeng.cn 的资源仓库。

博客主页 https://gukaifeng.cn  
这个仓库存储了博客站的全部文章以及文章所用到资源。

博客由 [hexo](hexo.io) 驱动，  
此仓库的内容与 hexo 本地根目录下 source 文件夹的内容一致。

博客的评论系统由 [utterances](https://utteranc.es/) 驱动，由于 utterances 依赖 GitHub issues，  
因此你需要确保你的网络可以正常访问 GitHub issues，才可正常使用这个博客的评论功能。  

大部分文章都在 \_posts 文件夹内，对应的文章博客链接是 `https://gukaifeng.cn/posts/[文章名字]`，  
文章名字就是 markdown 文件去掉后缀 `.md` 剩下的部分，直接用中文就好，空格用横线 `-` 替代。

---

本站文章的发布与更新流程是，我在本地编辑 markdown📝，然后 push 到此仓库🏡，触发 GitHub Webhook🪝，随即博客站服务器拉取此仓库中的最新内容，同步更新♻️。

依据此原理，如果你发现了某篇文章中的错误，你可以在这个仓库中发起一个 pull request，当你的 pull request 被合并进 main 分支时，博客站 gukaifeng.cn 的内容随即会自动更新。