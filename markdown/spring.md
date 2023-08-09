# 1. Spring 基础

## 1.1 核心概念

> * 提供了IOC容器，充当外部
> * 在IOC中创建或管理的对象称为Bean

DI：依赖注入

> 按照本来的关系在容器中建立bean之间的依赖关系

AOP：不改原有的基础上增强功能

> 通知类与通知方法：程序中的共性功能抽取出来单列
>
> 连接点：程序执行的任意位置，在SpringAop中为方法的执行
>
> 切入点：匹配连接点的式子，在SpringAop中为通知方法加入的方法
>
> 切面：连接切入点和通知

# 1.2 基础练习

### 1.2.1 IoC基础练习

1. pom.xml中导入spring坐标

```java
 <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
 </dependency>
```

2. 定义spring管理的类与接口
3. resources中创建spring配置文件，使用bean标签配置bean相关信息

   ```java
   <bean id="IoCData" class="data.IoCDataImpl"/> //id为某一不重复id，class为实现类名
   ```
4. 初始化IOC容器，通过容器获取bean

   ```java
    ApplicationContext ctx=new ClassPathXmlApplicationContext("appContext.xml");//获取IOC容器
    IoCData ioCData= (IoCData) ctx.getBean("IoCData");//获取Bean方式1：按名称
    IoCData ioCData= ctx.getBean("IoCData",IoCData.class);//获取Bean方式2 ：按名称并指定类型
    IoCData ioCData= ctx.getBean(IoCData.class);//获取Bean方式3：按类型，但要保证同类型的bean唯一
    ioCData.display();//使用Bean调用容器内方法
   ```
5. 删除业务层中new出来的对象（为了解耦）
6. 提供对应的set方法（后续由容器内部调用）

   ```java
   public void setIoCData(IoCData ioCData) 
        this.ioCData = ioCData;
   ```
7. 使用property配置bean中对象的关系

   ```java
   <property name="ioCData" ref="IoCData"/>//配置在需要注入的bean标签中，name表示配置的属性，如private IoCData ioCData中的对象ioCData;ref表示参照/连接哪个bean id，即name中对象所在的类IoCData
   ```

### 1.2.2 spring整合mybatis基础练习

1. pom中导入坐标，略
2. 分别设计数据库连接池等两个配置类来替代mybatis中的xml配置文件

   ```java
   public class jdbcConfig {
    @Value("com.mysql.jdbc.Driver")
    private String driver;
    @Value("jdbc:mysql:///test?useSSL=false")
    private String url;
    @Value("root")
    private String username;
    @Value("1234")
    private String password;
    @Bean
    public DataSource dataSource(){
        DruidDataSource druidDataSource=new DruidDataSource();
        druidDataSource.setDriverClassName(driver);
        druidDataSource.setUrl(url);
        druidDataSource.setUsername(username);
        druidDataSource.setPassword(password);
        return druidDataSource;
      }
   }
   ```

   ```java
   public class SpringConfig {
    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource){
        SqlSessionFactoryBean sqlSessionFactoryBean=new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean;
    }
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer mapperScannerConfigurer=new MapperScannerConfigurer();
        mapperScannerConfigurer.setBasePackage("Data");
        return mapperScannerConfigurer;
    }
   }
   ```

   ```java
   @Configuration
   @ComponentScan({"Data","Service"})
   @PropertySource("classpath:jdbc.properties")
   @Import({jdbcConfig.class,SpringConfig.class})
   public class Config { //这是总的配置类
   }

   ```
3. 服务层代码也要配置为bean和进行自动装配

   ```java
   @Service
   public class UserServiceImpl implements UserService{//这是一个服务层实现类
    @Autowired
    private Mapper mapper;//这是mybatis的查询接口
    public List<User> selectAge(){ //接口中的查询方法
        return mapper.selectAge();
    }
   }
   ```
4. 最后写的main

   ```java
   public class DemoSpring {
    public static void main(String[] args) {
        ApplicationContext applicationContext=new AnnotationConfigApplicationContext(Config.class);
        UserService userService=applicationContext.getBean(UserService.class);
        List<User>user=userService.selectAge();
        System.out.println(user);
    }
   }
   ```

### 1.2.3 AOP基础练习

1. 导坐标
   ```java
   <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.5</version>
   </dependency>
   ```
2. 定义接口和实现类
   ```java
   @Repository
   public class AopImpl implements Aop{//实现类
    @Override
    public void save(){ 
        System.out.println(System.currentTimeMillis());
        System.out.println("save");
    }
    @Override
    public void update(){
        System.out.println("update");
    }
   }
   ```
