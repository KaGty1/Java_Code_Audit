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

在 parseObject()方法处打断点：

<img width="886" alt="image" src="https://github.com/user-attachments/assets/e4b654ae-84b2-4492-8853-ec9f1380c372" />

跟进 parseObject()方法：

<img width="1040" alt="image" src="https://github.com/user-attachments/assets/00697679-34c7-4b24-a2b6-24aca49ee102" />

obj是解析后的结果，然后返回。

继续跟进 parse()方法：

<img width="1023" alt="image" src="https://github.com/user-attachments/assets/d3b26d16-1a4e-43c0-8474-01293c473d8b" />

继续跟进 这个 parse()方法中的 parse()方法：

<img width="1012" alt="image" src="https://github.com/user-attachments/assets/6278ecd3-dc73-4f6e-958f-0194190d0ae0" />

使用 parser.parse()方法进行解析，并将解析结果赋值给 value之后返回，跟进 parser.parse()方法： (注：核心逻辑就在 parser.parse()方法中)

<img width="1015" alt="image" src="https://github.com/user-attachments/assets/e15718f9-eb58-43e9-b0dd-63c9379d5c07" />

parse()方法使用 switch来判断用户输入字符串的类型，case12为左大括号 '{'，由此判断输入为 JSON格式字符串，故创建 JSONObject对象进行解析，跟进 parseObejct()方法：

<img width="955" alt="image" src="https://github.com/user-attachments/assets/78508a86-886e-4ef2-ad5b-7f51fb6225db" />

parseObject() 方法用于解析 Key和 Value，首先解析Key -> 两个双引号之间，Key 为 @type

<img width="1042" alt="image" src="https://github.com/user-attachments/assets/3d2e3b9f-8c9e-4a3f-9dca-140d7d5c474f" />

获取到 Key值后，会做一个判断，判断 Key是否等于 JSON.DEFAULT_TYPE_KEY (即 @type)，结果是相等，则使用 loadClass()方法去加载这个类，并将其加入缓存中并赋值给 clazz

