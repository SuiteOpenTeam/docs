# Utility Script
在工具类脚本重定义常量、~~静态~~方法以供所有脚本复用。

Demo示例：

```java
package expand.script.utility;

import com.suiteopen.boot.expand.sdk.annotation.Script;
import com.suiteopen.boot.expand.sdk.entity.ScriptType;

import java.util.Random;

@Script(type = ScriptType.UTILITY)
public class DemoUtilityScript {

    public static String getRandomString(int length) {
        String str = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
        Random random = new Random();
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < length; i++) {
            int number = random.nextInt(62);
            sb.append(str.charAt(number));
        }
        return sb.toString();
    }
}


```

