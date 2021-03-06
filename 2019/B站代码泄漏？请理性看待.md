下午听说B站源代码泄漏，而且代码中还硬编码了用户口令，其实类似事件是偶然也是必然，今天说说我的一些想法，当然并不完全针对B站这个事件，也算反映一下目前的开发现状。

1：少谈概念，多做实事

其实很多大公司的技术人员很喜欢参加各类会议，使用各类新技术，给人布道，有的时候还鄙视其他公司的技术人员。

可自己呢？可能完全不了解自己公司的技术现状，不解决实际的问题，只关心所谓的战略。

针对B站这个事情，为什么会在 Github 上有这么个仓库？而且还硬编码口令？那技术管理者在做什么？我们的技术负责人应该实实在在帮助一线开发人员，针对问题提出改善意见，而且不能光提意见，最好手把手改善代码。

我曾经的领导，有一次我见他在手把手看业务人员写的 JavaScript 代码，而且一坐就很长时间，给我极大的震撼，我相信这种行动会感染每一个人，团队会更有凝聚力。

2：聪明的程序员

有一次我去上海见到一个开发人员，非常的聪明，什么也难不倒，可一看代码，觉得代码的个人色彩特别严重，没有任何的约束，注重实现，不注重“异常”，比如调用一个接口不考虑超时问问请题，也不会考虑数据库负载。这让我觉得很奇怪，是不是在技术管理和技术引导方面有一些缺失呢？

我觉得这是一些大公司要重点注意的问题，在发挥天赋的同时也要提升他们的软实力。

再比如我见过一个同事，也是非常聪明的一个人，完全做到随学随用，非常的有激情，可显得不正规化，比如什么文档也没有，硬编码也很严重，还忽视别人的意见。

我想表达的是，这些执行能力超牛的技术人员有时候太随性，可能潜在会带来一些问题，比如B站这个事件，怎么会傻到将代码发布上去呢？

3：无奈的程序员

就我观察到的，很多人接手别人代码的时候，都恨不得想重写，这说明什么？代码质量确实不理想，一方面可能是时间原因，也有可能是能力问题，而且会恶性循环。

比如老代码可能理解不透，也没有文档，完全靠自己的理解和猜，轻易不敢修改别人的代码，也有可能在错误的路上越来越远，总是修修补补，比如我今天就遇到了。

那别人说，你就不能把他弄好？可技术这个东西确实很奇怪，你没有正当理由，重新去优化，弄好了没人夸你，弄错了还弄的一身骚，另外优化也需要时间，别人问你，你优化的原因是什么？来自产品的需求？

这就是悲哀之处，程序员其实并不是开发产品代码才在干活，看代码，看书，学习其实都在工作。

恶性循环是一切悲剧的根源。

4：将心比心

以前我老说别人代码写的不好，说这很简单啊，这你怎么没考虑到，现在我不敢这么说了，一线开发真的不是想象的那么简单，谁都想做好，如果没有达到你的要求，可能有很多原因，千万别觉得什么都很简单，既然简单，你为啥不亲自写？

还是我上面说的，帮助人才重要，少埋怨，少形式帮助人。

5：开发和研发是割裂的

比如阿里云以前也出过看上去非常不可思议的小错误，微博也说过以后明星事件不会搞垮服务器，可最后呢？本质上研发人员和开发人员还是割裂。

研发人员一般开发中间件服务，不太会从开发的角度去考虑实际的应用问题，更不会管你的应用是不是有问题。

可开发人员开发出的服务才是面向最终用户的，实话实说，相对业务能力较差一点，因为大家都觉得 Web 界面，后台系统这些能用就行了，重视程度差了很多。

比如说你中间件性能非常高，可如果应用开发没写好，搞的性能特别差，那你这个中间件的高性能有啥用？

我表达的意思很明显了，技术开发一定要从整体全面考虑，尤其要重视最末端的开发，即面向用户的业务代码一定要注意。

总之，B站事件，大家不要觉得很奇怪，因为其实你也可能犯了很多错误，只是没暴露出来，而且诡异的是你其实知道问题，但没去改。

技术开发确实要实实在在，谨慎第一，我今年也犯了一些错误，希望吸收教训。

--- 

欢迎关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳），一直在用心写。