.. _201201_a_bash_script_of_apache_log_analysis:

处理Apache日志的Bash脚本
===========================================

`ruanyifeng.com <http://www.ruanyifeng.com/blog/2012/01/a_bash_script_of_apache_log_analysis.html>`__

去年一年，我写了将近100篇网络日志。

现在这一年结束了，我要统计”访问量排名”，看看哪些文章最受欢迎。\ **（隆重预告：本文结尾处将揭晓前5名。）**

以往，我用的是\ `AWStats <http://awstats.sourceforge.net/>`__\ 日志分析软件。它可以生成很详细的报表，但是不太容易定制，得不到某些想要的信息。所以，我就决定自己写一个Bash脚本，统计服务器的日志，顺便温习一下脚本知识。

事实证明，这件事比我预想的难。虽然最终脚本只有20多行，但花了我整整一天，反复查看手册，确认用法和合适的参数。下面就是我的日志分析脚本，虽然它还不是通用的，但是我相信里面用到的命令，足以满足一般的日志分析需求，同时也是很好的学习Bash的实例。如果下面的每一个命令你都知道，我觉得可以堪称熟练使用Bash了。

**一、操作环境**

在介绍脚本之前，先讲一下我的服务器环境。

我的网络服务器软件是Apache，它会对每一个http请求留下记录，就像下面这一条：

    　　203.218.148.99 - - [01/Feb/2011:00:02:09 +0800] “GET
    /blog/2009/11/an\_autobiography\_of\_yang\_xianyi.html HTTP/1.1”
    200 84058 “http://www.ruanyifeng.com/blog/2009/11/freenomics.html”
    “Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-TW; rv:1.9.2.13)
    Gecko/20101203 Firefox/3.6.13”

它的意思是2011年2月1日，IP地址为203.218.148.99的访问者，向服务器请求访问网址/blog/2009/11/an\_autobiography\_of\_yang\_xianyi.html。

当天所有的访问记录，组成一个日志。过去一年，一共生成了365个日志文件。它们存放在12个目录中，每一个目录表示一个月（2011-01、2011-02、……2011-12），里面的日志文件依次为www-01.log、www-02.log、……www-31.log（假定该月有31天）。

在不压缩的情况下，365个日志文件加起来，要占掉10GB空间。我的目标就是分析这10GB日志，最后得到一个如下形式的访问量排名：

    | 　　访问量 网址1 　　访问量 网址2
    |  　　访问量 网址3
    |  　　…… ……

**二、为什么要用Bash**

很多计算机语言，都可以用来完成这个任务。但是，如果只是简单的日志分析，我觉得Bash脚本是最合适的工具。

主要原因有两个：一是”开发快”，Bash脚本是各种Linux命令的组合，只要知道这些命令怎么用，就可以写脚本，基本上不用学习新的语法，而且它不用编译，直接运行，可以边写边试，对开发非常友好。二是”功能强”，Bash脚本的设计目的，就是为了处理输入和输出，尤其是单行的文本，所以非常合适处理日志文件，各种现成的参数加上管道机制，威力无穷。

前面已经说过，最终的脚本我只用了20多行，处理10GB的日志，20秒左右就得到了结果。考虑到排序的巨大计算量，这样的结果非常令人满意，充分证明了Bash的威力。

**三、总体思路**

我的总体处理思路是这样的：

　　第一步，处理单个日志。统计每一天各篇文章的访问量。

　　第二步，生成月度排名。将每一天的统计结果汇总，得到月度访问量。

　　第三步，生成年度排名。将12个月的统计结果汇总，进行年度访问量的排序。

**四、处理单个日志**

以2011年1月1日的日志为例，它在目录2011-01之中，文件名是www-01.log，里面有10万条如下格式的记录：

    　　203.218.148.99 - - [01/Feb/2011:00:02:09 +0800] “GET
    /blog/2009/11/an\_autobiography\_of\_yang\_xianyi.html HTTP/1.1”
    200 84058 “http://www.ruanyifeng.com/blog/2009/11/freenomics.html”
    “Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-TW; rv:1.9.2.13)
    Gecko/20101203 Firefox/3.6.13”

处理这个日志，我只用了一行代码：


    　　awk ‘$9 == 200 {print $7}’ www-01.log \| grep -i
    ‘^/blog/2011/.\*.html$’ \| sort \| uniq -c \| sed ‘s/^ \*//g’ >
    www-01.log.result

它用管道连接了5个命令，每一个都很简单，我们依次来看：


**（1） awk ‘$9 == 200 {print $7}’ www-01.log**

awk命令默认用空格，将每一行文本分割成若干个字段。仔细数一下，我们需要的只是第7个字段，即http请求的网址，{print
$7}表示将第7个字段输出，结果就是：

    　　/blog/2009/11/an\_autobiography\_of\_yang\_xianyi.html

