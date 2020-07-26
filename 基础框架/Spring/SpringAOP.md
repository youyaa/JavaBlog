## AOP机制
Spring框架根据bean是否实现接口决定使用何种AOP实现：

如果bean实现了接口则使用JDK动态代理对bean进行增强，如果bean没有实现接口则使用CGLib对bean进行增强。二者均生成了代理对象。

Cglib调用目标对象时，不是直接调用的目标对象，而是通过cglib创建的代理对象调用的目标对象。

