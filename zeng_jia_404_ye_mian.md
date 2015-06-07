# 增加404页面

现在我们来给博客添加 404 页面，即当访问的路径都不匹配时，跳转到 404 页面。

打开 index.js ，在：

function checkLogin(req, res, next){ ... }
前添加如下代码：

app.use(function (req, res) {
  res.render("404");
});
在 views 文件夹下新建 404.ejs ，添加如下代码：

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Blog</title>
</head>
<body>
<script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8"></script>