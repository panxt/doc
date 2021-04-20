
# logrotate配置

## 新建配置文件/etc/logrotate.d/graylog
/usr/andisec/graylog/log/*.log{
    copytruncate    
    rotate 10
    size 50M
    compress
    delaycompress 
    missingok
    ifempty
    dateext
    dateformat -%Y%m%d-%H 
}

## 配置每小时执行一次
crontab -e

#Logrotate
59  * * * * /usr/sbin/logrotate -f /etc/logrotate.d/graylog

## 配置解析
参数	说明
daily	每天轮替一次
weekly	每周轮替一次
monthly	每月轮替一次
yearly	每年轮替一次
rotate	保留几个轮替日志文件
ifempty	日志没有内容的时候也进行轮替
notifempty	若日志为空，则不进行轮替
create	旧日志文件轮替后创建新的日志文件
size	日志达到多少后进行rotate，size参数跟滚动周期参数：hourly、daily、monthly、yearly完全是互斥的，满足才滚动
minsize	文件容量一定要超过多少后才进行rotate
nocompress	轮替但不进行压缩
compress	压缩轮替文件
delaycompres： 和 compress 一起使用时，转储的日志文件到下一次转储时才压缩
dateext	轮替旧日志文件时，文件名添加-%Y %m %d形式日期，可用dateformat选项扩展配置。
dateformat .%s	对日期进行格式定制
nodateext	旧日志文件不使用dateext扩展名，后面序数自增如"*.log.1"
sharedscripts	作用域下文件存在至少有一个满足轮替条件的时候，执行一次prerotate脚本和postrotate脚本。
prerotate/endscript	在轮替之前执行之间的命令，prerotate与endscript成对出现。
postrotate/endscript	在轮替之后执行之间的命令，postrotate与endscript成对出现。
olddir	将轮替的文件移至指定目录下
missingok	如果日志文件不存在，继续进行下一个操作，不报错

更多信息请参考 man logrotate 帮助文档

## create和copytruncate底层区别
### create
这也就是默认的方案，可以通过 create 命令配置文件的权限和属组设置；这个方案的思路是重命名原日志文件，创建新的日志文件。详细步骤如下：

重命名正在输出日志文件，因为重命名只修改目录以及文件的名称，而进程操作文件使用的是 inode，所以并不影响原程序继续输出日志。
创建新的日志文件，文件名和原日志文件一样，注意，此时只是文件名称一样，而 inode 编号不同，原程序输出的日志还是往原日志文件输出。
最后通过某些方式通知程序，重新打开日志文件；由于重新打开日志文件会用到文件路径而非 inode 编号，所以打开的是新的日志文件。
如上也就是 logrotate 的默认操作方式，也就是 mv+create 执行完之后，通知应用重新在新文件写入即可。mv+create 成本都比较低，几乎是原子操作，如果应用支持重新打开日志文件，如 syslog, nginx, mysql 等，那么这是最好的方式。

不过，有些程序并不支持这种方式，压根没有提供重新打开日志的接口；而如果重启应用程序，必然会降低可用性，为此引入了如下方式。

### copytruncate
该方案是把正在输出的日志拷 (copy) 一份出来，再清空 (trucate) 原来的日志；详细步骤如下：

将当前正在输出的日志文件复制为目标文件，此时程序仍然将日志输出到原来文件中，此时，原文件名也没有变。
清空日志文件，原程序仍然还是输出到预案日志文件中，因为清空文件只把文件的内容删除了，而 inode 并没改变，后续日志的输出仍然写入该文件中。
如上所述，对于 copytruncate 也就是先复制一份文件，然后清空原有文件。

通常来说，清空操作比较快，但是如果日志文件太大，那么复制就会比较耗时，从而可能导致部分日志丢失。不过这种方式不需要应用程序的支持即可。



