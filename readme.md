### 异地部署博客环境

1.确保安装git、node.js、hexo
>https://git-scm.com/   
https://nodejs.org/en/    
https://hexo.io/zh-cn/docs/     
npm install hexo-cli -g   

2.进入本地仓库执行hexo安装。不需要init！否则会覆盖原有配置。   
>npm install    

3.相关操作

- 通过 git clone 的是文档 hexo 分支，里面包含所有文章的版本记录，即文章的同步需要依靠 git 版本来管理

- hexo deploy 该功能是 hexo 博客系统的发布，其通过 git push 到 master 分支，实现实际博客界面的更新

- 注：在编译好新文章并成功 hexo d 之后，记得同步提交本地文件夹中未受版本控制的新文件

### 基本操作


```
- hexo clean 清除项目
- hexo g (generate) 构建项目
- hexo server 开启本地 hexo 服务
- hexo d (deploy) 发布更新
- hexo new [layout] <title> 创建新文件
- hexo new draft 草稿 发布草稿文件
· hexo publish [layout] <title> 将草稿文件转移到发布文件
```