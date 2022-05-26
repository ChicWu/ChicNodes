---
title: ThreadLocal的使用
date: 2018-08-20 17:19:43
top: 5
tags:
	Java基础
---
　　这家伙很懒竟然没写摘要
<!-- more -->

# 一、 ThreadLocal(线程局部变量)
## 1.1 ThreadLocal API
　　ThreadLocal类只有三个方法：
```
    void set(T value)：保存值；
    T get()：获取值；
    void remove()：移除值;
```
## 1.2 ThreadLocal的内部是Map
　　ThreadLocal内部其实是个Map来保存数据。虽然在使用ThreadLocal时只给出了值，没有给出键，其实它内部使用了当前线程做为键。

```
    class MyThreadLocal<T> {
        private Map<Thread,T> map = new HashMap<Thread,T>();
        public void set(T value) {
            map.put(Thread.currentThread(), value);
        }

        public void remove() {
            map.remove(Thread.currentThread());
        }

        public T get() {
            return map.get(Thread.currentThread());
        }
    }
```

# 二、 Service事务
##  2.1 DAO中的事务
　　在DAO中处理事务真是“小菜一碟”。
```
    public void xxx() {
        Connection con = null;
        try {
            con = JdbcUtils.getConnection();
            con.setAutoCommitted(false);
            QueryRunner qr = new QueryRunner();
            String sql = …;
            Object[] params = …;
            qr.update(con, sql, params);
            sql = …;
            Object[] params = …;
            qr.update(con, sql, params);
            con.commit();
        } catch(Exception e) {
            try {
                if(con != null) {con.rollback();}
            } catch(Exception e) {}
            } finally {
                try {
                con.close();
                } catch(Exception e) {}
            }
        }
    }
```
## 2.2 Service才是处理事务的地方
　　我们要清楚一件事，DAO中不是处理事务的地方，因为DAO中的每个方法都是对数据库的一次操作，而Service中的方法才是对应一个业务逻辑。也就是说我们需要在Service中的一方法中调用DAO的多个方法，而这些方法应该在一起事务中。
　　怎么才能让DAO的多个方法使用相同的Connection呢？方法不能再自己来获得Connection，而是由外界传递进去。
```
    public void daoMethod1(Connection con, …) {
    }
    public void daoMethod2(Connection con, …) {
    }
```
　　在Service中调用DAO的多个方法时，传递相同的Connection就可以了。
```
	public class XXXService() {
		private XXXDao dao = new XXXDao();
		public void serviceMethod() {
		Connection con = null;
		try {
			con = JdbcUtils.getConnection();
			con.setAutoCommitted(false);
			dao.daoMethod1(con, …);
			dao.doaMethod2(con, …);
			com.commint();
			} catch(Exception e) {
				try {
					con.rollback();
				} catch(Exception e) {}
			} finally {
				try {
					con.close();
				} catch(Exception e) {}
			}
		}
	}
```
　　但是，在Service中不应该出现Connection，它应该只在DAO中出现，因为它是JDBC的东西，JDBC的东西是用来连接数据库的，连接数据库是DAO的事儿！！！但是，事务是Service的事儿，不能放到DAO中！！！

## 2.3 修改JdbcUtils
　　我们把对事务的开启和关闭放到JdbcUtils中，在Service中调用JdbcUtils的方法来完成事务的处理，但在Service中就不会再出现Connection这一“禁忌”了。
DAO中的方法不用再让Service来传递Connection了。DAO会主动从JdbcUtils中获取Connection对象，这样，JdbcUtils成为了DAO和Service的中介！
我们在JdbcUtils中添加beginTransaction()和rollbackTransaction()，以及commitTransaction()方法。这样在Service中的代码如下：
### SERVICE:
```
    public class XXXService() {
       private XXXDao dao = new XXXDao();
       public void serviceMethod() {
           try {
              JdbcUtils.beginTransaction();
              dao.daoMethod1(…);
              dao.daoMethod2(…);
              JdbcUtils.commitTransaction();
            } catch(Exception e) {
               JdbcUtils.rollbackTransaction();
            }
        }
    }
```
### DAO:
```
    public void daoMethod1(…) {
      Connection con = JdbcUtils.getConnection();
    }
    public void daoMethod2(…) {
      Connection con = JdbcUtils.getConnection();
    }
```
　　在Service中调用了JdbcUtils.beginTransaction()方法时，JdbcUtils要做准备好一个已经调用了setAutoCommitted(false)方法的Connection对象，因为在Service中调用JdbcUtils.beginTransaction()之后，马上就会调用DAO的方法，而在DAO方法中会调用JdbcUtils.getConnection()方法。这说明JdbcUtils要在getConnection()方法中返回刚刚准备好的，已经设置了手动提交的Connection对象。
　　在JdbcUtils中创建一个Connection con属性，当它为null时，说明没有事务！当它不为null时，表示开启了事务。
在没有开启事务时，可以调用“开启事务”方法；在开启事务后，可以调用“提交事务”和“回滚事务”方法；
　　getConnection()方法会在con不为null时返回con，再con为null时，从连接池中返回连接。
#### beginTransaction()
　　判断con是否为null，如果不为null，就抛出异常！
　　如果con为null，那么从连接池中获取一个Connection对象，赋值给con！然后设置它为“手动提交”。

