> Mybatis原理：即通过动态代理生成实现了Mapper接口的代理类，在代理类中转而执行XML文件中的SQL语句。

## JDK动态代理

来看看最关键的部分：

```java
Person person = new PersonImpl(); //需要被代理的继承了接口的真实对象
Person o = (Person) Proxy.newProxyInstance(person.getClass().getClassLoader(),
                person.getClass().getInterfaces(), new Handler(person));
```

自定义的Handler：

```Java
class Handler implements InvocationHandler{
    private Object target;    //被代理的真实对象
    public Handler(Object target) {
        this.target = target;
    }

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before();
        Object invoke = method.invoke(target, args);    //通过反射调用真实对象的方法
        after();
        return invoke;
    }

    private void before(){
        System.out.println("代理类前置方法");
    }
    private void after(){
        System.out.println("代理类后置方法");
    }
}
```

通过**Proxy**类的静态方法**newProxyInstance** 生成了代理对象，代理对象继承了Proxy类，并实现了我们的业务接口，重写了所有的业务接口，转而执行**this.h.invoke(this, m4, new Object[]{paramString})**，而这个h就是我们在创建代理类时候传入的自定义的Handler。



## Mybatis自动映射原理

org.apache.ibatis.binding.MapperProxy.java部分源码。

```Java
public class MapperProxy<T> implements InvocationHandler, Serializable {

  private static final long serialVersionUID = -6424540398559729838L;
  private final SqlSession sqlSession;
  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethod> methodCache;

  public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method,      MapperMethod> methodCache) {
    this.sqlSession = sqlSession;
    this.mapperInterface = mapperInterface;
    this.methodCache = methodCache;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (Object.class.equals(method.getDeclaringClass())) {
      try {
        return method.invoke(this, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    }
    // 投鞭断流，转而执行XML文件中的SQL
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    return mapperMethod.execute(sqlSession, args);
  }
  // ...
```

org.apache.ibatis.binding.MapperProxyFactory.java部分源码。

```Java
public class MapperProxyFactory<T> {

  private final Class<T> mapperInterface;

  @SuppressWarnings("unchecked")
  protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }
```



