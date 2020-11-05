# Spring JdbcTemplate和事务管理

## Java JDBC
- 在Java中，JDBC是Java访问数据库的一套规范，是一个API，提供的接口包括：
    - JAVA API：提供对JDBC的管理链接
    - JAVA Driver API：支持JDBC管理到驱动器的链接
- 通过使用这些接口，JAVA客户端程序可以使用一套接口访问不同类型的数据库，执行数据库操作

    ![JDBC原理图](img/JDBC原理图.webp)

- 通过JDBC操作数据库的步骤：
    - 注册驱动（仅做一次）
    - 建立连接
    - 创建运行SQL的语句
    - 运行语句
    - 处理运行结果
    - 释放资源
- 实例：
    - 假设当前有一个本地数据库`test`，`test`数据库中有两个表单：`test02_first_tb`和`test`，我们对两个表单进行查询操作和插入操作
    - 注册驱动：
        - `Class.forName("com.mysql.jdbc.Driver")`，这种方式不会对详细的驱动类产生依赖
        - `DriverManager.registerDriver(com.mysql.jdbc.Driver)`，会对详细的类产生依赖
        - `System.setProperty("jdbc.drivers", "driver1:driver2")`，不会对详细的驱动类产生依赖
        - 新版本的`mysql`不需要手动注册驱动，所以不需要写上述三种代码，如果写了可能会出错，低版本的依旧需要
    - 建立连接：
        - 通过`DriverManager.getConnection(url, username, password)`获得一个`Connection`对象获得数据库连接
        - `url`格式：
            - JDBC:子协议:子名称//主机名:port/数据库名？属性名=属性值&…
            - 比如文中所需要的url：`jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC`
        - `username`是登录数据库的用户名
        - `password`是登录数据库的密码
    - 创建运行时对象：
        - 运行对象为`Statement`接口类，可以通过`connection.createStatement()`来获得，同时`Statement`接口类派生出了`PreparedStatement`和`CallableStatement`两个接口类，其中`CallableStatement`又继承了`PreparedStatement`。  
            - `PreparedStatement`中存储的`SQL`语句是被预编译过的，而且预编译之后的`SQL`语句被存储在`PreparedStatement`对象中，多次调用只需要一次编译，提高了效率。可以通过`setString`、`setDouble`等方法来给`SQL`语句中的值赋值。其接口在运行时接受输入参数。
            - `CallableStatement`在想要访问数据库存储过程的时候使用。
                - 数据库的存储过程：一组可编程的函数，为了完成特定的功能的`SQL`语句集，经过编译创建并且保存在数据库中，用户可以通过指定存储过程的名字并且在需要的时候给定参数来调用执行该语句集。（相当于一个接口，需要调用是给参数赋值）
    - 运行`SQL`语句：
        - 对于`Statement`对象，可以通过调用对应操作的方法来执行，比如`executeQuery()`、`executeUpdate()`等方法
        - 对于`PreparedStatement`对象，可以通过调用`execute()`方法运行
    - 处理运行结果：
        - `Query`操作会获得一个`ResultSet`对象，用来保存从数据库查询获得的结果，通过`Key-value`的方式进行保存
    - 释放数据库资源
    - 代码：
        ```Java
        package com.example.JDBC;

        import java.sql.*;

        public class Query {
            public static void main(String[] args) throws SQLException {
                String url="jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC";
                String username="xxx";
                String password="xxx";
                Connection connection= DriverManager.getConnection(url, username, password);

                Statement statement=connection.createStatement();
                String selectString="select * from test02_first_tb";
                ResultSet resultSet=statement.executeQuery(selectString);
                while (resultSet.next()){
                    System.out.println("username: "+resultSet.getString(2)+" id: "+resultSet.getString("id"));
                }

                PreparedStatement preparedStatement=connection.prepareStatement("insert into test (name, sex) values (?,?)");
                preparedStatement.setString(1, "cc");
                preparedStatement.setString(2, "yy");
                Boolean result=preparedStatement.execute();
                System.out.println(result);
            }
        }

        ```
