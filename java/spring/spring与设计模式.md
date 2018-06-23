### spring与设计模式

一、spring aop与代理模式 </br>

1）静态代理：  </br>


2）动态代理：  </br>
- 基于接口的动态代理(jdk代理)</br>
java.lang.reflect.Proxy + InvocationHandler </br>
```
public interface ISubject {
  void request();
}
public class RealSubject  implements ISubject{

  @Override
  public void request() {
    System.out.println("RealSubject.request");
  }
}
public class JdkProxySubject implements InvocationHandler {

  ISubject realSubject;

  public JdkProxySubject(ISubject subject){
    this.realSubject = subject;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

    System.out.println("before.... ");
    Object o = null;
    try {
      o = method.invoke(realSubject, args);
    }catch (Exception e){
      System.out.println("em: " + e.getMessage());
      throw e;
    }finally {
      System.out.println("after.... ");
    }
    return o;
  }
}

public class Client {
  public static void main(String[] args) {
    ISubject proxy = (ISubject)Proxy.newProxyInstance(Client.class.getClassLoader(),
      new Class[]{ISubject.class}, new JdkProxySubject(new RealSubject()));
    proxy.request();
  }
}
```
ISubject.request()
ProxySubject implements InvocationHandler{
  ISubject realSubject;
  
  Obeject invoke(){
  
  }
}




- 基于继承的动态代理(cglib) </br>
