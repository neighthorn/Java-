# MyBatis

## MyBatis
- MyBatis相当于是JDBC的封装，MyBatis框架通过XML或者Annotation使得实体类和SQL语句中建立了一个映射，这样我们就可以直接操作JavaBean对象实现对数据库的操作：
    - 根据JDBC规范建立与数据库的链接
    - 使用反射建立Java对象与数据库之间的映射关系
- MyBatis的核心类：
    - SqlSessionFactory：
        - SqlSessionFactory是创建SqlSession的工厂
        - SqlSessionFactory是单个数据库映射关系经过编译后的内存镜像，SqlSessionFactoryBuilder通过XML配置文件或者预先定制的@Configuration实例构建出SqlSessionFactory的实例，SqlSessionFactory在使用过程中应该是单例模式
        - 线程安全，不需要手动关闭
    - SqlSession：
        - 应用程序和数据库进行交互的API，可以对数据库进行操作
        - 用户可以直接调用SqlSession的对应方法来完成对数据库的对应操作
        - 不是线程安全的，每次使用完需要关闭，应该使用`try-finally`来保证每次的关闭
- MyBatis的流程：
    - 读取Config.xml文件(mybatis全局配置文件)，加载mybatis运行环境等信息，并且读取SQL映射文件的信息
    - 通过配置文件中得到SQL映射文件，加载SQL映射文件(包含了SQL语句)
    - 构造会话工厂SqlSessionFactory
    - 通过SqlSessionFactory创建SqlSession对象，通过SqlSessionFactory对象对数据库进行操作
- Mybatis使用流程：
    - mybatis-config.xml文件：
        ```XML
        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
                PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
        <configuration>
            <environments default="development">
                <environment id="development">
                    <transactionManager type="JDBC"/>
                    <dataSource type="POOLED">
                        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                        <property name="url" value="jdbc:mysql://127.0.0.1:3306/test?serverTimezone=UTC"/>
                        <property name="username" value="root"/>
                        <property name="password" value="xxx"/>
                    </dataSource>
                </environment>
            </environments>
            <mappers>
                <mapper resource="MybatisDemo/UserMapper.xml"/>
            </mappers>
        </configuration>
        ```
    - Mapper文件：UserMapper.xml
        ```XML
        <?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

        <mapper namespace="com.example.MybatisDemo.UserDao">

            <select id="getByName" parameterType="String" resultType="com.example.MybatisDemo.User">
                SELECT * FROM test WHERE name=#{name};
            </select>

            <select id="getAll" resultType="com.example.MybatisDemo.User">
                SELECT * FROM test;
            </select>
        </mapper>
        ```
    - UserDao接口：
        ```Java
        package com.example.MybatisDemo;

        import java.util.List;

        public interface UserDao {
            public User getByName(String name);
            public List<User> getAll();
        }

        ```
    - User类：
        ```Java
        package com.example.MybatisDemo;

        public class User {
            private String name;
            private String sex;

            public String getName() {
                return name;
            }

            public void setName(String name) {
                this.name = name;
            }

            @Override
            public String toString() {
                return "User{" +
                        "name='" + name + '\'' +
                        ", sex='" + sex + '\'' +
                        '}';
            }

            public User(String name, String sex) {
                this.name = name;
                this.sex = sex;
            }

            public String getSex() {
                return sex;
            }

            public void setSex(String sex) {
                this.sex = sex;
            }
        }

        ```
    - 测试类：
        ```Java
        package com.example.MybatisDemo;

        import org.apache.ibatis.io.Resources;
        import org.apache.ibatis.session.SqlSession;
        import org.apache.ibatis.session.SqlSessionFactory;
        import org.apache.ibatis.session.SqlSessionFactoryBuilder;

        import java.io.IOException;
        import java.io.InputStream;
        import java.util.Iterator;
        import java.util.List;

        public class MyBatisTest {
            public static void main(String[] args) throws IOException {
                String resource="mybatis-config.xml";
                InputStream inputStream= Resources.getResourceAsStream(resource);
                SqlSessionFactory sqlSessionFactory=new SqlSessionFactoryBuilder().build(inputStream);

                SqlSession sqlSession=sqlSessionFactory.openSession();

                User user;
                try{
                    String statement="com.example.MybatisDemo.UserDao.getByName";
                    user=sqlSession.selectOne(statement, "xx");
                    System.out.println(user);

                    statement="com.example.MybatisDemo.UserDao.getAll";
                    List<User> list=sqlSession.selectList(statement);
                    Iterator iterator=list.listIterator();
                    while (iterator.hasNext()){
                        user=(User)iterator.next();
                        System.out.println(user);
                    }
                }finally {
                    sqlSession.close();
                }
            }
        }

        ```
    - 注意事项：
        - Mapper.xml映射文件中namespace是面向接口的，所以需要一个接口来定义对应的SQL语句
        - 当从数据库中查询所有记录的时候，也就是上例中的getAll方法，resultType依旧是User而不是一个List
