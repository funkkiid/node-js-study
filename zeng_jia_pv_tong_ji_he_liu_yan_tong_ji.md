# 增加pv统计和留言统计

现在我们来给每篇文章增加 pv 统计和留言统计。

我们设定：在主页、用户页和文章页均显示 pv 统计和留言统计。

修改 post.js ，将：
    
    var post = {
        name: this.name,
        time: time,
        title:this.title,
        tags: this.tags,
        post: this.post,
        comments: []
    };
    

修改为：
    
    var post = {
        name: this.name,
        time: time,
        title:this.title,
        tags: this.tags,
        post: this.post,
        comments: [],
        pv: 0
    };
    

**注意**：我们给要存储的文档添加了 pv 键并直接赋初值为 0。

打开 post.js ，将 `Post.getOne()` 修改为：
    
    //获取一篇文章
    Post.getOne = function(name, day, title, callback) {
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
          //根据用户名、发表日期及文章名进行查询
          collection.findOne({
            "name": name,
            "time.day": day,
            "title": title
          }, function (err, doc) {
            if (err) {
              mongodb.close();
              return callback(err);
            }
            if (doc) {
              //每访问 1 次，pv 值增加 1
              collection.update({
                "name": name,
                "time.day": day,
                "title": title
              }, {
                $inc: {"pv": 1}
              }, function (err) {
                mongodb.close();
                if (err) {
                  return callback(err);
                }
              });
              //解析 markdown 为 html
              doc.post = markdown.toHTML(doc.post);
              doc.comments.forEach(function (comment) {
                comment.content = markdown.toHTML(comment.content);
              });
              callback(null, doc);//返回查询的一篇文章
            }
          });
        });
      });
    };
    

更多关于 `$inc` 的知识请参阅《mongodb权威指南》。

增加留言统计就简单多了，直接取 `comments.length` 即可。修改 index.ejs 、user.ejs 及 article.ejs ，在：
    
    <p><%- post.post %></p>
    

下一行添加一行代码：
    
    <p class="info">阅读：<%= post.pv %> | 评论：<%= post.comments.length %></p>