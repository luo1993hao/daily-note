### 2018年6月
- ~~新需求~~
- 分布式调度任务
- **测试用例**
- 巡检sql

```
需求-- 一个站点，一天最多只有三条记录，早中晚班，每班的结果，如果有一个异常则为异常。
SELECT
  a.*
FROM
  (
    SELECT
      a.id,
      a.live_flag,
      a.user_id,
      a.station_id,
      a.remark,
      a.CODE,
      a.classic,
      a.inspect_date,
      a.result,
      a.create_time,
      b.NAME AS stationName,
      c.realname AS userName
    FROM
      monitor_inspect_all_term_record a
      LEFT JOIN station_basic b ON a.station_id = b.id
                                   AND b.live_flag = 1
      LEFT JOIN upms_user c ON c.user_id = a.user_id
    ORDER BY
      a.result
  ) a
GROUP BY
  a.inspect_date,
  a.station_id,
  a.classic
```
- amap接口重构