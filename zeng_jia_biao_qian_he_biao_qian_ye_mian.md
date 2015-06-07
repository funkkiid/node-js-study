# 增加标签和标签页面

现在我们来给博客增加标签和标签页。

我们设定：每篇文章最多有三个标签（少于三个也可以），当点击主页左侧标签页链接时，跳转到标签页并列出所有已存在标签；当点击任意一个标签链接时，跳转到该标签页并列出所有含有该标签的文章。

### [](https://github.com/nswbmw/N-blog/wiki/%E7%AC%AC9%E7%AB%A0--%E5%A2%9E%E5%8A%A0%E6%A0%87%E7%AD%BE%E5%92%8C%E6%A0%87%E7%AD%BE%E9%A1%B5%E9%9D%A2#%E6%B7%BB%E5%8A%A0%E6%A0%87%E7%AD%BE)添加标签

首先我们来实现给文章添加标签的功能。

打开 post.ejs ，在 `<input type="text" name="title" /><br />` 后添加：
    
    标签：<br />
    <input type="text" name="tag1" /><input type="text" name="tag2" /><input type="text" name="tag3" /><br />
    

打开 index.js ，将 `app.post('/post')` 内的：
    
    var currentUser = req.session.user,
        post = new Post(currentUser.name, req.body.title, req.body.post);
    

修改为：
    
    var currentUser = req.session.user,
        tags = [req.body.tag1, req.body.tag2, req.body.tag3],
        post = new Post(currentUser.name, req.body.title, tags, req.body.post);
    

打开 post.js ，将：
    
    function Post(name, title, post) {
      this.name = name;
      this.title= title;
      this.post = post;
    }
    

修改为：
    
    function Post(name, title, tags, post) {
      this.name = name;
      this.title = title;
      this.tags = tags;
      this.post = post;
    }
    

将：
    
    var post = {
        name: this.name,
        time: time,
        title: this.title,
        post: this.post,
        comments: []
    };
    

修改为：
    
    var post = {
        name: this.name,
        time: time,
        title: this.title,
        tags: this.tags,
        post: this.post,
        comments: []
    };
    

现在我们就可以在发表文章的时候添加标签了。接下来我们修改 index.ejs 、 user.ejs 和 article.ejs 来显示文章的标签。

修改 index.ejs 、 user.ejs 和 article.ejs，将：
    
    <p class="info">
      作者：<a href="/u/<%= post.name %>"><%= post.name %></a> | 
      日期：<%= post.time.minute %>
    </p>
    

修改为：
    
    <p class="info">
      作者：<a href="/u/<%= post.name %>"><%= post.name %></a> | 
      日期：<%= post.time.minute %> | 
      标签：
      <% post.tags.forEach(function (tag, index) { %>
        <% if (tag) { %>
          <a class="tag" href="/tags/<%= tag %>"><%= tag %></a>
        <% } %>
      <% }) %>
    </p>
    

