.. _articles2053:

最为奇怪的程序语言的特性
========================

2010年1月21日 `陈皓 <http://coolshell.cn/articles/author/haoel>`__

这些最为奇怪的程序语言的特性，来自stackoverflow.com，原贴在\ `这里 <http://stackoverflow.com/questions/1995113?sort=votes&page=1>`__\ 。我摘选了一些例子，的确是比较怪异，让我们一个一个来看看。 

**1、C语言中的数组** 

在C/C++中，a[10] 可以写成 10[a] 

“Hello World”[i] 也可以写成 i[“Hello World”] 

这样的特性是不是很怪异？如果你想知道为什么的话，你可以看看本站的这篇文章——《\ `C语言的谜题 <http://coolshell.cn/articles/945.html>`__\ 》中的第12题。 

**2、在Javascript中** 

|  ‘5’ + 3 的结果是：’53’
|   ‘5’ – 3 的结果是：2 

**3、C/C++中的Trigraphs** 

::

    int main() {
       cout << "LOL??!";
    }

上面的这段程序会输出： “LOL\|”，这是因为 ??! 被转成了 \|
，关于Trigraphs，下面有个表格： 

??= # ??( [ ??/ \\ ??) ] ??’ ^ ??< { ??! \| ??> } ??- ~

  

**4、JavaScript 的条件表** 

看到下面这个表，不难理解为什么Javascript程序员为什么痛苦了——《\ `Javascript程序员嘴最脏?? <http://coolshell.cn/articles/1850.html>`__\ 》 

::


    ''        ==   '0'           //false
    ''        ==   '0'           //false
    0         ==   ''            //true
    0         ==   ''            //true
    0         ==   '0'           //true
    0         ==   '0'           //true
    false     ==   'false'       //false
    false     ==   'false'       //false
    false     ==   '0'           //true
    false     ==   '0'           //true
    false     ==   undefined     //false
    false     ==   undefined     //false
    false     ==   null          //false
    false     ==   null          //false
    null      ==   undefined     //true
    null      ==   undefined     //true
    " \t\r\n" ==   0             //true

 

**5、Java的Integer cache** 

::

    Integer foo = 1000;
    Integer bar = 1000;
     
    foo <= bar; // true
    foo >= bar; // true
    foo >= bar; // true
    foo == bar; // false
     
    //然后，如果你的 foo 和 bar 的值在 127 和 -128 之间（包括）
    //那么，其行为则改变了：
     
    Integer foo = 42;
    Integer bar = 42;
     
    foo <= bar; // true
    foo >= bar; // true
    foo >= bar; // true
    foo == bar; // true

为什么会这样呢？你需要了解一下Java Interger
Cache，下面是相关的程序，注意其中的注释

::

    /**
         * Returns a Integer instance representing the specified
         * int value.
         * If a new Integer instance is not required, this method
         * should generally be used in preference to the constructor
         * {@link #Integer(int)}, as this method is likely to yield
         * significantly better space and time performance by caching
         * frequently requested values.
         *
         * @param  i an int value.
         * @return a Integer instance representing i.
         * @since  1.5
         */
        public static Integer valueOf(int i) {
            if(i >= -128 && i <= IntegerCache.high)
                return IntegerCache.cache[i + 128];
            else
                return new Integer(i);
        }

**5、Perl的那些奇怪的变量**

::

    $.
    $_
    $_#
    $$
    $[
    @_

其所有的这些怪异的变量请参看：\ `http://www.kichwa.com/quik\_ref/spec\_variables.html <http://www.kichwa.com/quik_ref/spec_variables.html>`__ 

**6、Java的异常返回**

请看下面这段程序，你觉得其返回true还是false？

::

    try {
        return true;
    } finally {
        return false;
    }

在 javascript 和python下，其行为和Java的是一样的。 

**7、C语言中的Duff device**

下面的这段程序你能看得懂吗？这就是所谓的Duff Device，相当的怪异。

::

    void duff_memcpy( char* to, char* from, size_t count ) {
        size_t n = (count+7)/8;
        switch( count%8 ) {
        case 0: do{ *to++ = *from++;
        case 7:     *to++ = *from++;
        case 6:     *to++ = *from++;
        case 5:     *to++ = *from++;
        case 4:     *to++ = *from++;
        case 3:     *to++ = *from++;
        case 2:     *to++ = *from++;
        case 1:     *to++ = *from++;
                }while(--n>0);
        }
    } 

**8、PHP中的字符串当函数用**

PHP中的某些用法也是很怪异的

::

    $x = "foo";
    function foo(){ echo "wtf"; }
    $x();

**9、在C++中，你可以使用空指针调用静态函数**

::

    class Foo {
      public:
        static void bar() {
          std::cout << "bar()" << std::endl;
        }
    };
     
    int main(void) {
      Foo * foo = NULL;
      foo->bar(); //=> WTF!?
      return 0; // Ok!
    }

呵呵。的确是挺怪异的。

.. |image6| image:: /coolshell/static/20140922105053045000.jpg

.. note::
    原文地址: http://coolshell.cn/articles/2053.html 
    作者: 陈皓 

    编辑: 木书架 http://www.me115.com