#### getConnection()
　　判断con是否为null，如果为null说明没有事务，那么从连接池获取一个连接返回；
　　如果不为null，说明已经开始了事务，那么返回con属性返回。这说明在con不为null时，无论调用多少次getConnection()方法，返回的都是同个Connection对象。

#### commitTransaction()
　　判断con是否为null，如果为null，说明没有开启事务就提交事务，那么抛出异常；
　　如果con不为null，那么调用con的commit()方法来提交事务；
　　调用con.close()方法关闭连接；
　　con = null，这表示事务已经结束！

```
    JdbcUtils.java
    public class JdbcUtils {
        private static DataSource dataSource = new ComboPooledDataSource();
        private static Connection con = null;

        public static DataSource getDataSource() {
            return dataSource;
        }

        public static Connection getConnection() throws SQLException {
            if(con == null) {
                return dataSource.getConnection();
            }
            return con;
        }

        public static void beginTranscation() throws SQLException {
            if(con != null) {
                throw new SQLException("事务已经开启，在没有结束当前事务时，不能再开启事务！");
            }
            con = dataSource.getConnection();
            con.setAutoCommit(false);
        }

        public static void commitTransaction() throws SQLException {
            if(con == null) {
                throw new SQLException("当前没有事务，所以不能提交事务！");
            }
            con.commit();
            con.close();
            con = null;
        }

        public static void rollbackTransaction() throws SQLException {
            if(con == null) {
                throw new SQLException("当前没有事务，所以不能回滚事务！");
            }
            con.rollback();
            con.close();
            con = null;
        }
```
## 2.4 再次修改JdbcUtils
　　现在JdbcUtils有个问题，如果有两个线程！第一个线程调用了beginTransaction()方法，另一个线程再调用beginTransaction()方法时，因为con已经不再为null，所以就会抛出异常了。
　　我们希望JdbcUtils可以多线程环境下被使用！这说明最好的方法是为每个线程提供一个Connection，这样每个线程都可以开启自己的事务了。
　　还记得ThreadLocal类么？
```
    public class JdbcUtils {
        private static DataSource dataSource = new ComboPooledDataSource();
        private static ThreadLocal<Connection> tl = new ThreadLocal<Connection>();

        public static DataSource getDataSource() {
            return dataSource;
        }

        public static Connection getConnection() throws SQLException {
            Connection con = tl.get();
            if(con == null) {
                return dataSource.getConnection();
            }
            return con;
        }

        public static void beginTranscation() throws SQLException {
            Connection con = tl.get();
            if(con != null) {
                throw new SQLException("事务已经开启，在没有结束当前事务时，不能再开启事务！");
            }
            con = dataSource.getConnection();
            con.setAutoCommit(false);
            tl.set(con);
        }

        public static void commitTransaction() throws SQLException {
            Connection con = tl.get();
            if(con == null) {
                throw new SQLException("当前没有事务，所以不能提交事务！");
            }
            con.commit();
            con.close();
            tl.remove();
        }

        public static void rollbackTransaction() throws SQLException {
            Connection con = tl.get();
            if(con == null) {
                throw new SQLException("当前没有事务，所以不能回滚事务！");
            }
            con.rollback();
            con.close();
            tl.remove();
        }
    }
```

## 2.5 转账示例
```
    public class AccountDao {
        public void updateBalance(String name, double balance) throws SQLException {
            String sql = "update account set balance=balance+? where name=?";
            QueryRunner queryRunner = new QueryRunner();
            Connection conn = JdbcUtils.getgetConnection();
            queryRunner.query(sql,conn,name,blance);
        }
    }
```
```
    public class AccountService {
        private AccountDao dao = new AccountDao();
        public void transfer(String from, String to, double balance) {
            try {
                JdbcUtils.beginTranscation();
                dao.updateBalance(from, -balance);
                dao.updateBalance(to, balance);
                JdbcUtils.commitTransaction();
            } catch(Exception e) {
                try {
                    JdbcUtils.rollbackTransaction();
                } catch (SQLException e1) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
```
```
    public class AccountServlte extends HttpServlet
        protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
            AccountService as = new AccountService();
            as.transfer("zs", "ls", 100);
        }
```

