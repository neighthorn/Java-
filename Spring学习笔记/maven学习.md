# maven学习笔记

## maven主要功能：
- 提供标准化的项目结构
- 提供标准化的构建流程（编译、测试、打包、发布等）
- 提供依赖管理机智

maven的出现可以使得Java项目的管理更加方便

---

## maven项目结构
举例说明：
```
└─SpringFrameworkIOC(项目名称)
    │  pom.xml(项目描述文件)
    ├─.idea
    ├─src
    │  ├─main
    │  │  ├─java(存放Java源码的目录)
    │  │  │  └─org
    │  │  │      └─example
    │  │  │              App.java
    │  │  │              ImplementAspect.java
    │  │  │              SampleClass.java
    │  │  │
    │  │  └─res(存放资源文件的目录)
    │  │          bean.xml
    │  │
    │  └─test
    │      └─java(存放测试源码的文件)
    │          └─org
    │              └─example
    │                      AppTest.java
    │
    └─target(所有编译打包生成的文件的存放目录)
        ├─classes
        │  │  bean.xml
        │  │
        │  └─org
        │      └─example
        │              App.class
        │              ImplementAspect.class
        │              Logger.class
        │              SampleClass$1.class
        │              SampleClass.class
        │
        ├─generated-sources
        │  └─annotations
        ├─generated-test-sources
        │  └─test-annotations
        └─test-classes
            └─org
                └─example
```
---

## pom.xml
- 标识三元组：\<groupID>、\<artifactID>、\<version>
- 一个Maven工程以上述三个变量作为唯一标识，同时，引用其他第三方库，比如引入依赖的时候，也由上述三个变量确定

## 依赖管理
- 当pom.xml声明一个\<dependency>的时候，Maven会自动下载这个依赖包和这个包所依赖的其他包并且把这个包放入classpath中
- classpath：JVM用到的一个环境变量，是一组目录的集合，用来指示JVM如何搜索.class文件
    - classpath的设定方法有两种：在系统环境变量中设置classpath环境变量；在启动JVM的时候设置classpath变量(给java命令传入-classpath 或 -cp参数)
    - 如果既没有在系统环境变量中设置classpath变量也没有传入-cp参数，那么默认的classpath为`.`，也就是当前目录
- jar包：
    - jar包就是一个zip压缩文件，把package组织的目录层级以及各个目录下所有的文件(包括.class文件和其他文件)都打包成一个jar包，jar包可以作为一个目录添加到classpath中。
    - 需要注意的是，在打包jar包时，要保证jar包里的第一层目录不能是bin而是package目录，比如下面这个例子：
        ```
        package_sample
        └─ bin
        ├─ hong
        │  └─ Person.class
        │  ming
        │  └─ Person.class
        └─ mr
            └─ jun
                └─ Arrays.class

        ```

        如果要将上面的组织目录打包成jar包，就应该是下面这样：
        ```
        Hello.zip
        ├─ hong
        │  └─ Person.class
        │  ming
        │  └─ Person.class
        └─ mr
            └─ jun
                └─ Arrays.class

        ```
        而不是下面这种目录结构：
        ```
        Hello.zip
        └─ bin
        ├─ hong
        │  └─ Person.class
        │  ming
        │  └─ Person.class
        └─ mr
            └─ jun
                └─ Arrays.class

        ```
        原因在于JVM从jar包里面查找.class文件时查找的是`hong/Person.class`而不是`bin/hong/Person.class`
    - /META-INF/MANIFEST.MF文件：MANIFEST.MF文件中可以通过指定Main-Class指定启动类名，还可以通过配置classpath信息包含其他jar包
- 依赖说明的要素：
```
<dependency>
    <groupId>junit</groupId>(必须)
    <artifactId>junit</artifactId>(必须)
    <version>4.11</version>(必须)
    <scope>test</scope>(作用域，可选，默认为compile)
    <type>jar</type>(可选)
    <optional>true</optional>(可选)
</dependency>
```
- 依赖关系作用域(maven的依赖关系通过\<dependency>中的\<scope>变量来声明)
    - compile：编译时需要用到该包(默认)
    - test：编译和运行的时候都不需要用到该包，只有在测试编译和测试运行阶段才需要用到(比如JUnit)
    - runtime：编译时不需要用到，但是运行和测试时需要用到比如mysql(编译时不需要用到JDBC驱动器，只有在运行和测试阶段才需要引入JDBC驱动器)
    - provided：编译时需要用到，但是运行时由JDK或者某个服务器提供
- \<type>：默认为jar，可以创建一个打包方式为pom的项目将某些通用的依赖打包在一起，供其他项目直接引用，引用pom项目时依赖指定类型应该为\<type>pom\</type>
- 下载过的jar包被缓存到用户主目录的.m2文件夹中，所以不是每次都重新下载

---

## Maven的生命周期(lifecycle)
- lifecycle包含多个phase，一个phase指向到多个goal，在mvn命令后面加一个或者多个phase让maven运行到指定的phase，比如`mvn clean package`就代表执行以下几个phase：
    ```
    > preclean
    > clean(前两个phase为clean生命周期的phase，此处的clean是phase而不是生命周期)
    > validate
    ...
    > package(从validate到package都是default生命周期的phase)
    ```
- Maven构建项目就是执行lifecycle执行到指定的phase为止
- maven执行phase的本质就是通过调用插件执行goal来完成的，比如对于compile这个phase，maven本身并不知道如何compile，需要调用插件来执行默认的compile:compile这个goal来完成编译

---

## 模块管理
- 待补

---

## Maven自定义插件
- 待补
