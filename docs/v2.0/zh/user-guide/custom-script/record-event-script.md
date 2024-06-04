***
# RecordEvent Script
RecordEvent在用户对对象数据执行创建、加载、更新、复制、删除或提交等某些操作时执行。通过RecordEvent脚本，我们可以：

- 数据/权限等校验；

- 改变当前表单的样式

- 数据同步

- 业务流转

RecordEvent定义3个切入点，可类比AOP的切面，如下：

| 切入点                                       | 说明           |
| ----------------------------------------- | ------------ |
| beforeLoad(ScriptContext scriptContext)   | 读取对象数据详情时触发  |
| beforeSubmit(ScriptContext scriptContext) | 对象数据的写入操作前执行 |
| afterSubmit(ScriptContext scriptContext)  | 对象数据的写入操作后执行 |

通过实现RecordEventService接口来编写RecordEvent脚本。

示例脚本代码：

```java
package expand.script.recordevent;

import cn.hutool.core.collection.CollectionUtil;
import cn.hutool.core.util.ObjectUtil;
import cn.hutool.core.util.StrUtil;
import cn.hutool.extra.spring.SpringUtil;
import com.suiteopen.boot.common.domain.customrecord.Form;
import com.suiteopen.boot.common.domain.customrecord.Record;
import com.suiteopen.boot.common.domain.customrecord.RecordEventActionType;
import com.suiteopen.boot.expand.sdk.annotation.Script;
import com.suiteopen.boot.expand.sdk.api.RecordApi;
import com.suiteopen.boot.expand.sdk.entity.ScriptContext;
import com.suiteopen.boot.expand.sdk.entity.ScriptResult;
import com.suiteopen.boot.expand.sdk.entity.ScriptType;
import com.suiteopen.boot.expand.sdk.service.RecordEventService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Script(type = ScriptType.RECORD_EVENT)
public class DemoRecordEvent implements RecordEventService {
    private final Logger logger = LoggerFactory.getLogger(DemoRecordEvent.class);
    private final RecordApi recordApi = SpringUtil.getBean(RecordApi.class);

    @Override
    public ScriptResult beforeLoad(ScriptContext context) {
        Record currentRecord = context.getCurrentRecord();
        Form currentForm = context.getCurrentForm();
        if (CollectionUtil.isEmpty(currentForm.getButtons())) {
            return ScriptResult.ok();
        }
        // 状态判断，移除相关按钮
        String status = currentRecord.getValue("status");
        if (!StrUtil.equalsAny(status, "pending_receipt", "partial_receipt")) {
            currentForm.getButtons().removeIf(i -> StrUtil.contains(i.getName(), "入库"));
        }

        return ScriptResult.ok();
    }

    @Override
    public ScriptResult beforeSubmit(ScriptContext context) {
        // 新增和编辑场景下，校验邮箱必填
        if (StrUtil.equalsAny(context.getAction(), RecordEventActionType.CREATE.getValue(), RecordEventActionType.EDIT.getValue())) {
            Record currentRecord = context.getCurrentRecord();
            String email = currentRecord.getValue("email");
            if (StrUtil.isEmpty(email)) {
                return ScriptResult.fail("请填写邮箱!");
            }
        }
        return ScriptResult.ok();
    }

    @Override
    public ScriptResult afterSubmit(ScriptContext context) {
        Record currentRecord = context.getCurrentRecord();
        // 删除记录时，同步删除其他关联数据
        if (StrUtil.equals(context.getAction(), RecordEventActionType.DELETE.getValue())) {
            Long relatedUser = currentRecord.getValue("related_user");
            if (ObjectUtil.isNotNull(relatedUser)) {
                recordApi.delete(relatedUser, "user");
            }
        }

        return ScriptResult.ok();
    }

}


```