- Java JDBC中的SPI案例：
    - SPI：Service provider interface，Java提供的一套用来被第三方实现或者扩展的接口，它可以用来启用框架扩展和替换组件。SPI的作用就是为这些被扩展的API寻找服务实现。
        - API是实现方制定接口并完成对接口的实现，调用方仅仅依赖接口调用，且无权选择不同实现。从使用人员上来说，API 直接被应用开发人员使用。
        - SPI （Service Provider Interface）是调用方来制定接口规范，提供给外部来实现，调用方在调用时则选择自己需要的外部实现。  从使用人员上来说，SPI 被框架扩展人员使用。
    - 其实看了很多没太理解SPI的原理，结合网上的一个例子，个人认为，SPI是一种类似IOC的思想，都是为了实现解耦，把具体的类的装载交给程序外部实现，在使用一个特定的驱动的实现的时候，我们不需要修改具体的代码，而是通过修改配置文件使得程序运行过程中装载我们想要的具体的驱动。
    - 具体的例子：
        - 接口：
            ```Java
            package com.example.SPI;

            import java.util.List;

            public interface Search {
                public List<String> searchDOC(String keyword);
            }

            ```
        - 具体实现类——文件搜索：
            ```Java
            package com.example.SPI;

            import java.util.List;

            public class FileSearch implements Search{
                @Override
                public List<String> searchDOC(String keyword) {
                    System.out.println("file search: "+keyword);
                    return null;
                }
            }

            ```
        - 具体实现类——数据库搜索：
            ```Java
            package com.example.SPI;

            import java.util.List;

            public class DatabaseSearch implements Search{
                @Override
                public List<String> searchDOC(String keyword) {
                    System.out.println("database search: "+keyword);
                    return null;
                }
            }

            ```
        - 测试类：
            ```Java
            package com.example.SPI;

            import java.util.Iterator;
            import java.util.ServiceLoader;

            public class SPITest {
                public static void main(String[] args) {
                    ServiceLoader<Search> searches=ServiceLoader.load(Search.class);
                    Iterator<Search> iterator=searches.iterator();
                    while(iterator.hasNext()){
                        Search search=iterator.next();
                        search.searchDOC("hello world");
                    }
                }
            }

            ```
        - 配置文件：
            - 我们需要在resource文件夹下新建META-INF/services/文件夹，其中新建文件，文件命名为对应的接口：`com.example.SPI.Search`
            - 如果我们在配置文件中写入一行`com.example.SPI.FileSearch`，那么输出结果为`file search: hello world`，证明只加载FileSearch的实现类
            - 如果我们在配置文件写入两行：
                ```
                com.example.SPI.FileSearch
                com.example.SPI.DatabaseSearch
                ```
                那么输出结果也是两个：
                ```
                file search: hello world
                database search: hello world
                ```
    - JDBC中的`DriverManager`也是同样的原理：
        - 我们发现`DriverManager`有一个静态方法：
            ```Java
            static {
                loadInitialDrivers();
                println("JDBC DriverManager initialized");
            }
            ```
        - `loadInitialDrivers()`的源码中也包含了SPI机制，具体代码如下：
            ```Java
            ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
            Iterator<Driver> driversIterator = loadedDrivers.iterator();
            ```