考虑到我们只统计成功的请求，因此再加一个限制条件，服务器的状态代码必须是200（表示成功），写成”$9
考虑到我们只统计成功的请求，因此再加一个限制条件，服务器的状态代码必须是200（表示成功），写成”$9
== 200”，即第9个字段必须是200，否则不输出第7个字段。

更精细的统计，还应该区分网络蜘蛛和真实访问者，由于我想不出简单的分辨方法，这里只好忽略了。

**（2）grep -i ‘^/blog/2011/.\*.html$’**

在输出的所有记录的第7个字段之中，并不是每一条记录都需要统计的。根据我的文章的命名特点，它们的网址应该都以”/blog/2011/”开头，以”.html”结尾。所以，我用一个正则表达式”^/blog/2011/.\*.html$”，找出这些记录。参数i表示不区分大小写。

**（3）sort**

这时，所有需要统计的记录应该都列出来了，但是它们的次序是杂乱的。接着，使用sort命令，不过目的不是为了排序，而是把相同的网址排列在一起，为后面使用uniq命令创造条件。

**（4）uniq -c**

uniq的作用是过滤重复的记录，只保留一行。c参数的作用，是在每行的开头添加该记录的出现次数。处理之后的输出应该是这样的：

    | 　　32
    /blog/2011/01/guidelines\_for\_english\_translations\_in\_public\_places.html
    |  　　32 /blog/2011/01/api\_for\_google\_s\_url\_shortener.html
    |  　　30 /blog/2011/01/brief\_history\_of\_arm.html

它表示以上三篇文章，在1月1日的日志中，分别有32条、32条、30条的访问记录（即访问次数）。

**（5）sed ‘s/^ \*//g’ > www-01.log.result**

上一步uniq命令添加的访问次数，是有前导空格的。也就是说，在上例的32、32、30之前有一连串空格，为了后续操作的方便，这里把前导空格删去。sed命令是一个处理行文本的编辑器，’s/^
\*//g’是一个正则表达式（^和\*之间有一个空格），表示将行首的连续空格替换为空（即删除）。接着，将排序结果重定向到文件www-01.result。单个日志分析就完成了。

**五、月度汇总排名**

经过上一步之后，1月份的31个日志文件，生成了31个对应的分析结果文件。为了汇总整个月的情况，必须把这31个结果文件合并。

**（6）合并分析结果**

    | 　　for i in www-\*.log.result 　　do
    |  　　　　cat $i » log.result
    |  　　done

这是一个循环结构，把所有www-01.log.result形式的文件，都写进log.result文件。

然后，我用一行语句，计算月度排名。

    　　sort -k2 log.result \| uniq -f1 —all-repeated=separate
    \|./log.awk \|sort -rn > final.log.result

这行语句由3个命令和1个awk脚本组成：

**（7）sort -k2 log.result**

由于是31个文件汇总，log.result文件里面的记录是无序的，必须用sort命令，将相同网址的记录归类在一起。但是此时，访问次数是第一个字段，网址是第二个字段，因此参数k2表示根据第二个字段进行排序。

**（8）uniq -f1 —all-repeated=separate**

uniq的作用是过滤重复的记录，参数f1表示忽略第一个字段（访问次数），只考虑后面的字段（网址）；参数表示all-repeated=separate，表示过滤掉所有只出现一次的记录，保留所有重复的记录，并且每一组之间用一个空行分隔。这一步完成以后，输出结果变成如下的形式：

    | 　　617
    /blog/2011/01/guidelines\_for\_english\_translations\_in\_public\_places.html
    |  　　455
    /blog/2011/01/guidelines\_for\_english\_translations\_in\_public\_places.html

    | 　　223 /blog/2011/01/2010\_my\_blogging\_summary.html
    |  　　253 /blog/2011/01/2010\_my\_blogging\_summary.html

相同网址都归在一组，组间用空行分割。为了简洁，上面的例子每一组只包含两条记录，实际上每一组都包含31条记录（分别代表当月每天的访问次数）。

**（9）log.awk脚本**

