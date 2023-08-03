# 1. Spring 基础

## 1.1 核心概念

> 把由自己new对象转为外部提供对象，为了解耦
>
> spring技术中：
>
> * 提供了IOC容器，充当外部
> * 在IOC中创建或管理的对象称为Bean

DI：依赖注入

> 按照本来的关系在容器中建立bean之间的依赖关系

# 1.2 基础练习

1. pom.xml中导入spring坐标

```
 <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
 </dependency>
```

2. 定义spring管理的类与接口
3. resources中创建spring配置文件，使用bean标签配置bean相关信息
   ```
   <bean id="某一不重复id" class="实现类名"/>
   ```
4. 初始化IOC容器，通过容器获取bean
   ```
    ApplicationContext ctx=new ClassPathXmlApplicationContext("appContext.xml");//获取IOC容器
    IoCData ioCData= (IoCData) ctx.getBean("IoCData");//获取Bean
    ioCData.display();//使用Bean调用容器内方法
   ```
5. 删除业务层中new出来的对象（为了解耦）
6. 提供对应的set方法（后续由容器调用）
   ```
   public void setIoCData(IoCData ioCData) 
        this.ioCData = ioCData;
   ```
7. 使用property配置bean中对象的关系
   ```
   <property name="ioCData" ref="IoCData"/>//配置在bean标签中，name表示配置的属性，如private IoCData ioCData中的对象;ref表示参照/连接哪个bean id
   ```