## JdbcTemplate
- 模板方法模式：
    - 模板方法是类的行为模式。准备一个抽象类，将部分逻辑以具体方法以及具体构造函数的形式实现，然后声明一些抽象方法来迫使子类实现剩余的逻辑。不同的子类可以以不同的方式实现这些抽象方法，从而对剩余的逻辑有不同的实现。具体来说，就是在父类中定义好了骨架，比如某些公共方法的定义、方法的调用顺序等，然后子类通过抽象方法或者钩子方法来实现特定的逻辑。
    - 模板方法模式中方法的分类：
        - 模板方法
        - 基本方法：
            - 抽象方法：子类需要重写的方法
            - 钩子方法：一个空方法，子类继承了默认也是空的，子类可以加以扩展从而控制父类。与抽象方法不同的是，当子类不重写的时候就可以使用父类默认的实现。
            - 具体方法：父类实现，子类不需要重写的方法
    - 代表具体逻辑步骤的方法叫做基本方法，将基本方法汇总起来的方法叫做模板方法，模板方法中定义了基本方法的调用顺序、前后文等，而逻辑的部分具体实现可能推迟到子类去实现。
    - 具体实现：
        - 抽象类：
            ```Java
            package com.example.TemplateMethodPattern;

            public abstract class AbstractTemplate {
                public void templateMethod(){
                    abstractMethod();
                    //子类可以通过重写hookMethod来控制是否执行concreteMethod1
                    if(hookMethod())
                        concreteMethod1();
                    concreteMethod2();
                }

                protected abstract void abstractMethod();

                protected boolean hookMethod() {
                    return true;
                }

                private final void concreteMethod1(){
                    System.out.println("This is concreteMethod1...");
                }

                private final void concreteMethod2(){
                    System.out.println("This is concreteMethod2...");
                }
            }

            ```
        - 实现类：
            ```Java
            package com.example.TemplateMethodPattern;

            public class Concrete1 extends AbstractTemplate{
                @Override
                protected void abstractMethod() {
                    System.out.println("This is Concrete1...");
                }

                @Override
                protected boolean hookMethod(){
                    return false;
                }
            }

            public class Concrete2 extends AbstractTemplate{
                @Override
                protected void abstractMethod() {
                    System.out.println("This is Concrete2...");
                }
            }
            ```
        - 测试类：
            ```Java
            package com.example.TemplateMethodPattern;

            public class TemplateTest {
                public static void main(String[] args) {
                    AbstractTemplate concrete1=new Concrete1();
                    AbstractTemplate concrete2=new Concrete2();
                    concrete1.templateMethod();
                    concrete2.templateMethod();
                }
            }

            ```
        - 运行结果：
            ```
            This is Concrete1...
            This is concreteMethod2...
            This is Concrete2...
            This is concreteMethod1...
            This is concreteMethod2...
            ```
- 回调模式：
    - 回调是一个双向调用的关系，A类事先注册某个函数F到B类，A类在调用B类的P函数的时候，B类反过来调用了A类注册给他的F函数，F就是回调函数
