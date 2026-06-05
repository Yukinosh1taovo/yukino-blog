+++

title = 'Hugo'

date = '2026-04-21T13:13:58+08:00'
draft = true

tags = []
categories = []

+++

写作流程命令

`hugo new posts/xxx/index.md`
写内容 + 放图片
`hugo server -D` 预览
`git add .`

`git commit -m "new post"`

`git push`
自动 CI 部署上线



修改archetypes文件夹下的default.md文件 修改使用`hugo new`命令创建新文章的开头模板





## 关于图片

插入图片要把图片放在对应博客文件夹下  并在md插入图片时把前面的路径删掉 只留下图片名

如 【图片】( ~~"C:\Files\repos\yukino-blog\content\post\astrbot\~~ image-20260412104132487.webp") 
