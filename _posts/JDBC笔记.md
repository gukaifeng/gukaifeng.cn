
## 1. MySQL补充笔记

```mysql
show databases; # 显示所有数据库
select database(); # 显示当前使用的数据库
show tables; # 显示当前数据库中的所有表
desc 表名; # 显示表的信息
rename table 原表名 to 新表名; # 修改表名
select * from t_user limit [行数偏移量,] 行数; # 行数偏移量从 0 开始，用于分页查询
```

## 2. JDBC笔记

### 2.1. 查询数据库全部内容 代码示例

<!--more-->

```java
public static void selectAll() {
    Connection con = null;
    Statement stmt = null;
    ResultSet rs = null;

    try {
        // 1. 注册数据库驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

        // 2。建立数据库连接
//            String url = "jdbc:mysql://localhost:3306/userInfo"; // 设置 url, 3305 是 mysql 默认端口
        String url = "jdbc:mysql://localhost:3306/userInfo?useUnicode=true&characterEncoding=utf-8"; // 设置 url 以及编码格式 utf-8
        String user = "root";
        String password = "12345678";
        con = DriverManager.getConnection(url, user, password);

        // 3. 对数据库进行增删改查
        stmt = con.createStatement(); // Statement 类用来发起请求
        rs = stmt.executeQuery("select * from t_user"); // 参数为 sql 语句，ResultSet 类用于接收查询结果

        // 遍历输出查询结果
        while(rs.next()) System.out.println(rs.getInt(1) + "\t" + // getInt 是表示返回查询结果为整数，下面同理
                rs.getString(2) + "\t" + // 参数可以是列索引，sql 中列索引从 1 开始
                rs.getString("pwd")); // 参数也可以是标签
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        // 关闭各种。。。
        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 2.2. 登录检验 代码示例 （可能会造成 sql 注入）

```JAVA
public static boolean checkUserInfo(String uname, String pwd) {

    Connection con = null;
    Statement stmt = null;
    ResultSet rs = null;

    try {
        Class.forName("com.mysql.cj.jdbc.Driver");

        String url = "jdbc:mysql://localhost:3306/userInfo";

        con = DriverManager.getConnection(url, "root", "12345678");
        stmt = con.createStatement();
        String sql = "select * from t_user where uname = '" + uname + "' and pwd = '" + pwd + "'";
        rs = stmt.executeQuery(sql);

        return rs.next();

    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    return false;
}
```

### 2.3. 登录检验 代码示例 （使用 preparestatement 解决 SQL 注入问题）

```java
public static boolean checkUserInfo2(String  name, String pwd) {
    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;

    try {
        String url = "jdbc:mysql://localhost:3306/userInfo";
        con = DriverManager.getConnection(url, "root", "12345678");
        String sql = "select * from t_user where uname = ? and pwd = ?"; // 参数位置用 ? 代替
        pstmt = con.prepareStatement(sql); // 设置 sql 语句
        pstmt.setString(1, name); // 设置 ? 位置的参数，索引从 1 开始
        pstmt.setString(2, pwd);
        rs = pstmt.executeQuery(); // 开始查询
        return rs.next();

    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(pstmt != null) {
            try {
                pstmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        ;
        if(con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    return false;
}
```

对 PreparedStatement 防止 sql 注入的简单理解

假设 Java 中的查询语句为：

```java
String sql = "select * from t_user where uname = ? and pwd = ?";
```

添加参数

```javascript
pstmt.setString(1, name); // 设置 ? 位置的参数，索引从 1 开始
pstmt.setString(2, pwd);
```

* 若为正常的参数填充 `name = "gukaifeng"`, `pwd = "123456"`

    输出最终的查询语句 `System.out.println(pstmt);`结果为：

    ```
    com.mysql.cj.jdbc.ClientPreparedStatement: select * from t_user where uname = 'gukaifeng' and pwd = '123456'
    ```

    可以看到，原参数是没有单引号的，单引号由 PreparedStatement 自动添加。 

* 若有 sql 注入的参数填充 `nam e= "gukaifeng"`，`pwd = "123456' or '1' = '1"`

    输出最终的查询语句 `System.out.println(pstmt);`结果为：

    ```
    com.mysql.cj.jdbc.ClientPreparedStatement: select * from t_user where uname = 'gukaifeng' and pwd = '123456'' or ''1'' = ''1'
    ```

    可以看到，除了由 PreparedStatement 在原字符串两边自动添加的单引号外，字符串本身的单引号都被换成了两个。

暂时没有弄懂为什么可以防止注入，我试验的结果是加了每个单引号都变为两个。

另外的试验发现，如果验证信息中本来就有单引号，虽然变成了两个，依然可以正确的查询。

网上资料有人的试验结果是给单引号加了转义，与我的结果不同。

但是究竟 PreparedStatement 如何防止注入，又可以用本来就含有单引号的查询参数正常查询，还是没能够理解，这里先记录一个<font color="red">尾巴</font>。

### 2.4. JDBCUtils

就是把重复的代码封装一下，下面是封装后重写上面的 checkUserInfo2。

```java
// JDBCUtils
package com.jdbc.demo;

import java.sql.*;

public class JDBCUtils {

    private static final String SQLURL = "jdbc:mysql://localhost:3306/userInfo";
    private static final String USER = "root";
    private static final String PASSWORD = "12345678";

    public static Connection getConnection() {

        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            return DriverManager.getConnection(SQLURL, USER, PASSWORD);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
    public static PreparedStatement getPreparedStatement(Connection con, String sql, String... p) {
        if(con != null)
            try {
                PreparedStatement pstmt = con.prepareStatement(sql);
                for(int i = 0; i < p.length; ++i)
                    pstmt.setString(i + 1, p[i]);
                return pstmt;
            } catch (SQLException e) {
                e.printStackTrace();
            }
        return null;
    }
    public static void Close(Connection con, PreparedStatement pstmt, ResultSet rs) {
        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(pstmt != null) {
            try {
                pstmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

}

```

```java
// checkUserInfo2
public static boolean checkUserInfo2(String name, String pwd) {
    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;

    try {
        con = JDBCUtils.getConnection();
        String sql = "select * from t_user where uname = ? and pwd = ?"; // 参数位置用 ? 代替
        pstmt = JDBCUtils.getPreparedStatement(con, sql, name, pwd); // 开始查询
        rs = pstmt.executeQuery();
        return rs.next();
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        JDBCUtils.Close(con, pstmt, rs);
    }
    return false;
}
```

#### 2.4.1. 插入操作

```java
public static void insert(String name, String pwd) {
    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    try {
        con = JDBCUtils.getConnection();
        String sql = "insert into t_user(uname, pwd) values (?, ?)";
        pstmt = JDBCUtils.getPreparedStatement(con, sql, name, pwd);
        if (pstmt != null) {
            int affected = pstmt.executeUpdate(); // 返回受影响的行数
            System.out.println(affected);
        }
    } catch(Exception e) {
        e.printStackTrace();
    } finally {
        JDBCUtils.Close(con, pstmt, rs);
    }
}
```

#### 2.4.2. 删除操作

```java
// 根据 uid 删除某条记录
public static void delete(int uid) {
    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    try {
        con = JDBCUtils.getConnection();
        String sql = "delete from t_user where uid = ?";
        pstmt = con.prepareStatement(sql);
        pstmt.setInt(1, uid);
        int affected = pstmt.executeUpdate();
    } catch(Exception e) {
        e.printStackTrace();
    } finally {
        JDBCUtils.Close(con, pstmt, rs);
    }
}
```



#### 2.4.3. 修改操作

```java
// 根据 uid 修改用户密码
public static void update(int uid, String newPassword) {
    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    try {
        con = JDBCUtils.getConnection();
        String sql = "update t_user set pwd = ? where uid = ?";
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, newPassword);
        pstmt.setInt(2, uid);
        int affected = pstmt.executeUpdate();
    } catch(Exception e) {
        e.printStackTrace();
    } finally {
        JDBCUtils.Close(con, pstmt, rs);
    }
}
```

### 2.5. 事务

```java
con.setAutoCommit(false); // 关闭自动提交（开启事务）

// 这里写需要原子执行的语句

con.commit(); // 手动提交事务
```

### 2.6. 使用 C3P0 连接池

```java
// C3P0DataSources
package com.jdbc;

import com.mchange.v2.c3p0.ComboPooledDataSource;

import java.beans.PropertyVetoException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class C3P0DataSources {
    private static final String URL = "jdbc:mysql://localhost:3306/userInfo";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "12345678";
    private static ComboPooledDataSource ds;

    static {
        try {
            ds = new ComboPooledDataSource(); // new 一个连接池对象

            ds.setDriverClass("com.mysql.cj.jdbc.Driver"); // 设置 jdbc 驱动
            ds.setJdbcUrl(URL); // 设置数据库 url
            ds.setUser(USERNAME); // 设置数据库 username
            ds.setPassword(PASSWORD); // 设置数据库 password

            ds.setInitialPoolSize(5); // 设定初始连接数目
            ds.setMaxPoolSize(20); // 设置最大连接数目
        } catch (PropertyVetoException e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() {
        try {
            return ds.getConnection();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
    public static void ConnectionClose(Connection con) {
        if(con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    public static void PreparedStatementClose(PreparedStatement pstmt) {
        if(pstmt != null) {
            try {
                pstmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    public static void ResultSetClose(ResultSet rs) {
        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```JAVA
// SQLDemo
public static void selectAll() {
    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;

    try {
        con = C3P0DataSources.getConnection();
        if(con != null) {
            String sql = "select * from t_user";
            pstmt = con.prepareStatement(sql);
            rs = pstmt.executeQuery();
            while (rs.next()) {
                System.out.println(rs.getInt("uid") + "\t" +
                        rs.getString("uname") + "\t" +
                        rs.getString("pwd") + "\t" +
                        rs.getInt("balance"));
            }
        }
    } catch(Exception e) {
        e.printStackTrace();
    } finally {
        C3P0DataSources.ConnectionClose(con);
        C3P0DataSources.PreparedStatementClose(pstmt);
        C3P0DataSources.ResultSetClose(rs);
    }
}
```

