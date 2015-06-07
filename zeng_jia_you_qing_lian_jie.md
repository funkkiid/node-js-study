# 增加友情链接

现在我们来给博客添加友情链接。

打开 header.ejs ，在：
    
    <span><a title="标签" href="/tags">tags</a></span>
    

下一行添加一行代码：
    
    <span><a title="友情链接" href="/links">links</a></span>
    

修改 index.js ，在 `app.get('/search')` 前添加如下代码：
    
    app.get('/links', function (req, res) {
      res.render('links', {
        title: '友情链接',
        user: req.session.user,
        success: req.flash('success').toString(),
        error: req.flash('error').toString()
      });
    });
    

在 views 文件夹下新建 links.ejs ，添加如下代码：
    
    <%- include header %>
    <ul style="list-style:none">
      <li><h3><a href="https://love.alipay.com/donate/donateSingle.htm?name=201304201216494301">支付宝公益网</a></h3>天佑四川，为雅安地震捐款</li>
      <li><h3><a href="http://www.onefoundation.cn/html/cn/beneficence.html">壹基金</a></h3>壹基金雅安地震救援，刻不容缓！</li>
    </ul>
    <%- include footer %>