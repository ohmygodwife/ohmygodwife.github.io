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

enable Latex in header of post:

```markdown
mathjax: true
```

some useful Latex
https://detexify.kirelabs.org/classify.html #画图查
https://blog.csdn.net/zgj926503/article/details/52757631
```latex
\tag{1}
\varphi #欧拉函数
\leq #小于等于
\geq #大于等于
\equiv #恒等
\neq #不等于
\approx #约等
\nmid #不整除
\int_{-N}^{N} e^x\, dx #积分
\sum_{i=1}^k #求和
\prod_{i=1}^k #求积
\pi #π
\sqrt[n]{m} #对m开n次方
\frac{3}{5} #分数
\bmod #模
\infty #无穷大
\in #属于
\alpha \beta \gamma \delta \zeta \lambda \mu \epsilon #https://blog.csdn.net/xxzhangx/article/details/52778539
\lvert \rvert #||
#matrix
\left[
\begin{matrix}
 1 & 2 & 3\\
 4 & 5 & 6\\
\end{matrix}
\right]
#list
\left\{\begin{array}{c} a \\ b \\ \vdots \\ c \end{array} \right.
#cases
\begin{cases} 1, &p\nmid a\\
-1, &p\nmid a\\
0, &p\mid a\end{cases}
```

