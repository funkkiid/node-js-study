# 增加日志功能

现在我们来给博客增加日志，实现访问日志（access.log）和错误日志（error.log）功能。

前面我们介绍过，使用 Express 自带的 logger 中间件实现了终端日志的输出：
    
    app.use(logger('dev'));
    

那我们想把日志保存为日志文件该怎么办呢？很简单，我们只需在以上代码的下一行添加：
    
    app.use(logger({stream: accessLog}));
    

并在 var app = express(); 之前添加如下代码即可：
    
    var fs = require('fs');
    var accessLog = fs.createWriteStream('access.log', {flags: 'a'});
    var errorLog = fs.createWriteStream('error.log', {flags: 'a'});
    

这样，我们每一次访问的请求信息，不仅显示在了命令行中，也都保存在了工程根目录下的 access.log 文件里。但 Express 并没有提供记录错误日志的功能，所以我们需自己写一个简单的中间件，在 app.use(express.static(path.join(__dirname, 'public'))); 下一行添加如下代码 ：
    
    app.use(function (err, req, res, next) {
      var meta = '[' + new Date() + '] ' + req.url + '\n';
      errorLog.write(meta + err.stack + '\n');
      next();
    });
    

这样，当有错误发生时，就将错误信息保存到了根目录下的 error.log 文件夹里。