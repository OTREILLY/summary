#### java注解


1. java元注解 </br>
```
@Target    
@Retention  
@Inherited  
@Documented 
@Repeatable  
```

2. @Target </br>
表示这个注解能放在什么位置上，是只能放在类上？还是即可以放在方法上，又可以放在属性上 </br>
```
ElementType.TYPE：能修饰类、接口或枚举类型
ElementType.FIELD：能修饰成员变量
ElementType.METHOD：能修饰方法
ElementType.PARAMETER：能修饰参数
ElementType.CONSTRUCTOR：能修饰构造器
ElementType.LOCAL_VARIABLE：能修饰局部变量
ElementType.ANNOTATION_TYPE：能修饰注解
ElementType.PACKAGE：能修饰包
```


3. @Retention  </br>
表示生命周期  </br>
```
RetentionPolicy.SOURCE： 注解只在源代码中存在，编译成class之后，就没了。@Override 就是这种注解。
RetentionPolicy.CLASS： 注解在java文件编程成.class文件后，依然存在，但是运行起来后就没了。@Retention的默认值，即当没有显式指定@Retention的时候，就会是这种类型。
RetentionPolicy.RUNTIME： 注解在运行起来之后依然存在，程序可以通过反射获取这些信息
```

4. @Inherited   </br>
表示该注解具有继承性  </br>

5. @Documented  </br>
在用javadoc命令生成API文档后，文档里会出现该注解说明  </br>

6. @Repeatable </br>
当没有@Repeatable修饰的时候，注解在同一个位置，只能出现一次重复做两次就会报错了。 </br>
使用@Repeatable之后，再配合一些其他动作，就可以在同一个地方使用多次了。 </br>


