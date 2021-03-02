---
title: Java基础-JDBC连接MySQL数据库
abbrlink: d6399b35
date: 2020-01-30 12:06:54
categories:
  - Java
  - JDBC
tags: [Java, JDBC, MySQL]
---

> JDBC连接MySQL数据库

<!--more-->



## 简介

使用 `JDBC`(java data base connectivity) 实现连接 `MySQL` 数据库,完成基本的增删改查,事务控制和存储过程的调用.

[代码链接](https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/jdbc/MysqlJDBC)



## 包结构

* `action`  处理客户端请求,调用业务逻辑
* `dao`  数据库持久化类.
* `model`  封装对象模型
* `property`  存放属性文件配置
* `until`  一些用到的工具类



## 子包描述

### `model`包
包含一个`Goddess`类对象,作为数据对象模型,对象属性映射数据库中的表字段.
**注意:** 对象模型中的date类型属性引用的为`java.util.Date`.

### `property`包
配置文件`c3p0.properties`包含了对数据库链接池的配置中的基本信息.

### `until`包
工具包,通过JDBC或者数据库连接池,对外提供`java.sql.Connection`类.

#### **JDBC连接数据库**

这里采用静态代码块的方式,在该类加载时,创建数据库连接对象,减少了服务器重复创建对象的压力.

``` java
/**DB连接的工作获取类*/
public class DBUntil {
	
	private static final String driver = "com.mysql.jdbc.Driver"; 
	private static final String url = "jdbc:mysql://127.0.0.1:3306/javabase?useUnicode=true&amp;characterEncoding=UTF-8";
	private static final String user = "root";
	private static final String password = "123456";
	private static Connection connection = null;
	
	/**使用静态代码块在类初始化的时候加载到内存中,避免重复加载,减少内存开支*/
	static {
		
		try {
			//1.加载驱动程序
			Class.forName(driver);
			//2.获得数据库的连接
			connection = DriverManager.getConnection(url, user, password);
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("数据库连接获取发生异常...");
		}
	}
	
	/**获得数据库连接*/
	public static Connection getMySQLConnection(){
		return connection;
	}
}
```

#### **连接池连接数据库**
使用`C3P0`数据库连接池,读取配置文件信息,并向外返回数据库连接对象.

``` java
/**	使用C3P0连接池,完成数据库的连接.*/
public class C3P0Pool {

	private static ComboPooledDataSource dataSource = null;
	private static Properties properties = null;
	
	static {
		try {
			dataSource = new ComboPooledDataSource();	//初始化数据源
			properties = new Properties();				//初始化属性文件
			InputStream inputStream = new FileInputStream("F:\\JAVA_WorkSpace\\JavaBase\\src\\jdbc\\property\\c3p0.properties");
			properties.load(inputStream);				//以IO形式将属性文件加载
			dataSource.setDriverClass(properties.getProperty("c3p0.driverClass"));
			dataSource.setJdbcUrl(properties.getProperty("c3p0.jdbcUrl"));
			dataSource.setUser(properties.getProperty("c3p0.user"));
			dataSource.setPassword(properties.getProperty("c3p0.password"));
			inputStream.close();						//关闭输入流
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("加载属性文件出现异常........");
		}
	}
	
	public static Connection getC3P0Connection() {
		try {
			Connection connection = dataSource.getConnection();
			return connection;
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("C3P0连接池出现异常....");
		}
		return null;
	}
	
	public static void main(String[] args) {
		System.out.println(properties.getProperty("c3p0.driverClass"));
		System.out.println(properties.getProperty("c3p0.jdbcUrl"));
		System.out.println(properties.getProperty("c3p0.user"));
		System.out.println(properties.getProperty("c3p0.password"));
		System.out.println(getC3P0Connection());
	}
}
```

### `dao`包
数据库持久化层,包含基本的数据表的增删改查,数据库事务的控制以及MySQL存储过程的调用.

#### **基本的增删改查**
- 新增
- 删除
- 修改
- 列表/单个/条件查询

