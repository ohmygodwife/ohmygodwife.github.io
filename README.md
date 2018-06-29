Forked from [jekyll-theme-next](https://github.com/Simpleyyt/jekyll-theme-next)

```sh
git clone https://github.com/Simpleyyt/jekyll-theme-next.git USERNAME.github.io
cd USERNAME.github.io
git remote set-url origin https://github.com/USERNAME/USERNAME.github.io.git
git push origin master
```

create post by Rakefile referring to [jekyll-bootstrap](https://github.com/plusjade/jekyll-bootstrap)

```sh
$ rake post title="A Title" [date="2012-02-09"] [tags=[tag1,tag2]] [category="category"]
```

some useful markdown syntax

```markdown
![]({{"/assets/images/post/git-command.jpg" | absolute_url }}) #include image
[unsortbin泄漏main_arena](#unsortbin泄漏main_arena "鼠标悬浮文字") #local href
| 标题和内容居左 | 标题和内容居中 | 标题居中内容居左  | 标题和内容居右 | #table
| :----------- | :-----------: | ---------------- | ------------: |
| 居左| 居中| 居中和居左| 居右|
<font face="微软雅黑" color="red" size="3">字体及字体颜色和大小</font>
```