3. 定义通知类，切入点并绑定
   ```java
   @Component
   @Aspect  //告诉spring按aop处理
   public class Notice { //通知类
      //切入点注解
    @Pointcut("execution(void aopdata.Aop.update())") 
    private void cutin(){} 

    @Before("cutin()") //通知类型，绑定连接点与通知
    public void commonMethod(){ //通知方法
        System.out.println(System.currentTimeMillis());
    }
   }
   ```
4. 更改配置类
   ```java
   @Configuration
   @ComponentScan({"aopdata","common"})//别忘了扫包
   @EnableAspectJAutoProxy //告诉spring有用注解开发的aop，系统会寻找@Aspect
   public class Config {
   }
   ```

## 1.3 Bean

### 1.3.1 Bean基础配置

> id: 见1.2.3
>
> class: 见1.2.3
>
> name：是某个bean id的别名，作用等同于cpp里的引用，可以定义多个
>
> scope： 定义bean作用范围。singleton（默认）：单例；即创建出来的对象都是同一个   prototype：非单例

### 1.3.2 Bean实例化

1. **调用实现类的无参构造方法**（哪怕是private）

   ```java
   public class IoCDataImpl implements IoCData{ //一个实现类的例子
   //    public IoCDataImpl() { 除非用有参构造方法覆盖，否则写不写都没事
   //    }
       @Override
       public void display() {
           System.out.println("data");
       }
   }
   ```

   ```java
   <bean id="hello" class="IoCDataImpl"/> //class为实现类
   ```
2. 静态工厂实例化

   ```java
   public class Factory{ //一个静态工厂的例子，返回实现类对象
      puublic static IoCData getIoCData(){
         return new IoCDataImpl();
      }
   }

   IoCData iocData= Factory.getIoCData();//main函数中的调用方式

   ```

   ```java
   <bean id="hello" class="Factory" factory-method="getIoCData"/> //class为工厂类名，factory-method为工厂里造对象的方法
   ```
3. 实例工厂实例化

   ```java
   public class Factory{ //一个实例工厂类的例子
      public IoCData getIoCData(){
         return new IoCDataImpl();
      }
   }

   Factory factory=new Factory(); //main函数中的调用方式,先创建实例工厂对象，再通过实例工厂对象来创建对象
   IoCData iocData= factory.getIoCData(); 
   ```

   ```java
   <bean id="实例工厂的bean" class="Factory" /> //先造实例工厂的bean
   <bean id="hello" factory-bean="实例工厂的bean" factory-method="getIoCData"/> //factory-bean为实例工厂的bean
   ```
4. **实例工厂实例化的spring优化版**

   ```java
   public class FactoryBean implements FactoryBean<IoCData>{ //一个改良版实例工厂类的例子,接口名为FactoryBean
      public IoCData getObject() throws Exception{ //代替3中工厂中创建对象的方法
         return new IoCDataImpl();
      }
      public Class<?> getObjectType(){ //获取对象类型的方法
         return IoCData.class;
      }
      public boolean isSingleton(){
         return true;//false则为非单例
      }
   }
   ```

   ```java
   <bean id="hello" class="FactoryBean" /> //class为改良版工厂类，配置比3简单多了
   ```

### 1.3.3 Bean的生命周期

1. bean初始化

   ```java
   public class IoCDataImpl implements IoCData{
   public void init(){ //在实现类中定义一个初始化方法
           System.out.println(1);
   	}
   }
   ```

   ```java
    <bean id="IoCData"  init-method="init"/> //配置时注明初始方法即可
   ```
2. bean销毁

   ```java
    <bean id="IoCData"  destory-method="destory"/> //如上述设置后，不会实现实现类中destory方法，因为虚拟机关闭时没有给bean销毁的机会，手动销毁如下：
   ```

   2.1 暴力法：直接手动关闭容器

   ```java
   ClassPathXmlApplicationContext ctx=new ClassPathXmlApplicationContext("appContext.xml");//其中ClassPathXmlApplicationContext是ApplicationContext接口的实现类
   ctx.close(); //后面的有关ctx的全部执行不了
   ```

   2.2 设置关闭钩子：告诉虚拟机做完所有事情之后关的时候先把容器关了

   ```java
   ClassPathXmlApplicationContext ctx=new ClassPathXmlApplicationContext("appContext.xml");
   ctx.registerShutdownHook();//只是告诉虚拟机有这回事，执行完记得关好吧
   ```
3. **初始化与销毁之spring优化版**：不需要配置bean的属性

   ```java
   public class IoCServiceImpl implements IoCService , InitializingBean, DisposableBean { //一个实现类的例子，调用了InitializingBean, DisposableBean两个接口

    private IoCServiceImpl() {
        System.out.println("nihao");
    }

    private IoCData ioCData;
    public void display(){
        System.out.println("service");
        ioCData.display();
    }

    public void setIoCData(IoCData ioCData) {
        this.ioCData = ioCData;
    }

    @Override
    public void destroy() throws Exception { //重写接口中销毁方法

    }

    @Override
    public void afterPropertiesSet() throws Exception { //重写接口中初始化方法，但是在setIoCData（对象属性注入）之后执行

    }
   }
   ```
