### spring


一、面向接口编程  </br>
二、控制反转(IOC)与依赖注入(DI):  </br>
  控制反转：应用程序本身不负责依赖对象的创建和维护，而是由外部容器负责创建和维护；  </br>
  依赖注入：是IOC的一种实现方式   </br>
  IOC的目的就是：创建对象并且组装对象间的关系   </br>
  
三、IOC容器  </br>
  1) 两个包：  </br>
    - org.springframework.beans  </br>
    - org.springframework.context  </br>
    - BeanFactory 提供配置结构和基础功能，加载和初始化Bean  </br>
    - ApplicationContext保存了Bean对象并在Spring中被广发使用  </br>
      - 本地文件  FileSystemXmlApplicationContext  </br>
      - Classpath  ClassPathXmlApplicationContext  </br>
      - Web应用中依赖servlet或Listener  </br>
    
  2) 容器的加载过程：</br>
    Resource定位（Bean的定义文件定位） </br>
    将Resource定位好的资源载入到BeanDefinition </br>
    将BeanDefiniton注册到容器中 </br>
  
四、Spring Bean </br>
  1) spring注入：启动Spring容器加载bean配置的时候，完成对变量的赋值行为  </br>
  - 设置注入  </br>
  - 构造注入  </br>
  
  2) Bean配置项:  </br>
  - Id (通过id获取时匹配)  </br>
  - Class  </br>
  - Scope  </br>
  - Constructor arguments  </br>
  - Properties  </br>
  - Autowiring mode  </br>
  - lazy-initialization mode  </br>
  - Initialization/destruction method  </br>
  
  3) Bean scope:  </br> 
  - singleton: 单例，一个bean容器中只存在一个实例  </br>
  - prototype: 每次请求创建新的实例，destroy方式不生效   </br>
  - request: 每次http请求创建一个实例且仅在当前request内有效  </br>
  - session: 每次http请求创建一个实例，在当前session内有效  </br>
  - global session: 基于portlet的web中有效  </br>
  
  4) Bean的生命周期：  </br>
  - 定义  </br>
  - 初始化  </br> 
  - 使用  </br>
  - 销毁  </br>
  
  5) bean初始化：  </br>
  - 实现org.springframework.beans.factory.InitializingBean接口，覆盖afterPropertiesSet方法  </br>
  - 配置init-method  </br>
  
  6) bean销毁：  </br>
  - 实现org.springframework.beans.factory.DisposableBean接口，覆盖destory方法  </br>
  - 配置destroy-method  </br> 
  
  7) bean后处理器  </br>
    如果我们想在Spring容器中完成bean实例化、配置以及其他初始化方法前后要添加一些自己逻辑处理。我们需要定义一个或多个BeanPostProcessor接口实现类，然后注册到Spring IoC容器中。 </br>
    BeanPostProcessor </br>
      BeanPostProcessor.postProcessBeforeInitialization   </br>
      BeanPostProcessor.postProcessAfterInitialization  </br>
            
  8) 初始化顺序：   </br>
    无参构造器 ->  </br>
      BeanPostProcessor.postProcessBeforeInitialization ->  </br>
        @PostConstruct ->   </br>
          InitializingBean.afterPropertiesSet ->  </br>
            init-method ->  </br>
              BeanPostProcessor.postProcessAfterInitialization ->  </br>
             //todo
        @PreDestroy ->  </br>
          DisposableBean.destroy ->   </br>
            @destory-method   </br>
                
  9) bean初始化与销毁  </br>
    - InitializingBean/DisposableBean 接口来定制初始化之后/销毁之前的操作方法；  </br>
    - 通过 <bean> 元素的 init-method/destroy-method属性指定初始化之后 /销毁之前调用的操作方法；  </br>
    - 在指定方法上加上@PostConstruct 或@PreDestroy注解来制定该方法是在初始化之后还是销毁之前调用。   </br>
 
   10) Spring实例化bean的三种方法  </br>
     - 通过构造函数 
   ```
    <bean id="exampleBean" class="examples.ExampleBean"/>
  ```
    - 通过静态工厂方法
   ```
    <bean id="exampleBean" class="examples.ExampleBean" factory-method="静态方法"/>
  ```
    - 通过实例工厂方法
   ```
    <bean id="serviceLocator"  class="examples.DefaultServiceLocator">
    <bean id="clientService" factory-bean="serviceLocator" factory-method="createClientServiceInstance"/>
  ```
 
  bean自动装配（Autowiring） </br>
  - no: 缺省情况下，自动配置是通过“ref”属性手动设定  </br>
  - byname: 根据名字查找并自动装配  </br>
  - bytype: 根据类型自动装配，如果存在多个同类型的实例，抛出异常  </br>
  - constructor: 与bytype类似，在构造函数参数的byType方式  </br>
  
  bean Resources 针对资源文件的统一接口  </br>
  - UrlResource:  </br>
  - ClassPathResource:  </br>
  - FileSystemResource:  </br>
  - ServletContextResource:  </br>
  - InputStreamResource:  </br>
  - ByteArrayResource:  </br>
  
  ResourceLoader 资源加载类：   </br>
  ```
  public interface ResourceLoader{  
    ResourceLoader getResourceLoader(String ) 
  }

  ```
  
  Bean注解管理   </br>
  - Classpath 扫描与组件管理  </br>
  - 类的自动检测与注册bean  </br>
  ```
    <context:annotation-config/> (开启注解)  </br>
    <context:component-scan base-package=${package}/>  </br>
  ```
    过滤器进行自定义扫描：  </br>
    ```
      <context:component-scan base-package="lesson3"> 
        <context:include-filter type="regex" expression=".*Service" />  
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service" />
      </context:component-scan>
    ```  

  - @Component (通用注解，适合任何bean)   </br>
  - @Repository @Service @Controller  </br> 
  - @Scope("")  //作用域   </br>
  - @Required 适用于bean属性的setter方法，并表示受影响的bean属性必须在XML配置文件在配置时进行填；否则，容器会抛出一个BeanInitializationException异常 </br>
  - @Autowired 适用于setter方法、构造器、成员变量  </br>
    1)在setter方法上使用@Autowired注解来摆脱XML配置文件中的<property>元素，尝试在方法上执行autowire="byType"的自动连接  </br>
    2)在属性上使用@Autowired注解来摆脱setter方法，当你使用<property>传递自动连线属性的值时，Spring将自动为传递的值或引用分配这些属性  </br>
    3)将@Autowired应用于构造函数，构造函数@Autowired注解表示构造函数在创建bean时应该是自动连线的，即使在XML文件中配置bean时也不使用<constructor-arg>元素  </br>
    4)默认情况下，@Autowired注解意味着依赖是必须的，它类似于@Required注解； </br>
    5)@Autowired(required=false) 针对非必要的属性装配，且每个类只有一个构造器可以被标记为required=false  </br>
    6)@Autowired默认是按照byType进行注入的，但是当byType方式找到了多个符合的bean，Autowired默认先按byType，如果发现找到多个bean，则又按照byName方式比对，如果还有多个，则报出异常。  </br>

  - @Qualifier </br>
  配合@Autowired使用，@Qualifier("id")  找到id的bean进行注入  </br>
    
  - @Resource  </br>
  ```
    * 如果在属性上找到@Resource注解 
         * 如果@Resource的注解name属性为"" 
         *    则把@Resource所在的属性的名称和Spring容器中的id作匹配 
         *      如果匹配成功，则赋值 
         *      如果匹配不成功，则会按照类型进行配置 
         *        如果匹配成功，则赋值；
         *        匹配不成功，报错 
         * 如果@Resource的注解的name的值不为"" 
         *    则解析@Resource注解name属性的值，把值和spring容器中的ID进行呢匹配 
         *       如果匹配成功，则赋值 
         *       如果匹配不成功，则报错
  ```
  
  - @Bean 与xml中的<bean/>一致 </br>
    与@Configuration一起使用  </br>
    ```
    @Configuration
    public static AppConfig{
    
      @Bean(name = "myFoo")
      public Foo foo(){
        return new Foo();
      }
    }
    ```
 
  - 资源加载  </br>
  
五、 Aware结尾的接口  </br>
  - ApplicationContextAware  </br>
  - BeanFactoryAware  </br>
  ...

六、 Spring AOP   </br>
  - AOP基本概念   </br>
    1）编程范式：面向过程编程、面向对象编程、函数式编程、事件驱动编程、面向切面编程   </br>
    2）面向切面编程   </br>
      与面向对象编程互补，解决代码重复性问题和关注点分离  </br>
      关注点分离：  </br>
        水平分离：展示层 ——> 服务层 ——> 持久层  </br>
        垂直分离：模块划分（订单、库存）  </br>
        切面分离：分离出功能性需求和非功能性需求，集中管理关注点  </br>
  ``` 
    Spring AOP
    Schema-based AOP
    Spring AOP API
    Aspectj
  ```
