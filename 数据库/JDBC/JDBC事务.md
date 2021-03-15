### 事务介绍

- **事务：一组逻辑操作单元,使数据从一种状态变换到另一种状态。**

- **事务处理（事务操作）：**保证所有事务都作为一个工作单元来执行，即使出现了故障，都不能改变这种执行方式。当在一个事务中执行多个操作时，要么所有的事务都**被提交(commit)**，那么这些修改就永久地保存下来；要么数据库管理系统将放弃所作的所有修改，整个事务**回滚(rollback)**到最初状态。**原子性**

- 为确保数据库中数据的**一致性**，数据的操纵应当是离散的成组的逻辑单元：当它全部完成时，数据的一致性可以保持，而当这个单元中的一部分操作失败，整个事务应全部视为错误，所有从起始点以后的操作应全部回退到开始状态。 

### JDBC处理事务

- 数据什么时候意味着提交？
  - **当一个连接对象被创建时，默认情况下是自动提交事务**：
  - **如果取消自动提交，但一旦关闭数据库连接，数据就会自动的提交。**
- **JDBC程序中为了让多个 SQL 语句作为一个事务执行：**
  - 调用 Connection 对象的 **setAutoCommit(false);** 以取消自动提交事务
  - 在所有的 SQL 语句都成功执行后，调用 **commit();** 方法提交事务
  - 在出现异常时，调用 **rollback();** 方法回滚事务


### 转账操作示例

```java
public void testJDBCTransaction() {
	Connection conn = null;
	try {
		// 1.获取数据库连接
		conn = JDBCUtils.getConnection();
		// 2.开启事务
		conn.setAutoCommit(false);
		// 3.进行数据库操作
		String sql1 = "update user_table set balance = balance - 100 where user = ?";
		update(conn, sql1, "AA");

		// 模拟网络异常
		//System.out.println(10 / 0);

		String sql2 = "update user_table set balance = balance + 100 where user = ?";
		update(conn, sql2, "BB");
		// 4.若没有异常，则提交事务
		conn.commit();
	} catch (Exception e) {
		e.printStackTrace();
		// 5.若有异常，则回滚事务
		try {
			conn.rollback();
		} catch (SQLException e1) {
			e1.printStackTrace();
		}
    } finally {
        try {
			//6.恢复每次DML操作的自动提交功能
			conn.setAutoCommit(true);
		} catch (SQLException e) {
			e.printStackTrace();
		}
        //7.关闭连接
		JDBCUtils.closeResource(conn, null, null); 
    }  
}
```

### ACID属性

- **原子性（Atomicity）**
  原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。 

- **一致性（Consistency）**
  事务必须使数据库从一个一致性状态变换到另外一个一致性状态。

- **隔离性（Isolation）**
  事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

- **持久性（Durability）**
  持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响。

**不同隔离级别导致的问题：**

- **脏读**: 对于两个事务 T1, T2, T1 读取了已经被 T2 更新但还**没有被提交**的字段。之后, 若 T2 回滚, T1读取的内容就是临时且无效的。
- **不可重复读**: 对于两个事务T1, T2, T1 读取了一个字段, 然后 T2 **更新**了该字段。之后, T1再次读取同一个字段, 值就不同了。
- **幻读**: 对于两个事务T1, T2, T1 从一个表中读取了一个字段, 然后 T2 在该表中**插入**了一些新的行。之后, 如果 T1 再次读取同一个表, 就会多出几行。

### java设置并读取隔离级别

```java
		Connection conn = JDBCUtils.getConnection();
		//获取当前连接的隔离级别
		System.out.println(conn.getTransactionIsolation());
		//设置数据库的隔离级别：
		conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
		//取消自动提交数据
		conn.setAutoCommit(false);//这一步保证在同一事务中
		
		String sql = "select user,password,balance from user_table where user = ?";
		User user = getInstance(conn, User.class, sql, "CC");
		
		System.out.println(user);
```

### 不可重复读测试

```java
	@Test
	public void testTransactionUpdate() throws Exception{
		Connection conn = JDBCUtils.getConnection();
		
		//取消自动提交数据
		conn.setAutoCommit(false);
		String sql = "update user_table set balance = ? where user = ?";
		update(conn, sql, 5000,"CC");
		
		Thread.sleep(15000);
		System.out.println("修改结束");
		JDBCUtils.closeResource(conn, null, null); 
        //关闭数据库链接，也会导致自动提交，以此检验另一个事务中是否发生不可重复读现象
	}
	
	//执行两次查询操作，先后返回数据表中的一条记录，检查两次结果是否相同
	public <T> T getInstance(Connection conn,Class<T> clazz,String sql, Object... args) {
		PreparedStatement ps = null;
		ResultSet rs = null;
		try {
			
			ps = conn.prepareStatement(sql);
			for (int i = 0; i < args.length; i++) {
				ps.setObject(i + 1, args[i]);
			}

			rs = ps.executeQuery();
			// 获取结果集的元数据 :ResultSetMetaData
			ResultSetMetaData rsmd = rs.getMetaData();
			// 通过ResultSetMetaData获取结果集中的列数
			int columnCount = rsmd.getColumnCount();

			if (rs.next()) {
				T t = clazz.newInstance();
				// 处理结果集一行数据中的每一个列
				for (int i = 0; i < columnCount; i++) {
					// 获取列值
					Object columValue = rs.getObject(i + 1);

					// 获取每个列的列名
					// String columnName = rsmd.getColumnName(i + 1);
					String columnLabel = rsmd.getColumnLabel(i + 1);

					// 给t对象指定的columnName属性，赋值为columValue：通过反射
					Field field = clazz.getDeclaredField(columnLabel);
					field.setAccessible(true);
					field.set(t, columValue);
				}
				return t;
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			JDBCUtils.closeResource(null, ps, rs);

		}

		return null;
	}
```

