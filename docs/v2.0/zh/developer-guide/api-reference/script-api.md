# Script API

## 介绍

suiteboot是一个低代码框架，所以需要一些脚本来定制系统的自定义逻辑，suiteboot支持服务端的脚本、客户端脚本。



## 服务端脚本



服务端脚本需要按照suiteboot的指定规范使用Java编写，脚本可以访问suiteboot基础框架的任何对象



### 服务端脚本类别

1. RecordScript：定义在record级别的脚本，触发点有2个：

    1. record新增时触发，例如新增record可以更改record的value值，细致化分类如下

        1. 新增之前触发

        1. 新增之后触发（例监听数据变更）

    1. record和表单信息被返回给前端时触发，例如按照record的不同状态，展示不同的信息

1. ButtonScript：定义在button级别的脚本，这里的button只能绑定在record详情页面，触发时机有1个

    1. 在record详情页面点击button时触发，例如点击自定义按钮Approve更改当前record的状态

1. ScheduleScript：可以在脚本中编写定时任务，例如定时同步第三方数据，suiteboot框架底层使用quartz调度，目前已经标记为过时，由PowerJobScript代替

1. PowerJobScript：可以在脚本中编写定时任务执行的逻辑，由powerjob平台触发



### 服务端脚本规范接口

| Java类                                                    | 类方法 | 备注                     |
| -------------------------------------------------------- | --- | ---------------------- |
| com.suiteopen.boot.expand.sdk.service.RecordEventService |     | CustomRecord返回详情时触发的方法 |
|                                                          |     | CustomRecord新增之前触发的方法  |
|                                                          |     | CustomRecord新增之后触发的方法  |
| com.suiteopen.boot.expand.sdk.service.RecordEventService |     | 按钮被点击时触发               |
| com.suiteopen.boot.expand.sdk.service.ScheduleService    |     | 定时任务触发                 |
|                                                          |     |                        |



### RecordScript 示例



如下脚本展示了在当record的状态Pending和End的时，隐藏record详情页面的Reopen按钮和Complete按钮

```java
@Script(type = ScriptType.RECORD_EVENT)
public class InventoryCountEvent implements RecordEventService {

    final String reopenBtnName = "Reopen";
    final String endCountBtnName = "Complete";
    final Logger log = LoggerFactory.getLogger(InventoryCountEvent.class);

    final Map<String, List<String>> mapping = ImmutableMap.<String, List<String>>builder()
            .put(CountStatus.PendingCount.name(), ImmutableList.of(endCountBtnName))
            .put(CountStatus.End.name(), ImmutableList.of(reopenBtnName))
            .build();

    @Override
    public ScriptResult beforeLoad(ScriptContext ctx) {
        Record record = ctx.getCurrentRecord();
        Form form = ctx.getCurrentForm();

        log.info("form buttons {} ", form.getButtons().stream().map(t->t.getName()).collect(Collectors.toList()));

        Object status = record.getValue("status");
        List<String> btnNameList = mapping.get(status.toString());

        if (btnNameList == null || btnNameList.isEmpty()) {
            form.getButtons().clear();
            return ScriptResult.ok();
        }
        List<? extends Button> removedBtnList = form.getButtons().stream()
                .filter(b -> !btnNameList.contains(b.getName()))
                .collect(Collectors.toList());
        log.info("inventory count record value {}  btnNameList {}  removedBtnList {}, form.getButtons {} ",
                record.getData(), btnNameList, removedBtnList, form.getButtons().stream().map(Button::getName).collect(Collectors.toList()));
        form.getButtons().removeAll(removedBtnList);
        return ScriptResult.ok();
    }
}

```



### ButtonScript示例



如下脚本展示了当点击Approve页面时，更新record的status字段的值

```java
@Script(type = ScriptType.BUTTON_EVENT)
public class ApproveAdjustmentButtonEvent implements ButtonEventService {

    final InventoryCountAdjustmentService inventoryAdjustmentService =
            SpringUtil.getBean(InventoryCountAdjustmentService.class);
    final CustomRecordService customRecordService = SpringUtil.getBean(CustomRecordService.class);

    @Override
    public ScriptResult execute(ScriptContext ctx) {
        try {
            Record record = ctx.getCurrentRecord();
            Object data = ctx.getAttribute("data");
            if (ObjectUtil.isNotNull(data)) {
                Map<String, Object> recordDataMap = CastUtils.cast(data);
                customRecordService.submitFields(record.getRecordType(), record.getRecordId(), recordDataMap);
            }
            inventoryAdjustmentService.approve(ctx.getCurrentRecord().getRecordId());
        } catch (Exception e) {
            e.printStackTrace();
            return new ScriptResult(false, e.getMessage());
        }
        return new ScriptResult(true, "Success");
    }
}


```



### ScheduleScript示例

如下展示了定时（每隔10min）脚本从第三方平台同步数据

```java
@Script(type = ScriptType.SCHEDULE)
public class ListDSVLabelSchedule implements ScheduleService {
    private final FindCustomRecordService findCustomRecordService = SpringUtil.getBean(FindCustomRecordService.class);
    private final Logger log = LoggerFactory.getLogger(ListDSVLabelSchedule.class);

    // 周期：15分钟
    @Scheduled(cron = "0 0/10 * * * ?")
    @Override
    public ScriptResult execute(ScriptContext ctx) {
        List<CustomRecord> salesOrderList = listSalesOrder();
        for (CustomRecord customRecord : salesOrderList) {
            log.info("soId {} ", customRecord.getStr("otherrefnum"));
        }
        return ScriptResult.ok();
    }

    private java.util.List<CustomRecord> listSalesOrder() {
        Long shippingMethodId = getDSVMethodId();
        Date now = new Date();
        Date start = DateUtil.date(now.getTime() - (3L * 30 * 24 * 60 * 60 * 1000));

        return findCustomRecordService.listUnOrder(
                newArrayList(

                        and(
                                eq("delete", false),
                                eq("data.shipping_method", shippingMethodId),
                                eq("data.status", "pendingFulfillment"),
                                gte("data.order_date", start),
                                or(
                                        exists("data.aws_dsv_label_data", false),
                                        eq("data.aws_dsv_label_data", null),
                                        eq("data.aws_dsv_label_data", Collections.emptyList())
                                ),
                                lte("data.order_date", now)
                        )

                ),
                Sets.newHashSet(),
                "sales_order");
    }

    private Long getDSVMethodId() {

        List<CustomRecord> shippingMethodList = findCustomRecordService.listUnOrder(
                newArrayList(
                        eq("delete", false),
                        eq("data.id", WmsConstants.AWS_DSV_SHIP_METHOD)
                ),
                Sets.newHashSet(),
                "shipping_method"
        );

        if (shippingMethodList.isEmpty())
            throw new IllegalStateException("Not found any dsv shipping method");

        return shippingMethodList.get(0).getRecordId();
    }
}

```



## 客户端脚本

客户端脚本目前仅支持表单提交时触发的脚本



### OnFormSubmit示例

```javascript
onFormInit(({formData, parentData,formId}) => {
    if(!parentData) {
        if (formId === 1087) { formData.bt_type = 'bin_putway';}
    }
})
```