``` java
	/**增*/
	public void addGoddess(Goddess goddess) {
		try {
			Connection connection = DBUntil.getMySQLConnection();
			String sql = " insert into goddess ("
					+ "		user_name,sex,age,birthday,email,mobile,"
					+ "		create_user,create_date,update_user,update_date,isDel)"
					+ " 	value(?,?,?,?,?,?,?,current_date(),?,current_date(),?)";
			PreparedStatement preparedStatement = connection.prepareStatement(sql);
			preparedStatement.setString(1, goddess.getUserName());
			preparedStatement.setString(2, goddess.getSex());
			preparedStatement.setInt(3, goddess.getAge());
			preparedStatement.setDate(4, new Date(goddess.getBirthday().getTime()));
			preparedStatement.setString(5, goddess.getEmail());
			preparedStatement.setString(6, goddess.getMobile());
			preparedStatement.setString(7, goddess.getCreateUser());
			preparedStatement.setString(8, goddess.getUpdateUser());
			preparedStatement.setInt(9, goddess.getIsDel());
			preparedStatement.execute();
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("修改时发生意外.........");
		}
	}
	
	/**删*/
	public void deleteGoddess(Goddess goddess) {
		try{
			Connection connection = DBUntil.getMySQLConnection();
			String sql = " delete from goddess "
					+ " 	where id = ?";
			PreparedStatement preparedStatement = connection.prepareStatement(sql);
			preparedStatement.setInt(1, goddess.getId());
			preparedStatement.execute();
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("删除时发生意外.........");
		}
	}
	
	/**改*/
	public void updateGoddess(Goddess goddess) {
		try{
			Connection connection = DBUntil.getMySQLConnection();
			String sql = " update goddess set "
					+ "		user_name = ?,sex = ?,age = ?,birthday = ?,email = ?,mobile = ?,"
					+ "		update_user = ?,update_date = current_date(),isDel =?"
					+ " 	where id = ?";
			PreparedStatement preparedStatement = connection.prepareStatement(sql);
			preparedStatement.setString(1, goddess.getUserName());
			preparedStatement.setString(2, goddess.getSex());
			preparedStatement.setInt(3, goddess.getAge());
			preparedStatement.setDate(4, new Date(goddess.getBirthday().getTime()));
			preparedStatement.setString(5, goddess.getEmail());
			preparedStatement.setString(6, goddess.getMobile());
			preparedStatement.setString(7, goddess.getUpdateUser());
			preparedStatement.setInt(8, goddess.getIsDel());
			preparedStatement.setInt(9, goddess.getId());
			preparedStatement.execute();
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("修改时发生意外.........");
		}
	}
	
	/**查询列表*/
	public List<Goddess> queryGoddess(){
		try {
			Connection connection = DBUntil.getMySQLConnection();
//			Connection connection = C3P0Pool.getC3P0Connection();
			Statement statement = connection.createStatement();
			String sql = "select user_name,age from goddess";
			ResultSet resultSet = statement.executeQuery(sql);
			List<Goddess> list = new ArrayList<Goddess>();
			Goddess goddess = null;
			while(resultSet.next()){
				goddess = new Goddess();
				goddess.setUserName(resultSet.getString("user_name"));
				goddess.setSex(resultSet.getString("age"));
				list.add(goddess);
			}
			return list;
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("查询时发生意外....");
		}
		return null;
	}
	
	/**查询单个*/
	public Goddess getGoddess(Goddess goddess) {
		try{
//			Connection connection = DBUntil.getMySQLConnection();
			Connection connection = C3P0Pool.getC3P0Connection();
			String sql = " select id, user_name,sex,age,birthday,email,mobile,"
					+ "		create_user,create_date,update_user,update_date,isDel "
					+ " 	from goddess where id = ?";
			PreparedStatement preparedStatement = connection.prepareStatement(sql);
			preparedStatement.setInt(1, goddess.getId());
			ResultSet resultSet = preparedStatement.executeQuery();	//针对查询时使用
			Goddess resultGoddess = null;
			while (resultSet.next()) {
				resultGoddess = new Goddess();
				resultGoddess.setId(resultSet.getInt(1));			//传入序号,告知获得第几个参数
				resultGoddess.setUserName(resultSet.getString(2));
				resultGoddess.setSex(resultSet.getString(3));
				resultGoddess.setAge(resultSet.getInt(4));
				resultGoddess.setBirthday(resultSet.getDate(5));
				resultGoddess.setEmail(resultSet.getString(6));	
				resultGoddess.setMobile(resultSet.getString(7));
				resultGoddess.setCreateUser(resultSet.getString(8));
				resultGoddess.setCreateDate(resultSet.getDate(9));
				resultGoddess.setUpdateUser(resultSet.getString(10));
				resultGoddess.setUpdateDate(resultSet.getDate(11));
			}
			return resultGoddess;
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("查询时发生意外.........");
		}
		return null;
	}
	
	/**条件查询*/
	public List<Goddess> queryGoddessBySome(List<Map<String, Object>> param){
		try {
			Connection connection = DBUntil.getMySQLConnection();
			Statement statement = connection.createStatement();
			StringBuffer sql = new StringBuffer(" select * from goddess where 1=1");
			if (param != null && param.size()>0) {
				for (int i = 0; i < param.size(); i++) {
					Map<String, Object> map = param.get(i);
					//	条件		关系		条件值
					sql.append(" and "+map.get("key")+" "+map.get("nexus")+" "+map.get("value"));
				}
			}
			ResultSet resultSet = statement.executeQuery(sql.toString());
			List<Goddess> list = new ArrayList<Goddess>();
			Goddess goddess = null;
			while(resultSet.next()){
				goddess = new Goddess();
				goddess.setId(resultSet.getInt("id"));			//传入序号,告知获得第几个参数
				goddess.setUserName(resultSet.getString("user_name"));
				goddess.setSex(resultSet.getString("sex"));
				goddess.setAge(resultSet.getInt("age"));
				goddess.setBirthday(resultSet.getDate("birthday"));
				goddess.setEmail(resultSet.getString("email"));	
				goddess.setMobile(resultSet.getString("mobile"));
				goddess.setCreateUser(resultSet.getString("create_user"));
				goddess.setCreateDate(resultSet.getDate("create_date"));
				goddess.setUpdateUser(resultSet.getString("update_user"));
				goddess.setUpdateDate(resultSet.getDate("update_date"));
				goddess.setIsDel(resultSet.getInt("isdel"));
				list.add(goddess);
			}
			return list;
		} catch (Exception e) {
			e.printStackTrace();
			System.err.println("条件查询时发生意外....");
		}
		return null;
	}
```

