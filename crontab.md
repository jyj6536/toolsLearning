# crontab

crontab 可以以特定的时间间隔或者时间点调度命令的执行。

## 选项

+ -u 指定要操作哪个用户的 crontab
+ -l 列出当前 crontab
+ -r 删除当前 crontab
+ -e 编辑当前 crontab
+ -i 交互式删除当前 crontab

## 格式

~~~
*    *    *   *    *  Command_to_execute
|    |    |    |   |       
|    |    |    |    Day of the Week ( 0 - 6 ) ( Sunday = 0 )
|    |    |    |
|    |    |    Month ( 1 - 12 )
|    |    |
|    |    Day of Month ( 1 - 31 )
|    |
|    Hour ( 0 - 23 )
|
Min ( 0 - 59 )
~~~

指定多个值

+ 星号 指定所有可能的值
+ 逗号 指定值列表，例如“1,2,3”
+ 横线 指定值范围，例如“1-3”代表“1,2,3”
+ 斜线 指定值间隔，例如“*/3”代表每隔3个时间单位

## 例子

每天的12:59运行/usr/bin/sample.sh并且丢弃输出

```
59 12 * * * /usr/bin/sample.sh > /dev/null 2>&1
```

每天的21点执行sample.sh并且丢弃输出

```
0 21 * * * sample.sh 1>/dev/null 2>&1
```

每周二到周六的1点执行sample.sh并且丢弃输出

```
0 1 * * 1-6 sample.sh 1>/dev/null 2>&1
```

每天的5点和17点执行两次sample.sh

```
0 5,17 * * * sample.sh
```

每分钟都执行

```
* * * * * sample.sh
```

每10分钟执行一次

```
*/10 * * * * sample.sh
```

在1月，5月，8月执行

```
* * * jan,may,aug *  sample.sh
```

在周日，周五执行

```
0 17 * * sun,fri  sample.sh
```

在每月的第一个周日执行

```
0 2 * * sun  [ $(date +%d) -le 07 ] && sample.sh
```

每30秒执行一次

```
* * * * * sample.sh
* * * * *  sleep 30; sample.sh
```

执行多个命令

```
* * * * * sample1.sh; sample2.sh
```

### 特殊标签

~~~
@reboot    :    Run once after reboot.      重启时执行一次
@yearly    :    Run once a year, ie.  "0 0 1 1 *".  一年执行一次，1月1号0点0分执行
@annually  :    Run once a year, ie.  "0 0 1 1 *". 一年执行一次，1月1号0点0分执行
@monthly   :    Run once a month, ie. "0 0 1 * *". 一月执行一次，每月1号0点0分执行
@weekly    :    Run once a week, ie.  "0 0 * * 0". 一周执行一次
@daily     :    Run once a day, ie.   "0 0 * * *".  一天执行一次
@hourly    :    Run once an hour, ie. "0 * * * *".  一小时执行一次
~~~

## 注意

1. crontab脚本中要写绝对路径
2. crontab执行的脚本不会加载.bash_profile等文件，如果脚本中用到了相关环境变量等需要自行导入

https://www.tutorialspoint.com/unix_commands/crontab.htm