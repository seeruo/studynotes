# MySql

## 初始化

```java
// 反射注册驱动,高版本可以不写注册，会自动注册。但是必须要加载驱动jar包
Class.forName("com.mysql.jdbc.Driver");
// 获取数据库连接
Connection con = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/dbname?useSSL=false", "user", "***");
// 获取执行语句对象
Statement db = con.createStatement();

// 关闭
db.close();
con.close();
```

> **注意**：数据库连接，如果不是ssl连接，后面需要加上'useSSL=false'标识。

## 语句

### 查询

```java
ResultSet res = db.executeQuery("SELECT sex FROM user");
while (res.next()){
    System.out.println(res.getInt("sex"));
}
res.close();
```

### 预编译

可以防止注入

```java
PreparedStatement pst = con.prepareStatement("SELECT * FROM user WHERE username=? AND sex=?");
pst.setString(1, "admin");
pst.setString(2, "m");
ResultSet res = pst.executeQuery();
while (res.next()) {
    System.out.println(res.getString("id"));
}
```

## DbUtils工具

### `QueryRunner`

```java
QueryRunner qr = new QueryRunner();
List<Map<String, Object>> q = qr.query(CON, "select id,name from user", new MapListHandler());
for (Map<String, Object> o : q) {
    System.out.println(o.get("id"));
}
```

支持类型：

* `ArrayHandler`
* `ArrayListHandler`,`BeanHandler`,`BeanListHandler`,`ColumnListHandler`,`ScalarHandler`,`MapHandler`,`MapListHandler`,`KeyedHandler`

## DBCP连接池

```java
BasicDataSource ds = new BasicDataSource();
ds.setUrl("jdbc:mysql://127.0.0.1/1to?useSSL=false");
ds.setUsername("root");
ds.setPassword("gQ952429d");
Connection con;
try {
    con = ds.getConnection();
} catch (SQLException e) {
    System.out.println(e.toString());
}
```