- Mybatis对JDBC的封装：
    - 在上例中，如果在selectOne出打一个断点，我们可以看到执行过程中所有调用的方法：
        - 首先调用了`DefaultSqlSession`类中的`selectOne()`方法，然后`selectOne()`方法进而调用了`selectList()`方法，并且如果List的大小大于1就会抛出异常
        - 然后会调用到`Configuration`类中的`getMappedStatement()`方法，这个方法的参数就是我们的statement语句，处理完statement语句后，会对我们的查询关键字也就是name进行一些格式上的包装判断
        - 然后就会调用到`CachingExecutor`类中的`query`方法，然后会调用一些方法获取绑定的sql命令，进而会通过调用另一个`query`方法进入到`BaseExecutor`类的`query`方法中，调用`queryFromDataBase()`方法，然后我们发现，真正去执行SQL操作的其实是`SimpleExecutor`类，`SimpleExecutor`类中的`doQuery`方法会使用`StatementHandler`，最后会调用到`prepareStatement()`方法也就是JDBC中对数据库进行交互的方法：
            ```Java
            private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
                Connection connection = this.getConnection(statementLog);
                Statement stmt = handler.prepare(connection, this.transaction.getTimeout());
                handler.parameterize(stmt);
                return stmt;
            }
            ```

## Springboot+MyBatis
- Annotation的方式：
    - application.properties：
        ```
        mybatis.type-aliases-package=com.example.Mybatis

        spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=UTF8&serverTimezone=UTC
        spring.datasource.username=root
        spring.datasource.password=xxxx
        spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
        spring.datasource.tomcat.max-wait=10000
        ```
    - 数据库记录对应的对象：
        ```Java
        package com.example.Mybatis;

        public class UserType {
            String name;
            String sex;

            public UserType(String name, String sex) {
                this.name = name;
                this.sex = sex;
            }

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
                return "UserType{" +
                        "name='" + name + '\'' +
                        ", sex='" + sex + '\'' +
                        '}';
            }
        }

        ```
    - Mapper：
        ```Java
        package com.example.Mybatis.mapper;

        import com.example.Mybatis.UserType;
        import org.apache.ibatis.annotations.*;

        import java.util.List;

        public interface MyMapper {
            @Select("select * from test")
            @Results({
                    @Result(property = "name", column = "name"),
                    @Result(property = "sex", column = "sex")
            })
            List<UserType> getAllRecord();

            @Select("select * from test where name=#{name}")
            @Results({
                    @Result(property = "name", column = "name"),
                    @Result(property = "sex", column = "sex")
            })
            UserType getSelectedRecord(String name);

            @Insert("insert into test(name, sex) values(#{name}, #{sex})")
            void insert(UserType user);

            @Update("update test set sex=#{sex} where name=#{name}")
            void update(UserType user);

            @Delete("delete from test where name=#{name}")
            void delete(String name);
        }

        ```
    - 启动类：
        ```Java
        package com.example.Mybatis;

        import org.mybatis.spring.annotation.MapperScan;

        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.SpringBootApplication;

        @SpringBootApplication
        @MapperScan("com.example.Mybatis.mapper")
        // 也可以采用在mapper类加@Mapper注释的方法，但是使用MapperScan不需要在每个Mapper类都写一遍Mapper
        public class MybatisTest {

            public static void main(String[] args) {
                SpringApplication.run(MybatisTest.class, args);
            }
        }

        ```
    - 测试类：
        ```Java
        package com.example.Mybatis;

        import com.example.Mybatis.mapper.MyMapper;
        import org.junit.Test;
        import org.junit.runner.RunWith;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.boot.test.context.SpringBootTest;
        import org.springframework.test.context.junit4.SpringRunner;

        import java.util.Iterator;
        import java.util.List;

        @RunWith(SpringRunner.class)
        @SpringBootTest
        public class MybatisTestTest {
            @Autowired
            private MyMapper myMapper;

            @Test
            public void insert(){
                UserType user=new UserType("xiaohong", "xx");
                myMapper.insert(user);
            }

            @Test
            public void update(){
                UserType user=new UserType("xx", "xx");
                myMapper.update(user);
            }

            @Test
            public void queryAll(){
                List<UserType> list=myMapper.getAllRecord();
                Iterator<UserType> iterator=list.listIterator();
                while(iterator.hasNext()){
                    UserType user=(UserType)iterator.next();
                    System.out.println(user);
                }
            }

            @Test
            public void query(){
                UserType user=myMapper.getSelectedRecord("xx");
                System.out.println(user);
            }

            @Test
            public void delete(){
                myMapper.delete("xx");
            }
        }

        ```