#### **事务控制**

- 事务的特点
原子性:事务是一个完整的操作
一致性:当事务完成时,数据必须处于一致状态
隔离性:对数据进行修改的所有并发事务是彼此隔离的
永久性:当事务完成后,它对数据库的修改是永久的

- 通过`connection.setAutoCommit(false)`将该类禁止自动提交事务.
- 通过`connection.commit()`和`connection.rollback()`完成事务的提交或者回滚

``` java
/**使用事务,保证操作一致性*/
public void transaction() throws Exception {

	Connection connection = DBUntil.getMySQLConnection();
	//1.禁止自动提交
	connection.setAutoCommit(false);
	try {
		/*业务部分,成立则提交,失败则回滚*/

		connection.commit();
	} catch (Exception e) {
		e.printStackTrace();
		System.err.println("事务操作出现异常.......");
		connection.rollback(); 	//回滚事务
	}
}
```

#### **调用存储过程**

- 通过`CallableStatement`类的`prepareCall(sql)`方法,传入需要的SQL语句,并调用`setString`完成sql参数注入,实现存储过程的调用.

``` java
/**
 * 	调用无参的存储过程
 * 	create  procedure `sp_select_nofilter`()
 *	begin
 *		select * from goddess;
 *	end
 * */
public List<Goddess> procedure_select_nofilter() {
	try {
		//1.获得数据库连接
		Connection connection = DBUntil.getMySQLConnection();
		//2.获得存储过程调用对象
		String sql = " call sp_select_nofilter() ";
		CallableStatement callableStatement = connection.prepareCall(sql);
		//3.执行存储过程,返回是否执行成功的布尔值
		callableStatement.execute();
		//4.处理返回的结果.结果集|出参
		ResultSet resultSet = callableStatement.getResultSet();
		List<Goddess> list = new ArrayList<Goddess>();
		Goddess goddess = null;
		while(resultSet.next()){
			goddess = new Goddess();
			goddess.setId(resultSet.getInt("id"));			//传入序号,告知获得第几个参数
			goddess.setUserName(resultSet.getString("user_name"));
			goddess.setSex(resultSet.getString("sex"));
			goddess.setAge(resultSet.getInt("age"));
			goddess.setBirthday(resultSet.getDate("birthday"));
			goddess.setEmail(resultSet.getString("email"));	
			goddess.setMobile(resultSet.getString("mobile"));
			goddess.setCreateUser(resultSet.getString("create_user"));
			goddess.setCreateDate(resultSet.getDate("create_date"));
			goddess.setUpdateUser(resultSet.getString("update_user"));
			goddess.setUpdateDate(resultSet.getDate("update_date"));
			goddess.setIsDel(resultSet.getInt("isdel"));
			list.add(goddess);
		}
		return list;
	} catch (SQLException e) {
		e.printStackTrace();
		System.err.println("调用存储过程中出现意外....");
	}
	return null;
}


/**
 * 	调用有参的储存过程
 *	create definer=`root`@`localhost` procedure `sp_select_filter`(in p_name varchar(20))
 *	begin
 *		if p_name is null or p_name = '' then
 *			select * from goddess;
 *		else
 *			select * from goddess where user_name like concat('%',p_name,'%');
 *		end if;
 *	end
 * */
public List<Goddess> procedure_select_filter(String userName) {
	try {
		//1.获得数据库连接
		Connection connection = DBUntil.getMySQLConnection();
		//2.获得存储过程调用对象
		String sql = " call sp_select_filter(?) ";
		CallableStatement callableStatement = connection.prepareCall(sql);
		callableStatement.setString(1, userName);
		//3.执行存储过程,返回是否执行成功的布尔值
		callableStatement.execute();
		//4.处理返回的结果.结果集|出参
		ResultSet resultSet = callableStatement.getResultSet();
		List<Goddess> list = new ArrayList<Goddess>();
		Goddess goddess = null;
		while(resultSet.next()){
			goddess = new Goddess();
			goddess.setId(resultSet.getInt("id"));			//传入序号,告知获得第几个参数
			goddess.setUserName(resultSet.getString("user_name"));
			goddess.setSex(resultSet.getString("sex"));
			goddess.setAge(resultSet.getInt("age"));
			goddess.setBirthday(resultSet.getDate("birthday"));
			goddess.setEmail(resultSet.getString("email"));	
			goddess.setMobile(resultSet.getString("mobile"));
			goddess.setCreateUser(resultSet.getString("create_user"));
			goddess.setCreateDate(resultSet.getDate("create_date"));
			goddess.setUpdateUser(resultSet.getString("update_user"));
			goddess.setUpdateDate(resultSet.getDate("update_date"));
			goddess.setIsDel(resultSet.getInt("isdel"));
			list.add(goddess);
		}
		return list;
	} catch (SQLException e) {
		e.printStackTrace();
		System.err.println("调用存储过程中出现意外....");
	}
	return null;
}

/**
 * 	调用出参的存储过程
 * 	
 * 
 * 
 * */
public int procedure_select_count() {
	try {
		//1.获得数据库连接
		Connection connection = DBUntil.getMySQLConnection();
		//2.获得存储过程调用对象
		String sql = " call sp_select_count(?) ";
		CallableStatement callableStatement = connection.prepareCall(sql);
		callableStatement.registerOutParameter(1, Types.INTEGER);
		//3.执行存储过程,返回是否执行成功的布尔值
		callableStatement.execute();
		//4.处理返回的结果.出参
		int count = callableStatement.getInt(1);
		return count;
	} catch (SQLException e) {
		e.printStackTrace();
		System.err.println("调用存储过程中出现意外....");
	}
	return 0;
}
```

