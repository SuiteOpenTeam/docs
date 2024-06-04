***
# Restlet Script
当需要开放API提供给外部系统调用时，可以用RestletScript脚本，通过实现RestletService接口来编写Restlet脚本。

以下为Restlet脚本的Demo示例：

```java
package expand.script.restlet;

import cn.hutool.extra.servlet.ServletUtil;
import cn.hutool.extra.spring.SpringUtil;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import com.suiteopen.boot.common.domain.AjaxResult;
import com.suiteopen.boot.common.domain.customrecord.Record;
import com.suiteopen.boot.expand.sdk.annotation.Script;
import com.suiteopen.boot.expand.sdk.api.RecordApi;
import com.suiteopen.boot.expand.sdk.entity.ScriptType;
import com.suiteopen.boot.expand.sdk.service.RestletService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import java.util.Map;

@Script(type = ScriptType.RESTLET)
@RequestMapping("/integration")
public class DemoRestlet implements RestletService {
    private final RecordApi recordApi = SpringUtil.getBean(RecordApi.class);

    /**
     * 创建一个Item
     * @param request 请求参数
     */
    @Override
    @PostMapping("/item")
    public ResponseEntity<AjaxResult> post(HttpServletRequest request) {
        String requestBody = ServletUtil.getBody(request);
        JSONObject itemObj = JSONUtil.parseObj(requestBody);

        Record itemRecord = recordApi.create("item");
        itemObj.forEach(itemRecord::setValue);
        recordApi.save(itemRecord);

        return ResponseEntity.ok(AjaxResult.success());

    }

    /**
     * 根据ID获取一个Item详细信息
     * @param request 请求参数
     */
    @Override
    @GetMapping("/item/info")
    public ResponseEntity<AjaxResult> get(HttpServletRequest request) {
        Map<String, String> paraMap = ServletUtil.getParamMap(request);
        if (!paraMap.containsKey("id")) {
            return ResponseEntity.ok(AjaxResult.error("Item id is required"));
        }
        Long itemId = Long.parseLong(paraMap.get("id"));

        Record itemRecord = recordApi.load(itemId,"item");

        return ResponseEntity.ok(AjaxResult.success(itemRecord.getData()));

    }
}


```

## 调用Restlet脚本开放的API

要调用已发布的Restlet脚本开放的API，需要首先获取一个API AccessToken。

1. 在系统设置-开发-API Token中注册要对接的系统应用，系统会自动分配一个App Key和App Secret；

1. 将appKey，appSecret，timeStamp以及请求参数按字段名的字母顺序拼接，参数名+ 参数值为一键值对，参数之间以&连接，如: sign_type=MD5&veison=1.0；

1. 使用AES加密签名串，加解密算法/模式/补码方式：AES/ECB/PKCS5Padding，秘钥为App Secret；

1. 将加密串以base64编码得到签名；

1. 调用接口 /system/integration/openApi/accessToken，传递三个参数：appKey，timestamp和sign；

1. 接口返回Access Token和过期时间。

签名生成的伪代码如下：

```java
TreeMap<String, Object> paramMap = new TreeMap<>(String::compareTo);
        paramMap.put("appKey", appKey);
        paramMap.put("appSecret", systemOaApplication.getAppSecret());
        paramMap.put("timestamp", timestamp);
        StringBuilder bizContent = new StringBuilder();
        for (Map.Entry<String, Object> entry : paramMap.entrySet()) {
            bizContent.append(entry.getKey()).append("=").append(entry.getValue()).append("&");
        }
        bizContent.deleteCharAt(bizContent.length() - 1);
        AES aes = new AES(systemOaApplication.getAppSecret().getBytes(StandardCharsets.UTF_8));
        String encryptStr = aes.encryptBase64(bizContent.toString());
        String sign = StrUtil.replace(sign, " ", "+");

```



