# 时间&时间戳
## 时间戳 ==> 时间
```
import time

#13位，毫秒
timestamp = 1552579199000
#换算为秒，若为10位（秒），则跳过这步
timestamp /= 1000

#转换成localtime
time_local = time.localtime(timestamp)
#转换成新的时间格式(2019-03-14 23:59:59)
t = time.strftime("%Y-%m-%d %H:%M:%S",time_local)
```
## 时间 ==> 时间戳
```
import time

t = "2019-03-14 23:59:59"

#转换成时间数组
timeArray = time.strptime(t, "%Y-%m-%d %H:%M:%S")
#转换成时间戳(1552579199.0)
timestamp = time.mktime(timeArray)
```
## 获取当前时间
```
>>> import time
>>> now = time.time()
>>> now
1553432442.5935082
```
## 时间差
```
>>> t1 = 1553432442.5935082
>>> t2 = 1552579199000/1000
>>> #做差得秒
... minus = t1 - t2
>>> #换算为天数
... minus/(60*60*24)
9.87550455449356
```