- JdbcTemplate是Spring框架中提供的一个对象，对原始的JDBC API进行简单的封装，是回调模式和模板模式的结合，把变化的逻辑通过传递一个参数的方式传递到JdbcTemplate的方法中，这就用到了回调模式。
- JdbcTemplate使用实例：
    - 配置文件
        ```XML
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

            <!-- Spring内置数据源DriverManagerDataSource -->
            <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
                <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=UTC"></property>
                <property name="username" value="xxx"></property>
                <property name="password" value="xxxx"></property>
            </bean>

            <!-- 创建JdbcTemplate对象 -->
            <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
                <property name="dataSource" ref="dataSource"></property>
            </bean>
        </beans>
        ```
    - Test类(数据库中对应表单)
        ```Java
        package com.example.JdbcTemplate;

        public class Test {
            String name;
            String sex;

            public String getName() {
                return name;
            }

            public void setName(String name) {
                this.name = name;
            }

            public String getSex() {
                return sex;
            }

            public void setSex(String sex) {
                this.sex = sex;
            }

            @Override
            public String toString() {
                return "Test{" +
                        "name='" + name + '\'' +
                        ", sex='" + sex + '\'' +
                        '}';
            }
        }

        ```
    - 测试类：
        ```Java
        package com.example.JdbcTemplate;

        import org.springframework.context.support.ClassPathXmlApplicationContext;
        import org.springframework.jdbc.core.BeanPropertyRowMapper;
        import org.springframework.jdbc.core.JdbcTemplate;

        import java.util.Iterator;
        import java.util.List;

        public class JdbcTemplateTest {
            public static void query(JdbcTemplate jdbcTemplate){
                List<Test> list=jdbcTemplate.query("select * from test", new BeanPropertyRowMapper<Test>(Test.class));
                Iterator<Test> iterator=list.listIterator();
                while (iterator.hasNext()){
                    Test test=iterator.next();
                    System.out.println(test.toString());
                }
            }

            public static void main(String[] args) {
                ClassPathXmlApplicationContext context=new ClassPathXmlApplicationContext("JdbcBean.xml");
                JdbcTemplate jdbcTemplate=(JdbcTemplate) context.getBean("jdbcTemplate");

                // 插入
                jdbcTemplate.update("insert into test(name ,sex) values (?, ?)", "lily", "xx");
                query(jdbcTemplate);

                // 删除
                jdbcTemplate.update("delete from test where name = ?", "lily");
                query(jdbcTemplate);

                //更新
                jdbcTemplate.update("update test set sex = ? where name = ?", "kk", "xx");
                query(jdbcTemplate);
            }
        }

        ```
    - 运行结果
        ```
        10:38:55.648 [main] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@6bc168e5
        10:38:55.818 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 2 bean definitions from class path resource [JdbcBean.xml]
        10:38:55.867 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'dataSource'
        10:38:55.942 [main] DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Loaded JDBC driver: com.mysql.cj.jdbc.Driver
        10:38:55.943 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'jdbcTemplate'
        10:38:55.976 [main] DEBUG org.springframework.jdbc.core.JdbcTemplate - Executing prepared SQL update
        10:38:55.977 [main] DEBUG org.springframework.jdbc.core.JdbcTemplate - Executing prepared SQL statement [insert into test(name ,sex) values (?, ?)]
        10:38:55.980 [main] DEBUG org.springframework.jdbc.datasource.DataSourceUtils - Fetching JDBC Connection from DataSource
        10:38:55.981 [main] DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC]
        10:38:56.278 [main] DEBUG org.springframework.jdbc.core.JdbcTemplate - Executing SQL query [select * from test]
        10:38:56.279 [main] DEBUG org.springframework.jdbc.datasource.DataSourceUtils - Fetching JDBC Connection from DataSource
        10:38:56.279 [main] DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC]
        10:38:56.301 [main] DEBUG org.springframework.jdbc.core.BeanPropertyRowMapper - Mapping column 'name' to property 'name' of type 'java.lang.String'
        10:38:56.302 [main] DEBUG org.springframework.jdbc.core.BeanPropertyRowMapper - Mapping column 'sex' to property 'sex' of type 'java.lang.String'
        Test{name='xx', sex='cc'}
        Test{name='cc', sex='yy'}
        Test{name='lily', sex='xx'}
        10:38:56.304 [main] DEBUG org.springframework.jdbc.core.JdbcTemplate - Executing prepared SQL update
        10:38:56.304 [main] DEBUG org.springframework.jdbc.core.JdbcTemplate - Executing prepared SQL statement [delete from test where name = ?]
        10:38:56.304 [main] DEBUG org.springframework.jdbc.datasource.DataSourceUtils - Fetching JDBC Connection from DataSource
        10:38:56.304 [main] DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC]
        10:38:56.309 [main] DEBUG org.springframework.jdbc.core.JdbcTemplate - Executing SQL query [select * from test]
        10:38:56.309 [main] DEBUG org.springframework.jdbc.datasource.DataSourceUtils - Fetching JDBC Connection from DataSource
        10:38:56.309 [main] DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC]
        10:38:56.312 [main] DEBUG org.springframework.jdbc.core.BeanPropertyRowMapper - Mapping column 'name' to property 'name' of type 'java.lang.String'
        10:38:56.312 [main] DEBUG org.springframework.jdbc.core.BeanPropertyRowMapper - Mapping column 'sex' to property 'sex' of type 'java.lang.String'
        Test{name='xx', sex='cc'}
        Test{name='cc', sex='yy'}
        10:38:56.313 [main] DEBUG org.springframework.jdbc.core.JdbcTemplate - Executing prepared SQL update
        10:38:56.313 [main] DEBUG org.springframework.jdbc.core.JdbcTemplate - Executing prepared SQL statement [update test set sex = ? where name = ?]
        10:38:56.313 [main] DEBUG org.springframework.jdbc.datasource.DataSourceUtils - Fetching JDBC Connection from DataSource
        10:38:56.313 [main] DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC]
        10:38:56.316 [main] DEBUG org.springframework.jdbc.core.JdbcTemplate - Executing SQL query [select * from test]
        10:38:56.317 [main] DEBUG org.springframework.jdbc.datasource.DataSourceUtils - Fetching JDBC Connection from DataSource
        10:38:56.317 [main] DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC]
        10:38:56.319 [main] DEBUG org.springframework.jdbc.core.BeanPropertyRowMapper - Mapping column 'name' to property 'name' of type 'java.lang.String'
        10:38:56.320 [main] DEBUG org.springframework.jdbc.core.BeanPropertyRowMapper - Mapping column 'sex' to property 'sex' of type 'java.lang.String'
        Test{name='xx', sex='kk'}
        Test{name='cc', sex='yy'}
        ```

## 事务管理