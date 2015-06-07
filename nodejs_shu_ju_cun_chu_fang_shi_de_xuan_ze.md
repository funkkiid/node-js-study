# Node.js 数据存储方式的选择

### 如何为你的 Node.js 应用挑选数据库

* * *

Node.js 应用一般有三种方式保存数据。

  1. 不使用任何数据库管理系统（DBMS），把数据保存在内存里或直接使用文件系统。
  2. 使用关系数据库。例如 MySQL, PostgreSQL.
  3. 使用非关系数据库。例如 Redis，MongoDB，CouchDB, PouchDB

#### 无服务器数据存储 (Serverless Data Storage)

从管理上来说，第一种方式是最方便易用的。不需要安装任何数据库，直接使用内存和文件就行了。

无需数据库的内存存储就是使用变量保存程序的计算结果或者状态。简单来说就是程序的变量。这种方式适合存储一些小的使用频率很高的数据，因为它的存取速度非常快。但是一旦程序重启或者服务器重启，这些数据就会丢失。

文件存储就是直接使用服务器的文件系统来保存数据。通常应用程序的配置信息都是使用这种方式。程序或服务器重启都不会影响到这些数据。但是这种方式有并行的问题。比如一个多用户的系统，两个用户同时读写一个文件就会造成一个用户的数据被另外一个覆盖。这个时候就应该使用数据库来存储数据了，数据库可以更好地处理并行的问题。

#### 关系数据库 (RDBMS)

关系数据库适合复杂信息的存储和查询。很多内容管理（CMS），客户关系管理（CRM）和网络商店使用这类数据库。

不过这类数据库需要专门的管理知识。一般都要安装在单独的服务器上，由专人（DBA）来管理。开发人员还需要懂得 SQL 语言。虽然有开发库提供关系对象映射（ORM)）的 API 供开发人员调用，不用直接写 SQL 语句，但是最好还是得懂 SQL。

这里要提一下：不要使用微软的 SQL Server，如果你是在 Linux 上使用 Node.js 开发的话。假如是使用 SQL Server 的认证方式还好说，但是一旦 SQL Server 要求使用 Windows Authentication/domain/NTLM 之类的认证就很麻烦。因为在 Linux 上两个对 SQL Server 支持最好的 Node.js 驱动模块 tedious 和 node-tds 都不支持这类认证，只有使用 odbc 这个模块来调用 FreeTDS，但是这个模块又没多少人用，更新也慢，很多操作都要自己写，所以不适合快速开发。我被迫这样搞过一次，当时不知道是坑，耽误不少时间。

关系数据库里，**MySQL** 是最流行的，Node.js 的支持也很好。另外一个流行的关系数据库是 **PostgreSQL**, 稳定而且标准兼容性好。它和 MySQL 的主要区别就是， PostgreSQL 支持递归查询和特别的数据类型，还支持 LDAP 和 GSSAPI 认证方式。不过 PostgreSQL 对 Windows 支持很差，Windows 上就不要用它了。

还有一个非常受欢迎的轻型关系数据库 SQLite。C 写的，异常小巧（400KB 左右），小到不用单独的进程来运行，可直接运行在程序里。SQLite 是一种前面提到的无服务器（Serverless）存储模式，不依赖外部库，无需配置，所以使用很方便。Ghost 博客系统就是用的它，SQLite 非常适合个人博客的应用场景，基本上是小型数据库的首选。

#### 非关系数据库（NoSQL Database)

其实在最早期一直是非关系数据库的天下，后来关系数据库逐渐流行开来变为主流。不过最近非关系数据库因为简单和可扩展性强又开始受欢迎。风水轮流转啊。

关系数据库牺牲了性能换来了可靠性。许多非关系数据库一般是把性能作为首位，所以非关系数据库可能更适合用于实时分析或即时消息的应用场景。另外，非关系数据库通常不要求事先定义数据的 schema，因此更为灵活，特别适合保存层级 (hierarchy) 变化的数据，所以和 JSON 特别匹配。而且非关系数据库的数据结构更类似程序里经常使用的数据结构，比如哈希表，链表和 键值对（key/value pair）等等，对开发更友好。

下面介绍几个最受欢迎的 NoSQL 数据库。

##### Redis

Redis 适合处理简单而又不需要长期保存的数据，例如即时消息，游戏数据。 Redis 把数据保存在内存（RAM）里，同时也把变化记录到硬盘上，一旦出问题可以从硬盘上重载数据。这样做的缺点就是存储量有限，因为一般内存也就几个G或者几十个G，远不如硬盘动辄上T。不过优点也明显，操作数据超快。

##### MongoDB

MongoDB 是一个通用的非关系数据库。一般的应用场景和之前提到的关系数据库类似，比如 CMS/CRM 和在线商店等。有一个专门的模块 Mongoose 使得操作 MongoDB 简单易上手。 MongoDB 是 MySQL 的一个有力竞争对手。

##### CouchDB

又叫 Apache CouchDB。号称完全拥抱网络的一款非关系数据库。使用 JSON 存储数据，JavaScript 作为查询语言。还有 MapReduce 模式和利用 HTTP 作为 API。CouchDB 性能一般，据说读写速度比 MySQL 还慢，比较适合大数据应用。

##### PouchDB

一款运行在浏览器里的离线数据库。应用程序把数据存储在本地端，所以离线用户用起来就和在线的感觉一样，一旦在线，PouchDB 就可在不同客户端之间同步数据，所以无论走到那里都可读取到最新数据。

#### 小结

希望到此你对各类数据库有个大致了解。另外还有其他很多数据库，比如 Oracle 的企业数据库，性能超好，就是价格高。这里只是列出了常用到的一些开源数据库。

总的来说，一个可以保存比较复杂的结构化数据，并且带有完善查询功能的数据库都会牺牲些读写性能。性能最好的数据库往往都是直接使用服务器内存，但是一旦服务器或者程序重启数据就会丢失。

**如何为你的 Node.js App 挑选数据库：**<br /> 如果只是内容或客户管理，MySQL 和 MongoDB 都是不错的选择，可能 MySQL 会更稳定一点点，Join Table 更容易，但是 MongoDB 的扩展性更好，读写速度更快，也更灵活。这也是关系数据库和非关系数据库之间的区别。<br />

单机或个人应用 SQLite 就足够了。

稍大一些的网站都是混合使用这些数据存储方式。例如网站的配置文件通常利用 JSON 文件保存，内容保存在 MongoDB 里，经常读取和更新的数据利用 Redis 保存。 如果是做在线工具类，可以使用 PouchDB 作为离线数据库，让用户有更好的体验。我见过有人使用 PouchDB 做弹幕页面，很有意思。