### `action`包
模拟客户端请求,调用DAO层完成数据库映射.

#### **基本操作和事务控制**

``` java
private GoddessDao dao = new GoddessDao();

/**查找对象集合*/
@Test
public void queryGoddess() {
	List<Goddess> list = dao.queryGoddess();
	System.out.println(list.toString());
}

/**新增对象*/
@Test
public void	addGoddess(){
	Goddess goddess = new Goddess(2, "小简", "女", 24, new Date(),
				"456@qq.com", "123456789", "张谦", null, "张谦", null, 0);
	dao.addGoddess(goddess);
}

/**更新对象*/
@Test
public void	updateGoddess(){
	Goddess goddess = new Goddess(2, "小梅梅", "女", 23, new Date(),
				"456@qq.com", "987654321", null, null, "张谦", null, 0);
	dao.updateGoddess(goddess);
}

/**删除对象*/
@Test
public void	deleteGoddess(){
	Goddess goddess = new Goddess();
	goddess.setId(2);
	dao.deleteGoddess(goddess);
}

/**查询单个对象*/
@Test
public void	getGoddess(){
	Goddess goddess = new Goddess();
	goddess.setId(1);
	goddess = dao.getGoddess(goddess);
	System.out.println(goddess);
}


/**条件查询对象*/
@Test
public void	queryGoddessBySome(){
	List<Map<String, Object>> params = new ArrayList<>();
	Map<String, Object> param1 = new HashMap<String, Object>();
	param1.put("key", "sex");
	param1.put("nexus", "=");
	param1.put("value", "'女'");
	params.add(param1);
	Map<String, Object> param2 = new HashMap<String, Object>();
	param2.put("key", "user_name");
	param2.put("nexus", "like");
	param2.put("value", "'%小%'");
	params.add(param2);
	List<Goddess> list = dao.queryGoddessBySome(params);
	System.out.println(list.toString());
}

```

#### 调用存储过程

``` java
private ProcedureDao dao = new ProcedureDao();

/**调用无参的存储过程*/
@Test
public void procedure_select_nofilter() {

	List<Goddess> list = dao.procedure_select_nofilter();
	System.out.println(list.toString());
}

/**调用无参的存储过程*/
@Test
public void procedure_select_filter() {
	String userName = "简";
	List<Goddess> list = dao.procedure_select_filter(userName);
	System.out.println(list.toString());
}

/**调用出参的存储过程*/
@Test
public void procedure_select_count() {
	int count = dao.procedure_select_count();
	System.out.println("总记录数为:"+count);
}
```