为了将31天的访问次数加总，我动了很多脑筋。最后发现，唯一的方法就是用awk命令，而且必须另写一个awk脚本。

    　　#!/usr/bin/awk -f

    | 　　BEGIN {
    |  　　　　RS=”” #将多行记录的分隔符定为一个空行
    |  　　}

    | 　　{ 　　　　sum=0 #定义一个表示总和的变量，初值为0
    　　　　for(i=1;i<=NF;i++){ #遍历所有字段 　　　　　　if((i%2)!=0){
    #判断是否为奇数字段 　　　　　　　　sum += $i
    #如果是的话，累加这些字段的值 　　　　　　} 　　　　}
    |  　　　　print sum,$2 #输出总和，后面跟上对应的网址
    |  　　}

我已经对上面这个\ `log.awk脚本 <http://pastebin.com/h6w6hUrK>`__\ 加了详细注释。这里再说明几点：首先，默认情况下，awk将”\\n”作为记录的分隔符，设置RS=”“表示改为将空行作为分隔符，因此形成了一个多行记录；其次，NF是一个awk的内置变量，表示当前行的字段总数。由于输入文件之中，每一行都包含两个字段，第一个是访问数，第二个是网址，所以这里做一个条件判断，只要是奇数字段就累加，偶数字段则一律跳过。最后，每个记录输出一个累加值和网址，它们之间用空格分割。

**（10）sort -rn > final.log.result**

对awk脚本的处理结果进行排序，sort默认使用第一个字段，参数r表示逆序，从大往小排；参数n表示以数值形式排序，不以默认的字典形式排序，否则会出现10小于2的结果。排序结果重定向到final.log.result。至此，月度排名完成。

**六、脚本文件**

用一个脚本，包含上面两节所有的内容。

    　　#!/bin/bash

    | 　　if ls ./\*.result &> /dev/null
    #判断当前目录中是否有后缀名为result的文件存在 　　then
    |  　　　　rm \*.result #如果有的话，删除这些文件
    |  　　fi

    　　touch log.result #创建一个空文件

    | 　　for i in www-\*.log #遍历当前目录中所有log文件 　　do
    　　　　echo $i … #输出一行字，表示开始处理当前文件 　　　　awk ‘$9
    　　　　echo $i … #输出一行字，表示开始处理当前文件 　　　　awk ‘$9
    == 200 {print $7}’ $i\|grep -i ‘^/blog/2011/.\*.html$’\|sort\|uniq
    -c\|sed ‘s/^ \*//g’ > $i.result #生成当前日志的处理结果 　　　　cat
    $i.result » log.result #将处理结果追加到log.result文件
    |  　　　　echo $i.result finished #输出一行字，表示结束处理当前文件
    |  　　done

    　　echo final.log.result … #输出一行字，表示最终统计开始

    　　sort -k2 log.result \| uniq -f1 —all-repeated=separate
    \|./log.awk \|sort -rn > final.log.result
    #生成最终的结果文件final.log.result

    　　echo final.log.result finished #输出一行字，表示最终统计结束

这就是月度排名的\ `最终脚本 <http://pastebin.com/UPUL2P8t>`__\ 。编写的时候，我假定这个脚本和log.awk脚本与日志文件在同一个目录中，而且这两个脚本都具有执行权限。

年度排名的处理与此类似，就不再赘述了。


=================================================================

关于脚本介绍，就到此为止。

接下来，揭晓2011年度访问量最大的我的5篇文章。

**第五名、\ `《我的Google
Adsense帐户被关》 <http://www.ruanyifeng.com/blog/2011/05/my_google_adsense_is_disabled.html>`__**

    我真想问问Google
    Adsense中国小组的成员：”难道你们都是机器人吗？难道你们看不出来哪些是流氓网站，哪些是正派网站吗？你们是否真的尽职工作了，还是在不负责任地草菅人命？”

**第四名、\ `《乔布斯的告别》 <http://www.ruanyifeng.com/blog/2011/10/steve_jobs_farewell.html>`__**

    斯蒂夫·乔布斯活着的时候，对病情讳莫如深，外界对他的身体状态毫不知情。现在他去世了，根据各方面透露的信息，我们终于可以还原他的病历，了解像他这样伟人怎样对待生与死。

**第三名、\ `《Dan计划：重新定义人生的10000个小时》 <http://www.ruanyifeng.com/blog/2011/04/the_dan_plan.html>`__**

    在此之前，他几乎没有打过高尔夫球，甚至对这项运动都没有太大兴趣。他的计划是，辞职以后，每天练习6个小时，一周练习6天，坚持6年，总计超过10000个小时，然后成为职业选手。他把这称为”Dan计划”。

**第二名、\ `《保持简单——纪念丹尼斯•里奇（Dennis
Ritchie）》 <http://www.ruanyifeng.com/blog/2011/10/dennis_ritchie.html>`__**

    13岁的丹尼斯•里奇（Dennis
    Ritchie），就这样随着父亲一起来到新泽西。那时，谁也没有想到，这个文静的少年将在这里待上一辈子，并且创造出改变世界的发明。

**第一名、\ `《人生只有900个月》 <http://www.ruanyifeng.com/blog/2011/05/900-month_lifespan.html>`__**

    你可以画一个30x30的表格，一张A4纸就够了。每过一个月，就在一个格子里打钩。你全部的人生就在这张纸上。你会因此有一个清晰的概念：你的人生是如何蹉跎的。

| （完）

.. note::
    原文地址: http://www.ruanyifeng.com/blog/2012/01/a_bash_script_of_apache_log_analysis.html 
    作者: 阮一峰 

    编辑: 木书架 http://www.me115.com