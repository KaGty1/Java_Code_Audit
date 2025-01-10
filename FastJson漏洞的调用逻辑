## FastJson的调用逻辑 (根据代码执行结果分析)

Fastjson漏洞产生在将 JSON字符串 反序列化为 JavaBean的过程中。

这是一段使用 parseObject()方法将 JSON字符串反序列化的代码：

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

