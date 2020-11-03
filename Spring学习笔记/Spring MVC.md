# SpringMVC

## MVC
- MVC：Model View Controller
    - 用户输入给controller，controller把数据和指令发送给model，model与数据库进行交互、进行业务逻辑判断，model根据业务逻辑选择不同的View，View把反馈发送给用户

        ![MVC](img/MVC.jpg)

    - 三层架构：视图层View，服务层Service，持久层DAO
        - 视图层：接收用户提交的请求
        - 服务层：业务逻辑的实现
        - 持久层：直接操作数据库
    - MVC和三层架构之间的关系：MVC中的View和Controller属于视图层，Model包含了服务层和持久层
    - SSM和三层架构之间的关系：SpringMVC属于视图层，MyBatis属于持久层

---

## SpringMVC
- 工作原理
    - SpringMVC有三大组件：处理器映射器(HandlerMapping)、处理器适配器(HandlerAdapter)、视图解析器(ViewResolver)，DispatcherServlet是调度的核心，三大组件都要被DispatcherServlet所调配

        ![工作原理](img/SpringMVC调度流程.png)
    
    - Controller和View是需要自己实现的部分
- 核心组件
    - DispatherServlet：SpringMVC前端控制器，控制中心
    - HandlerMapping：解析请求url，解析出控制器从而映射控制器(也就是说，handlermapping通过解析路径返回了系统所需要种类的控制器)
    - HandlerAdapter：调度Controller处理业务逻辑


---
- ResponseBody
- doDispatch：责任链模式

## 远程方法调用
- 请求参数：（以HTTP协议为例）（客户端发送给服务器端）
    - PATH
    - METHOD
    - HEADER
    - COOKIE
    - PARAMETERS（key-value）
    - BODY（key-value）

