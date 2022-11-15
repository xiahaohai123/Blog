### 背景


在一次工作中，我需要使用华为OBS的API接口来对接华为OBS。在添加MD5验证功能的时候，发现OBS接收的MD5验证字符串不是十六进制字符串，于是我去华为提供的SDK源码里看了看华为是怎么做MD5验证的，这一看，便发现了值得记录的东西。


### sun.misc.BASE64Encoder与java.util.Base64
- 在jdk1.8中，BASE64Encoder与Base64共同存在。
- 在jdk1.8之后，BASE64Encoder不再工作。
- 在jdk1.8之前，不存在Base64类。


### 问题
在这种情况下，<font color="red">**如何设计一个能兼容所有jdk版本的Base64编码程序？**</font>

### 解决

华为的SDK使用了反射，依次加载所有Base64类，加载失败也没有关系，总会有一个类加载成功。
所有被加载的类都会被标记，之后要进行base64编码的时候，检查哪个类被加载成功了，就使用哪个类的编码方法来编码。
以下是SDK的代码，省去不必要的部分：
```java
public class ReflectUtils {
    private static Class<?> androidBase64Class;

    private static Class<?> jdkBase64EncoderClass;

    private static Class<?> jdkBase64DecoderClass;

    private static Object jdkNewEncoder;

    private static Object jdkNewDecoder;

    static {
        try {
            androidBase64Class = Class.forName("android.util.Base64");
        } catch (ClassNotFoundException e) {
            ...
        }

        try {
            Class<?> base64 = Class.forName("java.util.Base64");
            jdkNewEncoder = base64.getMethod("getEncoder").invoke(null);
            jdkNewDecoder = base64.getMethod("getDecoder").invoke(null);
        } catch (ClassNotFoundException e) {
            ...
        } catch (otherException... e) {
            ...
        }

        try {
            jdkBase64EncoderClass = Class.forName("sun.misc.BASE64Encoder");
        } catch (ClassNotFoundException e) {
            ...
        }

        try {
            jdkBase64DecoderClass = Class.forName("sun.misc.BASE64Decoder");
        } catch (ClassNotFoundException e) {
            ...
        }
    }

    public static String toBase64(byte[] data) {
        if (androidBase64Class != null) {
            try {
                Method m = androidBase64Class.getMethod("encode", byte[].class, int.class);
                return new String((byte[]) m.invoke(null, data, 2), Charset.defaultCharset());
            } catch (Exception e) {
                throw new ServiceException(e);
            }
        }

        if (jdkNewEncoder != null) {
            try {
                Method m = jdkNewEncoder.getClass().getMethod("encode", byte[].class);
                return new String((byte[]) m.invoke(jdkNewEncoder, data), StandardCharsets.UTF_8).replaceAll("\\s", "");
            } catch (Exception e) {
                throw new ServiceException(e);
            }
        }

        if (jdkBase64EncoderClass != null) {
            try {
                Method m = jdkBase64EncoderClass.getMethod("encode", byte[].class);
                return ((String) m.invoke(jdkBase64EncoderClass.getConstructor().newInstance(), data)).replaceAll("\\s",
                        "");
            } catch (Exception e) {
                throw new ServiceException(e);
            }
        }

        throw new ServiceException("Failed to find a base64 encoder");
    }
}
```

### 总结
当我们遇到jdk版本兼容问题时，华为的SDK提供了一种解决思路，即使用反射一个个加载。这种解决思路不仅仅可以解决版本兼容问题，上方代码中可以看到还包含了android的base64编码器，这种解决思路也可以解决**平台适配**的问题。
