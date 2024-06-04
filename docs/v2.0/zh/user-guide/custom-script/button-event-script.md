
# ButtonEvent Script

自定义按钮请见，通过实现ButtonEventService接口来编写按钮事件脚本。

表单按钮示例Demo：

```java
package expand.script.button;

import cn.hutool.core.util.ObjectUtil;
import cn.hutool.extra.spring.SpringUtil;
import com.suiteopen.boot.common.domain.customrecord.Record;
import com.suiteopen.boot.expand.sdk.annotation.Script;
import com.suiteopen.boot.expand.sdk.api.RecordApi;
import com.suiteopen.boot.expand.sdk.entity.ScriptContext;
import com.suiteopen.boot.expand.sdk.entity.ScriptResult;
import com.suiteopen.boot.expand.sdk.entity.ScriptType;
import com.suiteopen.boot.expand.sdk.service.ButtonEventService;

@Script(type = ScriptType.BUTTON_EVENT)
public class DemoFormButtonEvent implements ButtonEventService {

    private final RecordApi recordApi = SpringUtil.getBean(RecordApi.class);

    @Override
    public ScriptResult execute(ScriptContext context) {
        Record record = context.getCurrentRecord();
        Object approvalNote = context.getAttribute("approval_note");
        if (ObjectUtil.isNull(approvalNote)) {
            return ScriptResult.fail("请填写审批意见!");
        }
        record.setValue("is_approved", true);
        record.setValue("approval_note", approvalNote);
        recordApi.save(record);

        return ScriptResult.ok();
    }
}


```



视图按钮示例Demo：

```java
package expand.script.button;

import cn.hutool.extra.spring.SpringUtil;
import com.suiteopen.boot.common.domain.customrecord.Record;
import com.suiteopen.boot.expand.sdk.annotation.Script;
import com.suiteopen.boot.expand.sdk.api.RecordApi;
import com.suiteopen.boot.expand.sdk.entity.ScriptContext;
import com.suiteopen.boot.expand.sdk.entity.ScriptResult;
import com.suiteopen.boot.expand.sdk.entity.ScriptType;
import com.suiteopen.boot.expand.sdk.service.ButtonEventService;
import org.springframework.data.util.CastUtils;

@Script(type = ScriptType.BUTTON_EVENT)
public class DemoViewButtonEvent implements ButtonEventService {

    private final RecordApi recordApi = SpringUtil.getBean(RecordApi.class);

    @Override
    public ScriptResult execute(ScriptContext context) {
        // 批量删除数据
        Record record = context.getCurrentRecord();
        Long[] selectedRecordIds = CastUtils.cast(context.getAttribute("recordIds"));
        for (Long recordId : selectedRecordIds) {
            recordApi.delete(recordId, record.getRecordType());
        }

        return ScriptResult.ok();
    }
}


```

