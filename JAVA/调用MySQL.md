```java
//TIP To <b>Run</b> code, press <shortcut actionId="Run"/> or  
// click the <icon src="AllIcons.Actions.Execute"/> icon in the gutter.  
import java.sql.Connection;  
import java.sql.DriverManager;  
import java.sql.ResultSet;  
import java.sql.Statement;  
  
public class TestConnection {  
    public static void main(String[] args) {  
        try {  
            Class.forName("com.mysql.cj.jdbc.Driver");  
            Connection conn = DriverManager.getConnection(  
                    "jdbc:mysql://localhost:3306/itheima",  
                    "root",  
                    "716111"  
            );  
            System.out.println("连接成功！");  
  
            String sql = "SELECT * FROM shitu;";  
            Statement stmt = conn.createStatement();  
            ResultSet rs =stmt.executeQuery(sql);  
  
            while (rs.next()) {  
                System.out.println(rs.getString("title"));  
            }  
  
            conn.close();  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
}


//显示视图  
            String sql = "SELECT * FROM shitu;";  
            Statement stmt = conn.createStatement();  
            ResultSet rs =stmt.executeQuery(sql);  
  
            while (rs.next()) {  
                System.out.println(rs.getString("title"));  
            }  
//更新数据库  
            String sql1="UPDATE   emp set gender='男' where id=1;";  
            Statement stmt1 = conn.createStatement();  
            stmt1.executeUpdate(sql1);
//如何让两条语句整体成为一个事务，一起成功一起失败  
            try {  
  
                //开启事务  
                conn.setAutoCommit(false);//不自动提交事务  
  
  
                stmt1.executeUpdate(sql1);  
                stmt2.executeUpdate(sql2);  
  
                //执行到这里提交事务  
                conn.commit();  
            } catch (Exception throwables) {  
                //回滚事务  
                conn.rollback();  
                throwables.printStackTrace();  
            }
```

#### Statement
作用：执行SQL语句
`int executeUpdate(sql)`  删除更改操作  返回受影响的行数，执行删除语句返回值可能为0
`ResultSet executeQuery`  查询操作