4. bean生命周期

   创建对象 》执行构造方法 》执行set（属性设置）》执行初始化方法 》使用bean干活 》执行销毁方法

### 1.3.4 依赖注入

1. setter引用注入：见1.2.6-7
2. setter简单类型诸如：

   ```java
   private int data; //在某一实现类中，不是像上面注入引用对象private IoCData ioCData;，而是注入简单类型
   public void setData(int data) {  //并生成对应的注入方法
        this.data = data;
    }
   ```

   ```java
   <property name="data" value="10"/> //value表示简单类型的值
   ```
3. 构造器注入

   ```java
   private IoCData ioCData;
   public IoCServiceImpl(IoCData ioCData) { //换了个方法形式而已，里面都一样
        this.ioCData = ioCData;
   }
   ```

   ```java
   <constructor-arg name="ioCData" ref="IoCData"/> //与property同样的位置，不过name表示构造器中形参的名称，不是对象的名称。后面的ref还是value同上

   ```
4. **自动装配**

   > 保留setter方法，更改bean配置，删除property等具体配置
   > 不能操作对简单类型
   > 优先级低于1-3
   >

   ```java
   <bean ··· autowire="byType"/> //byType按类型注入：按注入对象的类名去匹配，提供要求保证注入对象的类型存在唯一的bean
   <bean ··· autowire="byName"/> //byName:按名称注入：按set方法名去匹配（如有setData就去匹配data）要求保证存在一个id与注入对象相同的bean
   ```
5. 集合注入

   ```java
   <property name="arrayA">//集合变量的名字
      <array> //集合的类型
         <value>100</value> //可以写很多行这个。单值格式，如array，list等
         <entry key="1" value="100"/> //键值对格式，如map
         <prop key="country">china</prop> //properties格式，值写中间
      </array>
   </property>
   ```
6. 第三方资源配置

   1. pom中导入坐标
   2. 查看管理对象中的构造方法和setter方法是否符合注入要求并选择合适的注入方式
   3. 加载外部properties文件

      1. 开启context命名空间：复制至更改处并将beans改为context（原来的也别删）

      ```java
      <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" //更改1
       xsi:schemaLocation="http://www.springframework.org/schema/beans  
                           http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context  //更改2
                           http://www.springframework.org/schema/context/spring-context.xsd //更改3
      ">
      ```

      2. 使用context空间加载properties文件

      ```java
      <context:property-placeholder location="classpath*:*.properties" system-properties-mode="NEVER"/> 
      //classpath*:表示可以从当前工程及其依赖的jar包中读取(不想从依赖的jar包读取去掉*)  
      //*.properties表示所有的properties文件
      //最后一个属性防止配置的属性（如jdbc.username）与系统的冲突，可不加
      ```

      3. 使用属性占位符${}读取属性

      ```java
      <properties name="username" value="root"> //例：原格式
      jdbc.username=root;  //properties文件中属性
      <properties name="username" value="${jdbc.username}"> //更改后的格式
      ```

## 1.4 注解

### 1.4.1 配置Bean（替代功能1.2.3）

1. 加入注解
   ```java
   @Component("ioCData") //替代bean id。使用1.2.4中方式3获取时因为是按类型获取则不用加括号
   public class IoCDataImpl implements IoCData{ 
      ···· //这是一个实现类
   }
   ```
2. 开启context命名空间,见上
3. 配置标签
   ```java
   <context:component-scan base-package="包路径"/> //自动扫描包路径下的所有类，替代bean class
   ```

> @Component衍生注解：功能一样，场合不同
>
> * @Controller：表现层bean
> * @Service：业务层bean
> * @Repository：数据层bean

### 1.4.2纯注解开发（替代功能1.4.1中2-3）

1. 配置：

   ```java
      @Configuration//定义配置类代替配置文件
      @ComponentScan("包路径") //替代1.4.1中3，定义多个路径用{}格式
      public class SpringConfig{}
   ```
2. 获取IoC容器

   ```java
      //main中代码：
      Application ctx=new AnnotationConfigApplication(SpringConfig.class)//替代1.2.4中的IOC容器获取，括号里的是上面定义的配置类
   ```
3. 管理bean作用范围和生命周期

   ```java
      @Scope("prototype") //替代1.3.1中bean作用范围
      public class IoCDataImpl implements IoCData{ //这是一个实现类
         @PostContruct//替代1.3.3中bean的初始化方法
         public void init(){
            ···
         }
         @PreDistory//替代1.3.3中bean的销毁方法
         public void destory(){
            ···
         }
      }
   ```
