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

<img src=../../../1 onerror=alert(1)>

