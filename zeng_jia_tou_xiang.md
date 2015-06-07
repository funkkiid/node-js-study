# 增加头像

现在我们来给博客添加用户头像，注册的用户根据注册时的邮箱获取 gravatar 头像，未注册的用户则根据留言填的邮箱获取 gravatar 头像。  
什么是 gravatar ？详情请戳：[http://www.gravatar.com](http://www.gravatar.com/)

我们设定：在主页和用户页的文章标题右侧显示作者头像，在文章页面留言的人的头像显示在留言板左侧。

我们需要用到 Node.js 中的 crypto 模块，之前已经引入过，所以这里可以直接使用。

### [](https://github.com/nswbmw/N-blog/wiki/%E7%AC%AC14%E7%AB%A0--%E5%A2%9E%E5%8A%A0%E5%A4%B4%E5%83%8F#%E6%B7%BB%E5%8A%A0%E5%B7%B2%E6%B3%A8%E5%86%8C%E7%94%A8%E6%88%B7%E7%9A%84%E5%A4%B4%E5%83%8F)添加已注册用户的头像

打开 user.js ，在最上面添加一行代码：
    
    var crypto = require('crypto');
    

然后将 `User.prototype.save` 内的:
    
    var user = {
        name: this.name,
        password: this.password,
        email: this.email
    };
    

修改为:
    
    var md5 = crypto.createHash('md5'),
        email_MD5 = md5.update(this.email.toLowerCase()).digest('hex'),
        head = "http://www.gravatar.com/avatar/" + email_MD5 + "?s=48";
    //要存入数据库的用户信息文档
    var user = {
        name: this.name,
        password: this.password,
        email: this.email,
        head: head
    };
    

这里我们在用户文档里添加了 `head` 键，方便后面使用。

**注意**：需要把 email 转化成小写再编码。

打开 index.js ，将 `app.post('/post')` 中的：
    
    post = new Post(currentUser.name, req.body.title, tags, req.body.post);
    

修改成：
    
    post = new Post(currentUser.name, currentUser.head, req.body.title, tags, req.body.post);
    

修改 post.js ，将：
    
    function Post(name, title, tags, post) {
      this.name = name;
      this.title = title;
      this.tags = tags;
      this.post = post;
    }
    

修改为：
    
    function Post(name, head, title, tags, post) {
      this.name = name;
      this.head = head;
      this.title = title;
      this.tags = tags;
      this.post = post;
    }
    

将：
    
    var post = {
        name: this.name,
        time: time,
        title:this.title,
        tags: this.tags,
        post: this.post,
        comments: [],
        pv: 0
    };
    

修改为：
    
    var post = {
        name: this.name,
        head: this.head,
        time: time,
        title:this.title,
        tags: this.tags,
        post: this.post,
        comments: [],
        pv: 0
    };
    

最后，修改 index.ejs 和 user.ejs ，在 `</h2>` 后添加一行代码：
    
    <a href="/u/<%= post.name %>"><img src="<%= post.head %>" class="r_head" /></a>
    

至此，我们实现了给已注册的用户添加头像的功能。

### [](https://github.com/nswbmw/N-blog/wiki/%E7%AC%AC14%E7%AB%A0--%E5%A2%9E%E5%8A%A0%E5%A4%B4%E5%83%8F#%E6%B7%BB%E5%8A%A0%E6%9C%AA%E6%B3%A8%E5%86%8C%E7%94%A8%E6%88%B7%E7%9A%84%E5%A4%B4%E5%83%8F)添加未注册用户的头像

修改 `app.post('/u/:name/:day/:title')`，将：
    
    var comment = {
        name: req.body.name,
        email: req.body.email,
        website: req.body.website,
        time: time,
        content: req.body.content
    };
    

修改为：
    
    var md5 = crypto.createHash('md5'),
        email_MD5 = md5.update(req.body.email.toLowerCase()).digest('hex'),
        head = "http://www.gravatar.com/avatar/" + email_MD5 + "?s=48"; 
    var comment = {
        name: req.body.name,
        head: head,
        email: req.body.email,
        website: req.body.website,
        time: time,
        content: req.body.content
    };
    

打开 comment.ejs ，将：
    
    <% post.comments.forEach(function (comment, index) { %>
      <p><a href="<%= comment.website %>"><%= comment.name %></a>
      <span class="info"> 回复于 <%= comment.time %></span></p>
      <p><%- comment.content %></p>
    <% }) %>
    

修改为：
    
    <% post.comments.forEach(function (comment, index) { %>
      <div style="padding-left:4em">
        <p><img src="<%= comment.head %>" class="l_head" /><a href="<%= comment.website %>"><%= comment.name %></a>
        <span class="info"> 回复于 <%= comment.time %></span></p>
        <p><%- comment.content %></p>
      </div>
    <% }) %>
    

最后，在 style.css 中添加两行样式：
    
    .l_head{float:left;margin-left:-4em;box-shadow:0px 1px 4px #888;}
    .r_head{float:right;margin-top:-2.5em;box-shadow:0px 1px 4px #888;}