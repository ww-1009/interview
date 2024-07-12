## Serializable（序列化）接口
> **定义**:把原本在内存中的对象状态 变成可存储或传输的过程称之为序列化。序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。

序列化前的对象和反序列化后得到的对象，内容是一样的(且对象中包含的引用也相同)，但两个对象的地址不同。换句话说，序列化操作可以实现对任何可Serializable对象的”深度复制（deep copy）"

```java
import java.io.Serializable;
public class Person implements Serializable {
	private static final long serialVersionUID = 1L;
	//两个属性及getter、setter方法
	private String name;
	private String age;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getAge() {
		return age;
	}

	public void setAge(String age) {
		this.age = age;
	}
}
```

### 什么情况下需要序列化
* 当你想把的内存中的对象状态保存到一个文件中或者数据库中，以便可以在以后重新创建精确的副本；

* 当你想用套接字在网络上传送对象的时候(从一个应用程序域发送到另一个应用程序域中)；

* 当你想通过RMI传输对象的时候；

### 注意事项
* 序列化时，只对对象的状态进行保存，而不管对象的方法；

* 当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；

* 当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；

* 并非所有的对象都可以序列化。

* 序列化会忽略静态变量，即序列化不保存静态变量的状态。静态成员属于类级别的，不能序列化。添加了static、transient关键字后的变量不能序列化。

### 序列化ID

```java
private static final long serialVersionUID = 1L;
```

序列化 ID在 Eclipse 下提供了两种生成策略，一个设为**固定的 1L**，另一个是**随机生成**一个不重复的 long 类型数据（实际上是使用 JDK 工具生成）。一般如果没有特殊需求，用默认的 1L 就可以，这样可以确保反序列化成功。因为不同的序列化id之间不能进行序列化和反序列化。