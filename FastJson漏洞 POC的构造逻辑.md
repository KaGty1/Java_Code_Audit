## FastJson的调用逻辑 (根据代码执行结果分析)

Fastjson漏洞产生在将 JSON字符串 反序列化为 JavaBean的过程中。

这是一段使用 parseObject()方法将 JSON字符串反序列化的代码：

### User类：

```
import java.util.Map;

public class User {
    private int id;
    private int age;
    private String name;
    private Map map;

    public Map getMap() {
        System.out.println("getMap");
        return map;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", age=" + age +
                ", name='" + name + '\'' +
                '}';
    }

    public int getId() {
        System.out.println("getId方法被调用");
        return id;
    }

    public void setId(int id) {
        this.id = id;
        System.out.println("setId : " + id);
    }

    public int getAge() {
        System.out.println("getAge方法被调用");
        return age;
    }

    public void setAge(int age) {
        this.age = age;
        System.out.println("setAge : " + age);
    }

    public String getName() {
        System.out.println("getName方法被调用");
        return name;
    }

    public void setStudentName(String name) {
        this.name = name;
        System.out.println("setName : " + name);
    }

}
```
### JSONUser类：

```
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;

public class JSONUser {
    public static void main(String[] args) {
        String s = "{\"@type\": \"com.xxxx.fastjson_demo.model.User\", \"age\":18,\"studentName\":\"admin\"}";
        JSONObject jsonObject = JSONObject.parseObject(s);
        System.out.println(jsonObject);
    }
}
```


可以看到 JSON字符串中的 "studentName" : "admin"，而不是 "name" : "admin"，是因为要向 setStudentName看齐：

"studentName" 运行结果 -> setStudentName()方法被调用

<img width="956" alt="image" src="https://github.com/user-attachments/assets/1f84d9a4-26e6-4bce-aa34-f9bf30aa1e8b" />

"name" 运行结果 -> setStudentName()方法未被调用

<img width="1028" alt="image" src="https://github.com/user-attachments/assets/1c145e5e-4229-4034-be6a-b5c3570d1a09" />

这个例子也就解释了为什么 Fastjson 1.2.24 - JdbcRowSetImpl链 POC中 需要给 autoCommit参数单独赋值(true和 false都行)，原因就是不赋值调用不了 setAutoCommit方法。

