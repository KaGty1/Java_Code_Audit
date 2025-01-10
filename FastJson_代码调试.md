### 环境信息：
#### Fastjson - version : 1.2.24
#### Java - version : 1.8.0_65

User类：

```
public class User {
    private int id;
    private int age;
    private String name;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", age=" + age +
                ", name='" + name + '\'' +
                '}';
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
        System.out.println("setAge : " + age);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
        System.out.println("setName : " + name);
    }
}
```

JSONUser类：

```
import com.alibaba.fastjson.JSONObject;

public class JSONUser {
    public static void main(String[] args) {
        String s = "{\"@type\": \"com.xxxx.fastjson_demo.model.User\", \"age\":18, \"name\":\"admin\"}";
        JSONObject jsonObject = JSONObject.parseObject(s);
        System.out.println(jsonObject);
    }
}
```

1、在 parseObject()方法处打断点：

<img width="886" alt="image" src="https://github.com/user-attachments/assets/e4b654ae-84b2-4492-8853-ec9f1380c372" />

2、跟进 parseObject()方法：

<img width="1040" alt="image" src="https://github.com/user-attachments/assets/00697679-34c7-4b24-a2b6-24aca49ee102" />

obj是解析后的结果，然后返回。

3、继续跟进 parse()方法：

<img width="1023" alt="image" src="https://github.com/user-attachments/assets/d3b26d16-1a4e-43c0-8474-01293c473d8b" />

4、继续跟进 这个 parse()方法中的 parse()方法：

<img width="1012" alt="image" src="https://github.com/user-attachments/assets/6278ecd3-dc73-4f6e-958f-0194190d0ae0" />

5、使用 parser.parse()方法进行解析，并将解析结果赋值给 value之后返回，跟进 parser.parse()方法： (注：核心逻辑就在 parser.parse()方法中)

<img width="1015" alt="image" src="https://github.com/user-attachments/assets/e15718f9-eb58-43e9-b0dd-63c9379d5c07" />

6、parse()方法使用 switch来判断用户输入字符串的类型，case12为左大括号 '{'，由此判断输入为 JSON格式字符串，故创建 JSONObject对象进行解析，跟进 parseObejct()方法：

<img width="955" alt="image" src="https://github.com/user-attachments/assets/78508a86-886e-4ef2-ad5b-7f51fb6225db" />

parseObject() 方法用于解析 Key和 Value，首先解析Key -> 两个双引号之间，Key 为 @type

<img width="1042" alt="image" src="https://github.com/user-attachments/assets/3d2e3b9f-8c9e-4a3f-9dca-140d7d5c474f" />

7、获取到 Key值后，会做一个判断，判断 Key是否等于 JSON.DEFAULT_TYPE_KEY (即 @type)，结果是相等，则使用 loadClass()方法去加载这个类，并将其加入缓存中并赋值给 clazz

<img width="1020" alt="image" src="https://github.com/user-attachments/assets/f640fb07-0f12-4879-a3cf-ce3210741b9d" />

8、接着要对 clazz类 即 @type的值进行反序列化，首先要先获取 反序列化器，跟进 this.config.getDeserializer()方法：

<img width="1027" alt="image" src="https://github.com/user-attachments/assets/5e902bda-ab90-41d2-b25d-2d623677ea81" />

首先先从缓存中去查找获取 User类 (即 @type, clazz)，这个缓存表为提前设置好的，如下所示：

<img width="798" alt="image" src="https://github.com/user-attachments/assets/f46ba3c9-e276-4e42-9b41-e64e730e29b5" />

但是 User类为我们自己定义的类，所以找不到，继续跟进代码

9、跟进 this.getDeserializer()方法：

<img width="1030" alt="image" src="https://github.com/user-attachments/assets/638ad0f6-1ea0-4a43-aa41-a61f1fcc25d7" />

10、跟进代码，来到 this.createJavaBeanDeserializer()方法，跟进：

<img width="1050" alt="image" src="https://github.com/user-attachments/assets/c36c67f9-5178-4901-b192-d2637c8faeb5" />

该方法中首先创建了一个布尔变量 asEnable，asEnabke通常代表着一个动态创建、加载类的这样一个变量，这里默认值为 true：

<img width="1029" alt="image" src="https://github.com/user-attachments/assets/1e2b5a01-29be-4fd1-aef4-5a4bf719e954" />

并且代码中对 clazz，即我们传入的 @type的值的类进行了诸多判断，比如必须是 public类，不可以是 interface接口等等

11、继续步进，跟进 build()方法：

<img width="1048" alt="image" src="https://github.com/user-attachments/assets/b49b308c-b63e-403f-9600-07022c985ea7" />

build()方法的意义：在创建类反序列化器的时候，需要去了解这个类的详细信息，比如类构造方法，setter，getter方法，field等信息，并最后组成一个 JavaBeanInfo

首先获取类中的成员变量、方法，以及类的默认构造方法(即无参构造)：

<img width="1045" alt="image" src="https://github.com/user-attachments/assets/f4d46c7c-fcea-42bf-b734-e5227a5ccd59" />

