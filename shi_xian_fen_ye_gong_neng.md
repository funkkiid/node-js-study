# 实现分页功能

现在我们给博客的主页和用户页面增加分页功能。

我们设定：主页和用户页面每页最多显示十篇文章。

这里我们要用到 mongodb 的 skip 和 limit 操作，具体可查阅《mongodb权威指南》。

打开 post.js ，把 `Post.getAll` 函数修改如下：
    
    //一次获取十篇文章
    Post.getTen = function(name, page, callback) {
      //打开数据库
      mongodb.open(function (err, db) {
        if (err) {
          return callback(err);
        }
        //读取 posts 集合
        db.collection('posts', function (err, collection) {
          if (err) {
            mongodb.close();
            return callback(err);
          }
          var query = {};
          if (name) {
            query.name = name;
          }
          //使用 count 返回特定查询的文档数 total
          collection.count(query, function (err, total) {
            //根据 query 对象查询，并跳过前 (page-1)*10 个结果，返回之后的 10 个结果
            collection.find(query, {
              skip: (page - 1)*10,
              limit: 10
            }).sort({
              time: -1
            }).toArray(function (err, docs) {
              mongodb.close();
              if (err) {
                return callback(err);
              }
              //解析 markdown 为 html
              docs.forEach(function (doc) {
                doc.post = markdown.toHTML(doc.post);
              });  
              callback(null, docs, total);
            });
          });
        });
      });
    };
    

打开 index.js ，修改 `app.get('/', function(req,res){` 如下：
    
    app.get('/', function (req, res) {
      //判断是否是第一页，并把请求的页数转换成 number 类型
      var page = req.query.p ? parseInt(req.query.p) : 1;
      //查询并返回第 page 页的 10 篇文章
      Post.getTen(null, page, function (err, posts, total) {
        if (err) {
          posts = [];
        } 
        res.render('index', {
          title: '主页',
          posts: posts,
          page: page,
          isFirstPage: (page - 1) == 0,
          isLastPage: ((page - 1) * 10 + posts.length) == total,
          user: req.session.user,
          success: req.flash('success').toString(),
          error: req.flash('error').toString()
        });
      });
    });
    

**注意**：这里通过 `req.query.p` 获取的页数为字符串形式，我们需要通过 `parseInt()` 把它转换成数字以作后用。同时把 `Post.getAll` 改成了 `Post.getTen` 。

修改 `app.get('/u/:name')` 如下：
    
    app.get('/u/:name', function (req, res) {
      var page = req.query.p ? parseInt(req.query.p) : 1;
      //检查用户是否存在
      User.get(req.params.name, function (err, user) {
        if (!user) {
          req.flash('error', '用户不存在!'); 
          return res.redirect('/');
        }
        //查询并返回该用户第 page 页的 10 篇文章
        Post.getTen(user.name, page, function (err, posts, total) {
          if (err) {
            req.flash('error', err); 
            return res.redirect('/');
          } 
          res.render('user', {
            title: user.name,
            posts: posts,
            page: page,
            isFirstPage: (page - 1) == 0,
            isLastPage: ((page - 1) * 10 + posts.length) == total,
            user: req.session.user,
            success: req.flash('success').toString(),
            error: req.flash('error').toString()
          });
        });
      }); 
    });
    

接下来在 views 文件夹下新建 paging.ejs ，添加如下代码：
    
    <br />
    <div>
      <% if (!isFirstPage) { %>
        <span class="prepage"><a title="上一页" href="?p=<%= page-1 %>">上一页</a></span>
      <% } %>
    
      <% if (!isLastPage) { %>
        <span class="nextpage"><a title="下一页" href="?p=<%= page+1 %>">下一页</a></span>
      <% } %>
    </div>
    

这里通过 `if(!isFirstPage)` 判断是否为第一页，不是第一页则显示 “上一页” ；通过`if(!isLastPage)` 判断是否为最后一页，不是最后一页则显示 “下一页” 。

接下来在主页和用户页引入分页。修改 index.ejs 和 user.ejs ，在 `<%- include footer %>` 前添加一行代码：
    
    <%- include paging %>
    

在主页和用户页面引入分页模块。

最后，在 style.css 中添加以下样式：
    
    .prepage a{float:left;text-decoration:none;padding:.5em 1em;color:#ff0000;font-weight:bold;}
    .nextpage a{float:right;text-decoration:none;padding:.5em 1em;color:#ff0000;font-weight:bold;}
    .prepage a:hover,.nextpage a:hover{text-decoration:none;background-color:#ff0000;color:#f9f9f9;-webkit-transition:color .2s linear;}
    

现在，我们实现了博客的分页功能。