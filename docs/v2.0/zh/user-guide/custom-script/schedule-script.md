# Schedule Script

通过定时任务脚本，按照设定的时间周期，一次性或循环执行某些业务逻辑，如异步批量处理单据，定时清理临时数据等。

通过实现ScheduleService接口来编写定时任务脚本。

在execute方法上添加@Scheduled注解，并填写cron表达式来确定任务执行周期。

示例Demo：

```java
package expand.script.schedule;

import cn.hutool.extra.spring.SpringUtil;
import com.suiteopen.boot.common.domain.customrecord.OperatorType;
import com.suiteopen.boot.common.domain.customrecord.Record;
import com.suiteopen.boot.common.domain.search.CustomSearch;
import com.suiteopen.boot.common.domain.search.SearchFilter;
import com.suiteopen.boot.common.domain.search.SearchResult;
import com.suiteopen.boot.expand.sdk.annotation.Script;
import com.suiteopen.boot.expand.sdk.api.RecordApi;
import com.suiteopen.boot.expand.sdk.api.SearchApi;
import com.suiteopen.boot.expand.sdk.entity.ScriptContext;
import com.suiteopen.boot.expand.sdk.entity.ScriptResult;
import com.suiteopen.boot.expand.sdk.entity.ScriptType;
import com.suiteopen.boot.expand.sdk.service.ScheduleService;
import org.springframework.scheduling.annotation.Scheduled;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;

@Script(type = ScriptType.SCHEDULE)
public class DemoSchedule implements ScheduleService {
    private final SearchApi searchApi = SpringUtil.getBean(SearchApi.class);
    private final RecordApi recordApi = SpringUtil.getBean(RecordApi.class);

	/**
     * 每天01:00清理过期的数据
     */
    @Override
    @Scheduled(cron = "0 0 1 * * ? *")
    public ScriptResult execute(ScriptContext context) {
        SearchResult searchResult = searchRecords();
        for (Record row : searchResult.getRows()) {
            recordApi.delete(row.getRecordId(), row.getRecordType());
        }
        return ScriptResult.ok();
    }

    private SearchResult searchRecords() {
        CustomSearch customSearch = searchApi.create("food");
        List<List<SearchFilter>> searchFilter = new ArrayList<>();
        List<SearchFilter> searchFilters = new ArrayList<>();
        searchFilters.add(new SearchFilter("expired_date", OperatorType.LESS_THAN, new Date()));
        searchFilter.add(searchFilters);
        customSearch.setSearchFilters(searchFilter);
        customSearch.setPageSize(100);
        customSearch.setAsc(false);
        customSearch.setSortBy("createDate");
        return searchApi.run(customSearch);
    }
}


```