4. 依赖注入

   > 引用类型注入
   >

   ```java
      @Service
      public class IoCServiceImpl implements IoCService{ //这是一个实现类
         @Autowired //按类型注入。替代1.3.4.4中的自动装配,且由于其基于反射，故不用提供setter方法
         public IoCData ioCData;

         @Autowired
         @Qulifier("bean id")//当有多个调用相同接口的实现类时，使用按名称注入。名称为实现类前使用注解定义的bean id
         public IoCData ioCData;
      }
   ```

   > 简单类型注入
   >

   ```java
      @Service
      public class IoCServiceImpl implements IoCService{ //这是一个实现类
         @Value(1)
         private int age;
      }
   ```
5. 第三方资源配置：替代1.3.4.6

   ```java
   @Configuration
   @ComponentScan("包路径") 
   @ProperSource("classpath:xxx.properties") //配置文件名,多个的话支持大括号，不支持*
   public class SpringConfig{} //配置类
   ```

   ```java
   @Service
      public class IoCServiceImpl implements IoCService{ 
         @Value(${properties文件中属性})
         private int age;
      }
   ```
6. 管理第三方bean

   ```java
   @Value(properties文件中属性)//简单类型依赖注入
     private int age;
   //第三方bean配置类  
   public class JdbcConfig（Data data）{ //引用类型依赖注入：做为形参
      @Bean
      public Data data(){//定义方法获取要管理的对象
         xxx ds=new xxx();//第三方的获取资源接口
         ds.setAge(age);//第三方setter方法设置属性
         return ds;
      }
   } 
   ```

   ```java
   @Configuration
   @ComponentScan("包路径") 
   @ProperSource("classpath:xxx.properties")
   @Import({JdbcConfig.class})
   public class SpringConfig{//导入spring配置类中
   } 
   ```

## 1.5 AOP

### 1.5.1 AOP工作流程

* spring容器启动
* 读取使用中的切入点（即@Pointcut和在切入方法前使用注解声明的）
* 初始化bean，判定bean对应的类是否匹配到切入点
  1. 匹配失败：创建对象
  2. 匹配成功：创建目标对象的代理对象（但类型是代理类型proxy）
* 1情况下，正常操作。2情况下，运行原始方法与增加的方法

### 1.5.2 AOP切入点表达式

* 描述方式：动作关键字（访问修饰符 返回值 包 . 类或接口 . 方法 异常）

  > 动作关键字：描述切入点行为，如execution：执行
  > 访问修饰符：可省略public
  >

  ```java
   execution(void aopdata.Aop.update()) //描述接口，通常情况下
   execution(void aopdata.AopImpl.update()) //描述实现类
  ```
* 通配符

  * *：任意存在值或模糊匹配
    ```java
    // 1.返回值不限 2.com.tecent包下的任意目录（一个*对应一层） 3.select开头的参数 4.任意参数（但至少有一个）
    execution(public * com.tecent.*.User.select*(*))
    ```
  * ..：任意值
    ```java
    // 1.任意目录下（多少层都行） 2.任意参数（可有可没有）
    execution(public int com..User.selectid(..))
    ```
  * +：匹配子类类型
    ```java
    // 匹配User的子类
    execution(public int com..User+.selectid(..))
    ```

### 1.5.3 AOP通知

1. 通知类型

> 格式：`@通知类型("切入点类")`

* @Before：加在基础方法之前执行
* @After：加在基础方法之后执行
* **@Around：基础方法前后都执行**

  ```java
  @Around("cutin()")
                                 //必须声明对基础方法的调用
   public Object commonMethod(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println(System.currentTimeMillis());
        Object obj=proceedingJoinPoint.proceed(); //调用基础方法
        System.out.println(System.currentTimeMillis());
        return obj；//基础方法返回值需要return才会显示,若基础方法无返回值，则commonMethod返回值为void
    }
  ```
* @AfterReturing：获取返回值

  ```java
  @AfterReturing(value="cutin()",returning="str") //定义str（同名于函数的形参）获取返回值
  public afterreturning(JoinPoint jp,String str){ 
      ···
  }
  ```
* @AfterThrowing：获取异常

  ```java
  @AfterReturing(value="cutin()",throwing="t") //定义t（同名于函数的形参）获取异常值
  public afterthrowing(JoinPoint jp,Throwable t){ 
      ···
  }
  ```

2. 获取参数

> 格式：见上面函数的形参

* JoinPoint：前，后，返回后，抛出异常后
* ProceedingJoinPoint：环绕

```java
Object[] args=proceedingJoinPoint.getArgs();//获取基础方法的参数
Object[] args=proceedingJoinPoint.getArgs(args);//仅限ProceedingJoinPoint，可在传参进来后更改再放回去
Signature sig=proceedingJoinPoint.getSignature();
String class=sig.getDeclaringTypeName();//基础方法所在位置
String name=sig.getName();//获取基础方法 
```
