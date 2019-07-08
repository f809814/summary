<h4 align="center">Spring原理</h4>

##### 1. Spring的IoC原理

------

> [关于Spring IOC (DI-依赖注入)你需要知道的一切]([https://blog.csdn.net/javazejian/article/details/54561302#spring-ioc-%E7%9A%84%E5%8E%9F%E7%90%86%E6%A6%82%E8%BF%B0](https://blog.csdn.net/javazejian/article/details/54561302#spring-ioc-的原理概述))

##### 2.Spring的AOP原理

---

> [关于 Spring AOP (AspectJ) 你该知晓的一切]([https://blog.csdn.net/javazejian/article/details/56267036#spring-aop%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E6%A6%82%E8%A6%81](https://blog.csdn.net/javazejian/article/details/56267036#spring-aop的实现原理概要))

*Spring AOP的实现原理是基于动态代理技术，而动态代理技术又分为Java JDK动态代理和CGLIB动态代理，前者是基于反射技术的实现，后者是基于继承的机制实现。*

- JDK动态代理

  底层也是基于反射实现的，通过`Proxy.newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)`创建代理对象，需要委托对象的类加载器 ，对应的接口和实现了InvocationHandler 接口的代理类，代理类是实现了InvocationHandler 接口的类，需要代理增强的逻辑功能在invoke方法中实现。缺点是委托类必须要实现接口。

  ```java
  public interface Test {
      void test();
  }
  
  //TestImpl类，Test
  public class TestImpl implements Test{
      @Override
      public void test(){
          System.out.println("执行TestImpl的test()方法...");
      }
  }
  //代理类的实现
  public class JDKProxy implements InvocationHandler{
  
      /**
       * 要被代理的目标对象
       */
      private TestImpl target;
  
      public JDKProxy(TestImpl target){
          this.target=target;
      }
  
      /**
       * 创建代理类
       * @return
       */
      public Test createProxy(){
          return (Test) Proxy.newProxyInstance(target.getClass().getClassLoader(),target.getClass().getInterfaces(),this);
      }
  
      /**
       * 调用被代理类(目标对象)的任意方法都会触发invoke方法
       * @param proxy 代理类
       * @param method 被代理类的方法
       * @param args 被代理类的方法参数
       * @return
       * @throws Throwable
       */
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
          //过滤不需要该业务的方法
          if("test".equals(method.getName())) {
              //前置处理
              doBefore();
              //调用目标对象的方法
              Object result = method.invoke(target, args);
              //后置处理
              doAfter();
              return result;
          }eles if("otherMethod".equals(method.getName())){
              //.....
          }
          //如果不需要增强直接执行原方法
          return method.invoke(target,args);
      }
  }
  ```

- CGLIB动态代理

  CGLIB动态代理不需要被代理对象实现接口，而是基于继承实现的。CGLIB根据字节码生成被代理类的子类:`enhancer.setSuperclass(target.getClass())`。

  ```java
  
  //Test类
  public class TestC{
      public void test(){
          System.out.println("执行TestC的test()方法...");
      }
  }
  //代理类的实现
  public class CGLIBProxy implements MethodInterceptor {
  
     /**
       * 被代理的目标类
       */
      private TestC target;
  
      public CGLibProxy(TestC target) {
          super();
          this.target = target;
      }
  
      /**
       * 创建代理对象
       * @return
       */
      public TestC createProxy(){
          // 使用CGLIB生成代理:
          // 1.声明增强类实例,用于生产代理类
          Enhancer enhancer = new Enhancer();
          // 2.设置被代理类字节码，CGLIB根据字节码生成被代理类的子类
          enhancer.setSuperclass(target.getClass());
          // 3.//设置回调函数，即一个方法拦截
          enhancer.setCallback(this);
          // 4.创建代理:
          return (TestC) enhancer.create();
      }
  
      /**
       * 回调函数
       * @param proxy 代理对象
       * @param method 委托类方法
       * @param args 方法参数
       * @param methodProxy 每个被代理的方法都对应一个MethodProxy对象，
       *                    methodProxy.invokeSuper方法最终调用委托类(目标类)的原始方法
       * @return
       * @throws Throwable
       */
      @Override
      public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
     //过滤不需要该业务的方法
        if("test".equals(method.getName())) {
            //前置处理
            doBefore();
            //调用目标对象的方法(执行A对象即被代理对象的execute方法)
            Object result = methodProxy.invokeSuper(proxy, args);
            //后置处理
            doAfter();
            return result;
        }else if("otherMethod".equals(method.getName())){
            //.....
            return methodProxy.invokeSuper(proxy, args);
        }
        //如果不需要增强直接执行原方法
        return methodProxy.invokeSuper(proxy, args);
  
      }
  }
  ```

  