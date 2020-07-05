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

一段典型的mybatis核心配置文件：

```xml
<configuration>
    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>
    
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" 		      value="jdbc:mysql://127.0.0.1:3306/testuseUnicode=true&characterEncoding=utf-8"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mapper/userMapper.xml"/>
    </mappers>
</configuration>
```

一段典型的mybatis的代码：

```java
				//1. 加载配置文件
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        //2. 通过SqlSessionFactoryBuilder构建SqlSessionFactory
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //3. 通过sqlSessionFactory获取SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession(true);
        //4. 获取Mapper接口的代理对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        //5. 通过代理对象调用方法
        System.out.println(mapper.queryAll());
```

### 源码分析

1. 从**sqlSession.getMapper(UserMapper.class)**说起，实际返回的是一个代理对象：

```java
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      //生成代理对象
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
  } 
```

2. **mapperProxyFactory.newInstance(sqlSession)**中的方法，通过JDK动态代理生成了代理对象。

```java
public class MapperProxyFactory<T> {

  private final Class<T> mapperInterface;

  @SuppressWarnings("unchecked")
  protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }
```

3. MapperProxy是自定义的Handler，其会在invoke()方法中执行映射文件中的sql语句。


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

