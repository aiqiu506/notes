通常我们使用 go去操作 mysql 的时候，会基于一个第三方库，如 gorm，而 gorm 在使用的时候，需要依赖于，go-sql-driver/mysql，mysql的驱动（个人理解，是封装了一个 mysql-client ，和与服务端通信的协议等）。

通常理解上的误区：**sql.Open()**是创建了一个与 mysql服务端的连接 。其实并非如此。Open 方法，只是创建了一个数据库的抽象对象（它提供了一些跟数据库交互的函数，同时管理维护一个数据库连接池）。而与 mysql 的连接，将由**连接池**进行管理。

> sql.DB表示是数据库抽象，因此你有几个数据库就需要为每一个数据库创建一个sql.DB对象。因为它维护了一个连接池，因此不需要频繁的创建和销毁。它需要长时间保持，因此最好是设置成一个全局变量以便其他代码可以访问。































参考：https://www.jianshu.com/p/340eb943be2e