<img width="642" alt="image" src="https://github.com/user-attachments/assets/e166ec56-cfa6-43b3-8c0e-9fd93c51b40d" />

继续跟进代码，来到3个 for循环处，第一个 for循环为找所有的 setter方法，第二个 for循环为找所有的 public变量(非 static和 final)，第三个 for循环为找所有的 getter方法：

<img width="953" alt="image" src="https://github.com/user-attachments/assets/d046295f-6f73-46e4-98ad-40917f721074" />

首先进入第一个 for循环，对 setter方法做了很多限制：

(1) setter方法名 >= 4

(2) 方法返回类型为 void或者 当前类

(3) 方法的参数个数为 1

(4) 方法名必须以 set开头，且第四位为大写字母：

<img width="967" alt="image" src="https://github.com/user-attachments/assets/f58be190-020b-4ced-bcbb-b3941a3bedeb" />

<img width="618" alt="image" src="https://github.com/user-attachments/assets/fff35baa-d1a7-480b-9e59-0e829da5ef60" />

再进入第二个 for循环，为找所有的 public变量(非 static和 final)，由于这里我写的 User类中的字段访问属性都是 private，所以跳过

再进入第三个 for循环，找所有的 getter方法：

和第一个 for循环类似，对 getter方法做了很多的限制：

(1) getter方法名 >= 4

(2) 不是静态方法，继承自 Collection|Map|AtomicBoolean|AtomicInteger|AtomicLong 类

(3) 方法不可以有参数传入

(4) 方法的第四个字母为大写字母

(5) 没有 setter，只有 getter的方法

 <img width="1197" alt="image" src="https://github.com/user-attachments/assets/09725356-fb64-48a8-a609-1eae10ee3ace" />

12、跳出最后一个 for循环后，步进 -> 回到 this.createJavaBeanDeserializer()方法中：

<img width="1155" alt="image" src="https://github.com/user-attachments/assets/1c802dab-a450-477d-a137-67c0e6a6d2b6" />

步进，获取到了 beanInfo的值

<img width="1522" alt="image" src="https://github.com/user-attachments/assets/12472a2c-0389-49ea-b87c-e4f8fbfadc64" />

接着对 beanInfo中的信息进了诸多判断，若判断符合则会将 asmEnable赋值为 false，若 asmEnable为 false，则将返回一个标准的 JavaBean类型的反序列化器：

<img width="650" alt="image" src="https://github.com/user-attachments/assets/6be6de37-887d-4d01-8fb4-152df9e7d8d1" />

这里我们的 asmEnable为 true，则会调用 this.asmFactory.createJavaBeanDeserializer()方法临时创建一个类作为反序列化器：

<img width="787" alt="image" src="https://github.com/user-attachments/assets/9955a9be-6a1c-4fc8-8d64-cac2f4888dc5" />

13、步进，回到getDeserializer()方法中，获取到了反序列化器如下图所示：

<img width="1081" alt="image" src="https://github.com/user-attachments/assets/4a037a1e-91fe-4b20-bd4d-0e27e18479ea" />

但是这个临时创建的类我们无法进行调试，所以还是想使用默认的 JavaBean反序列化器进行调试，具体看里面的细节

14、那我们如何使用默认的 JavaBean类型的反序列化器呢，使 asmEnale的值为 false即可

如图代码所示，当 getOnly = true时，asmEnable 会被赋值为 false：

<img width="395" alt="image" src="https://github.com/user-attachments/assets/6d649302-acab-4963-88c2-801bcfe51956" />

要使 getOnly = true，传入类中 method的参数个数 != 1即可，但是 setter方法通过检测的前提就是参数个数 = 1，那么只能靠 getter方法了，手动添加一个符合条件的 getter方法：

<img width="573" alt="image" src="https://github.com/user-attachments/assets/fff2519b-864b-4079-a6a1-9abfab293f4f" />

getMap()方法成功通过检测：

<img width="1081" alt="image" src="https://github.com/user-attachments/assets/01ffbc32-4dbc-4b3e-85c5-56e5f28589e6" />

跟进第三个 for循环的 FieldInfo构造函数：

<img width="1245" alt="image" src="https://github.com/user-attachments/assets/7528ac62-5b08-45cc-b2a0-f3fbf1816038" />

由于 getMap()方法的参数个数为0，所以 getOnly成功被赋值为 true：

<img width="1252" alt="image" src="https://github.com/user-attachments/assets/10133dd5-e3b9-40a2-8495-c710c0e32c17" />

getOnly = true -> asEnable = false -> 可以调用默认 JavaBeanDeserializer()方法。

15、可以调试之后，跟进 deserialze()方法，逐步步过代码，其中使用反射(method.invoke)的方法调用 setter方法并对参数进行赋值：

<img width="1230" alt="image" src="https://github.com/user-attachments/assets/a7908501-df58-406f-bf1c-8e3f7af651e6" />

16、最终调用结果如图所示，parse()方法调用 setter，toJSON()方法调用 getter：

<img width="1272" alt="image" src="https://github.com/user-attachments/assets/bb8ea129-1b2f-4027-a279-b98585cbacec" />

由此，分析结束。
