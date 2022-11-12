# JacksonJson关闭忽略反序列化时的未知字段

## 常见问题

当我们要把一个json字符串反序列化成一个java对象时，可能会遇到一个异常:
`UnrecognizedPropertyException: Unrecognized field <field>, not marked as ignorable`。

意思是反序列化时，遇到了一个不认识的字段，所以抛出了异常。

### 举例

假如我们有这么一个代码

```java
public class UnknownDes {
    public static void main(String[] args) throws IOException {
        Map<String, Object> map = new HashMap<>();
        map.put("username", "111");
        map.put("password", "222");
        map.put("port", 222);
        map.put("host", "10.10.10.10");
        map.put("unknown", "10.10.10.10");

        ObjectMapper objectMapper = new ObjectMapper();
        String json = objectMapper.writeValueAsString(map);
        System.out.println(json);
        Data data = objectMapper.readValue(json, Data.class);
        System.out.println(data);
    }

    static class Data {
        private String username;
        private String password;
        private Integer port;
        private String host;

        public String getUsername() {
            return username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public String getPassword() {
            return password;
        }

        public void setPassword(String password) {
            this.password = password;
        }

        public Integer getPort() {
            return port;
        }

        public void setPort(Integer port) {
            this.port = port;
        }

        public String getHost() {
            return host;
        }

        public void setHost(String host) {
            this.host = host;
        }

        @Override
        public String toString() {
            return "Data{" +
                    "username='" + username + '\'' +
                    ", password='" + password + '\'' +
                    ", port=" + port +
                    ", host='" + host + '\'' +
                    '}';
        }
    }
}
```

最终执行就会出现异常，因为unknown字段未能在Data中找到对应的数据字段。

## 解决方案

### 全局处理

`objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);`

这么设置之后，由该对象执行的反序列化都会忽略Unknown字段。

### 注解法

`@JsonIgnoreProperties(ignoreUnknown = true)`

将该注解加在类字段上后，该类的反序列化会忽略Unknown字段。