最后，在 style.css 中添加如下样式：
    
    .tag{background-color:#ff0000;border-radius:3px;font-size:14px;color:#ffffff;display:inline-block;padding:0 5px;margin-bottom:8px;}
    .tag:hover{text-decoration:none;background-color:#ffffff;color:#000000;-webkit-transition:color .2s linear;}
    

至此，我们给博客添加了标签功能。赶紧看看效果吧！

### [](https://github.com/nswbmw/N-blog/wiki/%E7%AC%AC9%E7%AB%A0--%E5%A2%9E%E5%8A%A0%E6%A0%87%E7%AD%BE%E5%92%8C%E6%A0%87%E7%AD%BE%E9%A1%B5%E9%9D%A2#%E6%B7%BB%E5%8A%A0%E6%A0%87%E7%AD%BE%E9%A1%B5)添加标签页

接下来我们给博客增加标签页。

修改 header.ejs ，在 `<span><a title="存档" href="/archive">archive</a></span>` 下一行添加：
    
    <span><a title="标签" href="/tags">tags</a></span>
    

修改 index.js ，在 `app.get('/archive')` 后添加如下代码：
    
    app.get('/tags', function (req, res) {
      Post.getTags(function (err, posts) {
        if (err) {
          req.flash('error', err); 
          return res.redirect('/');
        }
        res.render('tags', {
          title: '标签',
          posts: posts,
          user: req.session.user,
          success: req.flash('success').toString(),
          error: req.flash('error').toString()
        });
      });
    });
    

打开 post.js ，在最后添加：
    
    //返回所有标签
    Post.getTags = function(callback) {
      mongodb.open(function (err, db) {
        if (err) {
          return callback(err);
        }
        db.collection('posts', function (err, collection) {
          if (err) {
            mongodb.close();
            return callback(err);
          }
          //distinct 用来找出给定键的所有不同值
          collection.distinct("tags", function (err, docs) {
            mongodb.close();
            if (err) {
              return callback(err);
            }
            callback(null, docs);
          });
        });
      });
    };
    

**注意**：这里我们用了 distinct （详见《mongodb权威指南》）返回 tags 键的所有不同值，因为有时候我们发表文章的标签是一样的，所以这样避免了获取重复的标签。

在 views 文件夹下新建 tags.ejs ，添加如下代码：
    
    <%- include header %>
    <% posts.forEach(function (tag, index) { %>
      <a class="tag" href="/tags/<%= tag %>"><%= tag %></a>
    <% }) %>
    <%- include footer %>
    

至此，我们就给博客添加了标签页。

### [](https://github.com/nswbmw/N-blog/wiki/%E7%AC%AC9%E7%AB%A0--%E5%A2%9E%E5%8A%A0%E6%A0%87%E7%AD%BE%E5%92%8C%E6%A0%87%E7%AD%BE%E9%A1%B5%E9%9D%A2#%E6%B7%BB%E5%8A%A0%E7%89%B9%E5%AE%9A%E6%A0%87%E7%AD%BE%E7%9A%84%E9%A1%B5%E9%9D%A2)添加特定标签的页面

现在我们来添加特定标签的页面，即当点击任意一个标签链接时，跳转到该标签页并列出所有含有该标签的文章信息。

修改 post.js ，在最后添加如下代码：
    
    //返回含有特定标签的所有文章
    Post.getTag = function(tag, callback) {
      mongodb.open(function (err, db) {
        if (err) {
          return callback(err);
        }
        db.collection('posts', function (err, collection) {
          if (err) {
            mongodb.close();
            return callback(err);
          }
          //查询所有 tags 数组内包含 tag 的文档
          //并返回只含有 name、time、title 组成的数组
          collection.find({
            "tags": tag
          }, {
            "name": 1,
            "time": 1,
            "title": 1
          }).sort({
            time: -1
          }).toArray(function (err, docs) {
            mongodb.close();
            if (err) {
              return callback(err);
            }
            callback(null, docs);
          });
        });
      });
    };
    

修改 index.js ，在 `app.get('/tags')` 后添加如下代码：
    
    app.get('/tags/:tag', function (req, res) {
      Post.getTag(req.params.tag, function (err, posts) {
        if (err) {
          req.flash('error',err); 
          return res.redirect('/');
        }
        res.render('tag', {
          title: 'TAG:' + req.params.tag,
          posts: posts,
          user: req.session.user,
          success: req.flash('success').toString(),
          error: req.flash('error').toString()
        });
      });
    });
    

在 views 文件夹下新建 tag.ejs ，添加如下代码：
    
    <%- include header %>
    <ul class="archive">
    <% var lastYear = 0 %>
    <% posts.forEach(function (post, index) { %>
      <% if (lastYear != post.time.year) { %>
        <li><h3><%= post.time.year %></h3></li>
      <% lastYear = post.time.year } %>
        <li><time><%= post.time.day %></time></li>
        <li><a href="/u/<%= post.name %>/<%= post.time.day %>/<%= post.title %>"><%= post.title %></a></li>
    <% }) %>
    </ul>
    <%- include footer %>
    

最后，别忘了修改 edit.ejs ，为了保持和 post.ejs 一致。将 edit.ejs 修改为：
    
    <%- include header %>
    <form method="post">
      标题：<br />
      <input type="text" name="title" value="<%= post.title %>" disabled="disabled" /><br />
      标签：<br />
      <input type="text" name="tag1" value="<%= post.tags[0] %>" disabled="disabled" />
      <input type="text" name="tag2" value="<%= post.tags[1] %>" disabled="disabled" />
      <input type="text" name="tag3" value="<%= post.tags[2] %>" disabled="disabled" /><br />
      正文：<br />
      <textarea name="post" rows="20" cols="100"><%= post.post %></textarea><br />
      <input type="submit" value="保存修改" />
    </form>
    <%- include footer %>
    

**注意**：这里我们设定了编辑时不能编辑文章的标题和标签。

现在，我们的博客就增加了标签和标签页的功能。