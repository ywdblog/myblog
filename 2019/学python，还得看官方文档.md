掐指一算，python2 还有十个月的时间官方就不再维护了，从去年的下半年开始，我就一直在学python3，说说简单的一些感受，以及推荐一些学习资料。

虽然自认为是一个python程序员，但工作中python用的不多，甚至没有进行过python web 开发，平时偶尔用python写一些脚本，水平算中等偏下。

那么既然对python不是特别了解，且同时python3改变很大，为什么不考虑去学一门新的语言呢？比如说 go 语言，一方面不想轻易放弃，况且python的态势也非常好；另外python是非常现代化的编程语言，熟练了解它的特性，相信再去学其他语言就会事半功倍，所以我的目标就是精通python。

如果你从来没有学过python，那么恭喜，完全可以跳过python2，直接学习python3；当然对于python2的程序员来说，也大可不必懊恼，掌握好python2，对于python3中演变会有更深刻的理解。

现在面临的问题就是如何学习python3，学习资料从哪儿找呢？python书籍多如牛毛，但完全基于python3的并不多，如果你完全不想涉及python2，或者怕误导你，那么选择学习资料非常重要。

从去年下半年开始，我首先学习的资料就是官方的《Tutorial 教程》，由于有一定的基础，同时它是入门资料，所以读起来还算轻松，能够大体的了解python基础框架和语法。虽然是 Tutorial 教程，但用语还是非常精准的，不会产生歧义，所以如果你象了解python3或有一定的编程基础，所以推荐一读。

当然官方《Tutorial 教程》没有涉及太多的高阶知识，比如装饰器等概念，如果想进一步进阶，推荐《Python3 Tutorial》，这是一个python3教程，地址是 https://www.python-course.eu/python3_course.php，最大的优点就是没有废话，不会涉及太多的基础知识，适合有一定编码能力的python程序员使用，很多python核心的概念都讲到了，实用性非常好，讲解的也很通俗。看完后，我对于迭代器、闭包、生成器、装饰器、类等概念有了进一步的了解，当然光看是没有用的，比如生成器实际的应用场景是什么？这个可能要通过阅读优秀代码才能深刻掌握。

我在学习python的时候，看过《python核心编程（第二版）》，这本书非常老，这次又拿出来复习来一遍，主要看了第一部分（第二部分高级主题没看），而且是对照着英文版一起看，客观的说，翻译的不错，当然关于python2的部分（尤其完全废弃的部分，比如python2编码），选择跳过。如果你也有这本书，那么我建议读一读。

了解Python基础语法和核心概念后，后面就是编程了，在编码过程中，使用最多的就是官方的《Library Reference》，目前我只是看了很少一部分，如果熟练掌握，后面遇到问题的时候就用不着 google 了，如果觉得 《Library Reference》 说的有点繁琐和不通俗，可以参考 《PyMOTW-3》，地址是 https://pymotw.com/3，里面有很多的例子，可以借鉴使用。

对于Python包开发来说，使用第三方模块非常常见，如何寻找、下载、安装、使用包是非常关键的能力，由于python历史久远，包安装相对混乱，比如你知道 pip 和 esay_install 的区别吗？知道 whell 和 dist 的概念吗？所以这一块是我重点想学习的。

去年基于 python2 也温习了下包安装、分发的概念，写了两篇文章，分别是[《手把手教你发布一个Python包》](https://mp.weixin.qq.com/s/U5-fkTg26xkkeYssJXMDqg)和[《在Python中安装包的三种方法》](https://mp.weixin.qq.com/s/9TsMkq4Y3jSaqqaqB3g8yg)，但不得不说，python3 对于包安装和分发有了很大的改动，废弃了很多老的工具，如果你想系统学习，必须读一遍《Python Packaging Authority》这个第三方的文档，地址是 https://www.pypa.io/en/latest/future，这也是我下一阶段的学习重点，Python官方也有对应的教程，比如《Installing Python Modules》和《Distributing Python Modules》，但写的不是很详细，只是浅尝辄止，但对于整体理解包安装、分发还是非常有帮助的，后面我也想简单的写一篇文章，从python3的角度从全局把握包的概念。

--- 

推荐大家关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳）和我的书《深入浅出HTTPS：从原理到实战》
 