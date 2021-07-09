# 【λ 演算在 PY】扔掉游标卡尺后用 PY 拼接代码
<!-- 【λ 演算在 PY】扔掉游标卡尺后用 PY 拼接代码 -->

> PY 里字符串内容当 PY 代码执行用 `exec()` 或 `eval()` 。（这种操作是不是很像自己跟自己交易？😏）
> 
> - 前者可以执行一般的代码，但不会有返回，你就当执行一个 `.PY` 的脚本。  
> - 后者可以有返回，但只能是一条语句。嗯，一条语句。🙃  
> 

这个挺不错的： `https://github.com/pyecharts/pyecharts` 特别是图标。

> `'我永远喜欢海绵宝宝'`
> 


它有个示例。我跟着[这里](https://pyecharts.org/#/zh-cn/global_options)和[这里](https://gallery.pyecharts.org/#/Bar/stack_bar_percent)稍微改了一下，成了这样：

```python
from pyecharts.charts import Bar
from pyecharts import options as opts

(
    Bar(init_opts=opts.InitOpts(width="680px",height="390px",page_title="HAHAHAHA"))
    .add_xaxis(["衬衫", "毛衣", "领带", "裤子", "风衣", "高跟鞋", "袜子"])
    .add_yaxis("商家A", [114, 55, 27, 101, 125, 27, 105])
    .add_yaxis("商家B", [57, 134, 137, 129, 145, 60, 49])
    .set_global_opts(title_opts=opts.TitleOpts(title="某商场销售情况"))
).render("shangjias.html")

```

然后我有了一个想法：我想自动生成一个中间部分随机的可以指定商家数目的这段代码。


好，一边查一边干！（一边查是因为我不会 PY ，也正因此才有了这份……笔记？（扔游标卡尺则是顺带。来都来了，就顺便扔了嘛。。。。））

## 辨析 `exec` 和 `eval` 

其实 Bash 也有这俩玩意。不过，我只会玩前者，用来模仿一个 *先完成本次调用的函数再开始下次调用* 的效果，从而在 Bash 上实现 *函数式* 的尾递归。看 PY 里也有，一开始还有点小激动，但 PY 的进程逻辑跟 Bash 肯定不一样，我就姑且先放下这个小激动。

现在的大业是，玩 PY 不用游标卡尺，和，用 PY 玩动态代码，这两项，别的《稍》后再研究。。

其实辨析很简单，在 PY 里：

- 前者可以执行一般的代码，但不会有返回，你就当执行一个 `.PY` 的脚本，无返回。  
- 后者可以有返回，但只能是一条语句。嗯，一条语句。🙃  

> **`Note`**
> 
> **其实这个设计很合理。**  
> **不然，就会像 Powershell 那样，一个函数竟然可以有多个返回了，那就太吓人了。**  
> 

《一样简单的示例》：

你先复制下面的代码进入你的 PY 然后敲一下回车，你就知道它意味着什么了：

```python
'{}\n{}'.format(1,2)

```

知道上面的意味着什么就知道下面的意味着什么了：

```python
'{}\n{}'.format("'yaya{}rara'",".format(2)")

```

就知道下面的代码意味着什么了（返回结果上这两个一样但建议用前者即 *格式数据分别定义* 原则）：

```python
'({}\n{})'.format("'yaya{}rara'",".format(2)")
'{}\n{}'.format("('yaya{}rara'",".format(2))")

```

行了，睁大眼睛，我们要开始了！

我们先看看 `exec` 和 `eval` 解析下面那坨字符串时候的效果：

```python
exec('({}\n{})'.format("'yaya{}rara'",".format(2)")) # 没返回也没打印也没报错（代码没打印也没错）
eval('({}\n{})'.format("'yaya{}rara'",".format(2)")) # 返回: 'yaya2rara'

```

再看看这个：

```python
exec('{}\n{}'.format("'yaya{}rara'",".format(2)")) # 报错
eval('{}\n{}'.format("'yaya{}rara'",".format(2)")) # 报错

```

报错一样，因为 PY 不支持比较自由的格式化，**除非在括号里🙃**。（这也是我的可乘之机，当然我不光用圆括号 `()` ，主要还是方括号 `[]` 和拉拔哒 `lambda` 。）

## 堆数

这里先留下一个小疑惑：一般 Java 或 Scala 都可以写全类名来避免 `import` ，但 PY 就是不行。当然，也可能是还有别的我不知道的办法。但不管怎样，既然如此，我就给引入的随机重新起个名字好了，就叫 `随机` 好了！

```python
import random as 随机

```

> **`Note`**
> 
> 可能我需要特别明着强调一下：**下面的代码执行的前提是已经执行过了上面的这个 `import` 。**
> 其实我很不喜欢这种设计：我觉得一行代码自带了所有部分会比较好，而不是它只是**必要工作**的一部分。如果只是一部分，**你让孩子们上哪儿找别的部分去呢？**孩子们就要伤心了呀🤢🤢。。。。
> 


我们先搞一个随机列表：

```python
[随机.randint(222,333) for i in range(12)]

```

可以多执行几次，看看样子。为什么让你看看它的样子？因为你马上就再也见不到它了。。。。

在见不到它之前，先补充一下关于 `lambda` 的知识点。不难，执行下面代码比比看就好：

```python
1+2 # 返回: 3
(lambda x,y: x+y)(1,2) # 返回: 3

1+1 # 返回: 2
(lambda x: x+x)(1) # 返回: 2

[e+7 for e in range(4)] # 返回: [7, 8, 9, 10]
(lambda x: [e+7 for e in range(x)])(4) # 返回: [7, 8, 9, 10]

['ahaha {} jijijijijiji'.format(e) for e in range(4)] # 返回你自己试。
(lambda x: ['ahaha {} jijijijijiji'.format(e) for e in range(x)])(4) # 返回你自己试（反正跟上面一样）。

```

这么看来，可能会有人说了， "这个 `lambda` 不就是多此一举嘛" ，这你就不知道了，如果只让你写一行代码，这就不能赋值了，但又要你把某个随机数用两次，你怎么办呢？

先不说这个，反正后面马上会遇到。🙃

而且需要用到 `lambda` 的地方已经遇到了，那就是我们想要提取 *写定的数值* 为 *变量* 。

```python
(lambda cnt: [随机.randint(222,333) for i in range(cnt)])(7)

```

这样就是定义一个函数并立刻调用，返回结果是有 `7` 个随机数的列表，随机数是 `222` 和 `333` 之间的 `int` 数（即整数）。

另外， `lambda` 函数定义本身就相当于是一个 *写定的数值* ，只不过类型不是 `int` 也不是一般的别的什么类型。总之，它会像一个数值一样被返回。从而，可以有下面的定义：

```python
lambda cnt: 
    (lambda low,high: 
        ( [随机.randint(low,high) for i in range(cnt)] )
    )

```

调用：

```python
(lambda cnt: 
    (lambda low,high: 
        ( [随机.randint(low,high) for i in range(cnt)] )
    )
) (7) (222,333)

```

这里其实发生了两次调用：

- 第一次调用的参数列表是 `(7)` ，返回了一个**内容**是这样子的值： `lambda low,high: ( [随机.randint(low,high) for i in range(7)] )`   
- 然后它后面又有参数列表 `(222,333)` ，就又被调用了一次，这次调用就相当于**执行** `[随机.randint(222,333) for i in range(7)]` 了。  
  （事实上，**执行** `lambda 啥啥: 啥啥` **命令**的**返回结果**就是一个**内容**为 `lambda 啥啥: 啥啥` 的 *确定值* 。）  

> **`Note`**
> 
> 其实，把函数作为值传来传去这种事，用 Lisp 系列的语言写起来会非常直观。只不过本文是在玩 PY ，所以就不打算好好介绍 Lisp 的 *S 表达式* 的那种代码了，可以自己查一查 Scheme 的语法以及 `lambda` 的写法。
> 

上面的调用结果就是 *一个有 `7` 个随机数的列表，随机数是 `222` 和 `333` 之间的 `int` 数（即整数）* 。

> **好了，我现在想要搞好几个这样的列表！**

那么好，假设我们有这样一个**列表**：

```python
[
    ('商家1',(200,300)),
    ('商家2',(50,600)),
    ('商家3',(80,100))
]

```

接下来只需要把右边的变成参数列表调用之前的函数，然后把结果和前面的 `'商家x'` 拼起来就行……

> **这不够！我还想要每个的参数还都不一样！我不想自己定，就想用随机数！**

好，那就先生成一个可以指定数目的这样的列表。

我们需要对 `随机.randint(x,y)` 这个函数传入随机的参数，并且要确保第一个小于第二个。

**我的思路是，确定第一个随机值，第二个就随机加一个增量。**

思路有了，开干。

首先，弄一个 `'商家X'` 后面那样的一个元组出来，随机地：

```python
(随机.randint(40,300), 随机.randint(40,300) + 随机.randint(10,600))

```

等等！完蛋去吧！这样完全有可能会让第二个数更小！因为两个 `随机.randint(40,300)` 完全可能不是一样的值！！！！

怎么办？我现在已经不允许我自己把值赋值给变量了！怎么办！

> `lambda` ：别担心，小伙子！想一想这个美妙的结构： `(lambda a (a a))` （这个是 Scheme 里的写法，等价的 Python 写法是这样： `(lambda a: a(a))` 。怎么样？是不是 Scheme 比较牛逼？😶）（另外这个结构来自 *Y 组合子* 。）  
> 其实这样举例更简单：  `(lambda x: x+x)` 这不就是用了两次！同一个数！！！！不信可以验证： `(lambda x: (x,x+x))(随机.randint(40,300))`   
> 

总之，现在我们有主意搞定上面的需要了：

```python
(lambda rndlow, add: (rndlow , rndlow + add) ) (随机.randint(40,300), 随机.randint(10,600) )

```

这里就不把本来的第二个参数打印出来了，想看看的话无非是可以这样： `(lambda rndlow, add: (rndlow , rndlow + add , add)) (随机.randint(40,300), 随机.randint(10,600) )` ，其中第二个数肯定就是第一个第三个的和，每次肯定都是。

现在，一个这玩意（就是一个二值元组），我们搞出来了。现在，我们需要一个列表，列表里，要有好几个这玩意。

 指定长度的列表咋弄？看：

```python
[elem*2 for elem in range(12)]
['heihei {} miaomiao'.format(elem) for elem in range(12)]

[随机.randint(40,300) for elem in range(12)] # 返回 12 个 40 ~ 300 的随机数
[(随机.randint(40,300), 随机.randint(10,600)) for elem in range(12)] # 返回 12 个的二值元组，第一个值是 40 ~ 300 的随机数，第二个值是 10 ~ 600 的随机数（两个值的数值当然并没什么关系在逻辑上它们是独立着随机的）

```

后两行其实最后没有用到 `elem` ，被传递下去的 *信息* 只有 *`elem` 一共有几个* 这一点。

> 其实这个 `[新集合单个元素形式 for 依据元素 in 依据元素来源集合]` 的写法，一开始我十分不解，看不懂它各个部分为什么是这样的含义，因为我总往 `for` 循环的形式上想。后来了解 Erlang ，看到了列表构造器是这样写的： `[新集合单个元素形式 || 依据元素 <- 依据元素来源集合]` 我就一下就明白 PY 这种设计是哪儿来的想法了。
> 

行了，我想要一个二值元组，但第二个一定比第一个大：

```python
[ (lambda rndlow,add: (rndlow,rndlow+add)) (随机.randint(40,300), 随机.randint(10,600) ) for will_not_use in range(13) ]

```

来， `13` 个。

格式化一下吧，已经很长了。

```python
[   (lambda rndlow,add: 
        (rndlow,rndlow+add)
    ) (随机.randint(40,300), 随机.randint(10,600) ) 
    for will_not_use in range(13) 
]

```

发现没？**虽然我们是在玩 PY 但是这里居然是 *格式化自由* 的！！！！**

（这只是借用 PY 的特性罢了：**括号里连续空白符会被视为一个** —— 当然，这是我的个人总结，不代表 PY 那个委员会的态度。。。）

> **`Note`**
> 
> 这里我的格式化原则是这样的：
> 
> - 值个数确定的列表或元组，前后括号上下对齐、内容加一次缩进一项一行。
> - 列表生成器，前方括号后若干空格后跟第一项，然后 `for` 表达式另起一行并一次缩进。
> - `lambda` 表达式的话，被调用当然要包上圆括号了就。定义头可以尽量紧跟着前圆括号，也可以换行，换行的话，相对于前圆括号所在层级，再往后缩进两层，这个前括号对应的后括号也同样如此缩进**并且应当只在这时候才另起一行，除非你能让它上下对齐**；返回值部分的定义，可以另起一行，如果另起一行，就要相对 `lambda` 多往后缩进一次。
> - 除了包裹 `lambda` 定义的小括号在内容另起一行的时候以外，一切括号，应当想办法确保要么横着是一对要么竖着是一对。如果新增新的格式化规则，也是在确保这一条的前提下进行的，或者就是为了更好确保这一条而进行。
> - `lambda` 定义被调用的话，为了好看，最好确保包裹定义的后括号和参数列表之间有所间隔；但为了方便你调试，所以最好别间隔到下一行，只用空格间隔就好拉。
> - 参数列表内容如果单独换行，向后一次缩进，尾括号可以跟前面的逗号竖着对齐而不是跟对应的头括号。（这应该是唯一的允许上下一对括号不对其的地方。遵照这个原则，也可以做到：一看到不对齐的括号，就知道它前面是参数列表。）（以及，参数列表尽量别折行。这是为了 *一对括号竖或横对齐* 的原则。）
> - `lambda` 定义被赋值的时候可以不被括号包裹。如果被包裹，则按照参数列表的规则来，即向后一层。因为前括号不能另起一行。。。
> - 多个换行了的参数列表的尾括号连续结尾的时候，应该按照层级程度叠加空格。
> - 关于逗号，参数列表里每个逗号前面贴着后面有一个空格、元组中逗号前后都有空格就像操作符一样。调用时，参数列表务必逗号后有一个空格，别的则随意。换行的话就都是逗号前面有个空格，我们按照括号区分是不是参数列表。
> 

现在也可以轻易让结果跟上面那**列表**更像：

```python
[   (
        '商家{}'.format(i+1) ,
        (lambda rndlow,add: 
            (rndlow,rndlow+add)
        ) (随机.randint(40,300), 随机.randint(10,600))
    )
    for i in range(13)
]

```

格式换后看就很容易看了：只是 `for` 前面的内容进入了一对圆括号（这对是起到建立元组的作用），然后在 
`[0]` 位置加上了一个字符串值。当然这个值这里就用到了来自 `range(13)` 的元素 `i` 的内容。

那我想自己指定这个数量 `13` 呢：

```python
(lambda cnt: 
    [   (
            '商家{}'.format(i+1) ,
            (lambda rndlow,add: 
                (rndlow,rndlow+add)
            ) (随机.randint(40,300), 随机.randint(10,600))
        )
        for i in range(cnt)
    ]
) (13)

```

行了，现在它返回的是个列表，列表的**一个元素**的内容可能是这样： `('商家3', (240, 713))` 。

现在我们想形成这样一个列表，它的每个元素都是类似这样 `('商家3', [420, 285, 495, 539, 269, 296, 293])` 的：

```python
[   (
        sj[0], 
        (lambda cnt: 
            lambda low,high: 
                [随机.randint(low,high) for i in range(cnt)]
        )(7)(sj[1][0],sj[1][1]) 
    )
    
    for sj in 
        (lambda cnt: 
            [   (
                    '商家{}'.format(i+1) ,
                    (lambda rndlow,add: 
                        (rndlow,rndlow+add)
                    ) (随机.randint(40,300), 随机.randint(10,600))
                )
                for i in range(cnt)
            ]
        )(13)
] 

```

其中：

- 第一个 `[` 后面紧跟着新生列表的每个元素的计算逻辑，这个计算的变量依据就是 `for` 和 `in` 之间的那串玩意，这个玩意（在这里就是叫 `sj` 的那个玩意）的内容来自于后面的集合的每一项，当然，这个集合是依据一个输入数值才能生成的。
- 而关于在新生列表的每元素的逻辑，首先，它是个元组，第一项是商家号，而它**在此**刚好能根据 `sj[0]` 取得；后面需要有一个 `7` 长度的随机数列表，生成它的 `lambda` 定义前面已经定义过了，需要的参数则可以根据 `sj[1]` 这个元组取到，具体就是分别取 `sj[1][0]` 和 `sj[1][1]` ，填写在传入参数需要写在的那个正确位置，然后函数就能根据**参数的指定**生成**根据参数与函数定义而形成的随机数列表**了。
- 这样一来，只要 `sj` 一直能被确定是什么内容，而且这个内容的结构式以及结构各个位置的数据的类型也都是合乎要求的，那么最终结果就能出来了；好巧不巧，这两点我们的数据源都能达到。🙃🙃🙃



如此一来，类似于 `('商家3', [420, 285, 495, 539, 269, 296, 293])` 这样的一堆数据我们就能够生成了。

它有啥用？它的两项分别一填不就拼出代码来了吗？

```python
[   '    .add_yaxis("{}",{})\n'.format(sj_msg[0], sj_msg[1])

    for sj_msg in 
    [   (
            sj[0], 
            (lambda cnt: 
                lambda low,high: 
                    [随机.randint(low,high) for i in range(cnt)]
            )(7)(sj[1][0],sj[1][1]) 
        )
        
        for sj in 
            (lambda cnt: 
                [   (
                        '商家{}'.format(i+1) ,
                        (lambda rndlow,add: 
                            (rndlow,rndlow+add)
                        ) (随机.randint(40,300), 随机.randint(10,600))
                    )
                    for i in range(cnt)
                ]
            )(13)
    ] 
]

```

当然，这还不够，我们需要的是一个字符串。

下面又要引入一个东西： `reduce` 。  


不过，这里我想先多说一句（万一后面用到了呢）：

> **其实也不用引入，单纯用 `lambda` 我能做到手动实现一个差不多的东西！**  
> 

看：

```python
## 定义
(lambda selfff: selfff(selfff)) ( lambda selff: lambda op, list, res: res if (list == []) else (selff(selff))(op,list[1::],op(res,list[0])) ) 

## 定义并调用
(lambda selfff: selfff(selfff)) ( lambda selff: lambda op, list, res: res if (list == []) else (selff(selff))(op,list[1::],op(res,list[0])) ) (lambda x,y:x-y, [5,4,3,2,1], 6)
## 返回就像这个的结果一样：
6-5-4-3-2-1 # 返回: -9


### 把定义并调用格式化一下：


## 这是非通用的 Y 组合子 用来让匿名函数可以自己调用自己
(lambda selfff: selfff(selfff)) (
    lambda selff: 
        lambda op, list, res: 
            
            ## 这下面是退出逻辑
            res if (list == []) 
            else (selff(selff))(
                
                ## 这里是递归 下面是新的参数 上面是生成一个自己然后才能递归才能传新参数
                op, list[1::], op(res, list[0]) )  ) (
    
    ## 这个是这次定义在调用时候的参数列表: 传入逻辑是前减去后结果再减去后、列表是第二项、初始值是第三项。
    lambda x,y:x-y , 
    [5,4,3,2,1] , 
    6 )


### 哎，这有人可能会说了，说：你这不是多此一举了嘛，直接写算式不就行了嘛！
### 这我就要说了，这里我传入的列表是写好了的，如果是依据变量动态生成的列表呢？
### （是不是有点动态生成代码那味儿了？只不过这里不是加数字而是接字符串，当真就是在动态生成代码了。。。）
### 🙃🙃🙂🙈🦦🦀

```

上面的部分不需要完全掌握，主要还是自己执行看看结果就好（可以把最下面的参数随便换一换试试看）。  
**这部分我只是想说明一下，聚合函数是可以只通过 `lambda` 演算来定义的，并给出示例。。。万一后面能用到就更好了。🙃**

下面还是用引入的 `reduce` 。

引入的话，老惯例，换个名儿。

```python
from functools import reduce as 汇总

```

**请注意！要有这个引入，然后才能执行下面的代码！！**

先使用一下这个函数看看效果：

```python
汇总 (
    lambda x,y:x+y ,
    [   '    .add_yaxis("{}",{})\n'.format(sj_msg[0], sj_msg[1])

        for sj_msg in 
        [   (
                sj[0], 
                (lambda cnt: 
                    lambda low,high: 
                        [随机.randint(low,high) for i in range(cnt)]
                )(7)(sj[1][0], sj[1][1]) 
            )
            
            for sj in 
                (lambda cnt: 
                    [   (
                            '商家{}'.format(i+1) ,
                            (lambda rndlow,add: 
                                (rndlow,rndlow+add)
                            ) (随机.randint(40,300), 随机.randint(10,600))
                        )
                        for i in range(cnt)
                    ]
                )(13)
        ] 
    ] )

```

这里的返回值不会有转义，不过没事，紧接着会有效果更直观的预览。这里只需要确认是不是成功返回一个加总好的字符串就行了。

更好的预览方案：

```python
x = lambda : 汇总 (
    lambda x,y:x+y ,
    [   '    .add_yaxis("{}",{})\n'.format(sj_msg[0], sj_msg[1])

        for sj_msg in 
        [   (
                sj[0], 
                (lambda cnt: 
                    lambda low,high: 
                        [随机.randint(low,high) for i in range(cnt)]
                )(7)(sj[1][0], sj[1][1]) 
            )
            
            for sj in 
                (lambda cnt: 
                    [   (
                            '商家{}'.format(i+1) ,
                            (lambda rndlow,add: 
                                (rndlow,rndlow+add)
                            ) (随机.randint(40,300), 随机.randint(10,600))
                        )
                        for i in range(cnt)
                    ]
                )(13)
        ] 
    ] )
print(x())
print(x())
print(x())

```

这里的 `print(x())` 可以多执行几次。你会看到数在变，因为每次都是重新从头来一遍这个过程。

这里赋值给 `x` 只是为了方便下面的 `print` 函数。毕竟 Python 的 REPL 按上的话并不是逻辑意义上的一行都会出来，并不会像 Zsh 或者哪怕是 Bash 那样。

现在，赋值给 `x` 的最外层的 `lambda` 定义是无参的。要不要有一下呢？哎，这个商家数量我想在这控制！

那么就把里面的相应的抽象挪到外面来。整个定义就变成了这样：

```python
lambda cnt : 汇总 (
    lambda x,y:x+y ,
    [   '    .add_yaxis("{}",{})\n'.format(sj_msg[0], sj_msg[1])

        for sj_msg in 
        [   (
                sj[0], 
                (lambda cnt: 
                    lambda low,high: 
                        [随机.randint(low,high) for i in range(cnt)]
                )(7)(sj[1][0], sj[1][1]) 
            )
            
            for sj in 
                [   (
                        '商家{}'.format(i+1) ,
                        (lambda rndlow,add: 
                            (rndlow,rndlow+add)
                        ) (随机.randint(40,300), 随机.randint(10,600))
                    )
                    for i in range(cnt)
                ]
        ] 
    ] )
```

调用：

```python
(lambda cnt : 汇总 (
    lambda x,y:x+y ,
    [   '    .add_yaxis("{}",{})\n'.format(sj_msg[0], sj_msg[1])

        for sj_msg in 
        [   (
                sj[0], 
                (lambda cnt: 
                    lambda low,high: 
                        [随机.randint(low,high) for i in range(cnt)]
                )(7)(sj[1][0], sj[1][1]) 
            )
            
            for sj in 
                [   (
                        '商家{}'.format(i+1) ,
                        (lambda rndlow,add: 
                            (rndlow,rndlow+add)
                        ) (随机.randint(40,300), 随机.randint(10,600))
                    )
                    for i in range(cnt)
                ]
        ] 
    ] )
) (4)

```

看看结果，没问题。

现在可以愉快玩耍了：

```python
x = lambda cnt : 汇总 (
    lambda x,y:x+y ,
    [   '    .add_yaxis("{}",{})\n'.format(sj_msg[0], sj_msg[1])

        for sj_msg in 
        [   (
                sj[0], 
                (lambda cnt: 
                    lambda low,high: 
                        [随机.randint(low,high) for i in range(cnt)]
                )(7)(sj[1][0], sj[1][1]) 
            )
            
            for sj in 
                [   (
                        '商家{}'.format(i+1) ,
                        (lambda rndlow,add: 
                            (rndlow,rndlow+add)
                        ) (随机.randint(40,300), 随机.randint(10,600))
                    )
                    for i in range(cnt)
                ]
        ] 
    ] )
print(x(1))
print(x(2))
print(x(3))
print(x(0))

```

。。。不能是 `0` ，因为 `reduce` 传入的列表不能为空（因为 `reduce` 一般的逻辑就是把第一个做初始值然后把剩下的列表和初始值还有计算规则都传入另一个可以定义初始值的聚合函数。）

刚好，试试我前面自制的那个：

```python
x = lambda cnt : (lambda selfff: selfff(selfff)) (
    lambda selff: 
        lambda op, list, res: 
            
            res if (list == []) 
            else (selff(selff))(
                
                op, list[1::], op(res, list[0]) )  ) (
    lambda x,y:x+y ,
    [   '    .add_yaxis("{}",{})\n'.format(sj_msg[0], sj_msg[1])

        for sj_msg in 
        [   (
                sj[0], 
                (lambda cnt: 
                    lambda low,high: 
                        [随机.randint(low,high) for i in range(cnt)]
                )(7)(sj[1][0], sj[1][1]) 
            )
            
            for sj in 
                [   (
                        '商家{}'.format(i+1) ,
                        (lambda rndlow,add: 
                            (rndlow , rndlow + add)
                        ) (随机.randint(40,300), 随机.randint(10,600))
                    )
                    for i in range(cnt)
                ]
        ] 
    ] , 
    '' )
print(x(1))
print(x(2))
print(x(3))
print(x(0))

```

如果是 `0` 返回就是空。为啥？你看我给定的那第三个参数不就是空字串嘛！！

🤣🤣🤣🤣（嘿嘿嘿嘿嘿嘿嘿嘿…………）

（不过这没啥大不了的，我只是没能实现封装好的 `reduce` 罢了。。。）

现在再改改，就能拼接完整的代码了。我直接上结果：

```python
xforexec = (
    lambda cnt : 
    """
from pyecharts.charts import Bar
from pyecharts import options as opts

(
    Bar(init_opts=opts.InitOpts(width="680px",height="390px",page_title="HAHAHAHA"))
    .add_xaxis(["衬衫", "毛衣", "领带", "裤子", "风衣", "高跟鞋", "袜子"])
{}    .set_global_opts(title_opts=opts.TitleOpts(title="某商场销售情况"))
).render("shangjias.html")
    """.format(
        
        ## (-> (a) (a a))
        (lambda selfff: selfff(selfff)) (
            
            ## reduce
            lambda selff: 
                lambda op, list, res: 
                    
                    res if (list == []) 
                    else (selff(selff))(
                        
                        op, list[1::], op(res, list[0]) )  ) (
            
            
            ## str adds
            lambda x,y:x+y , 
            
            ## str list (need to know val of cnt)
            [   '    .add_yaxis("{}",{})\n'.format(sj_msg[0], sj_msg[1])

                for sj_msg in 
                [   (
                        sj[0], 
                        (lambda cnt: 
                            lambda low,high: 
                                [随机.randint(low,high) for i in range(cnt)]
                        )(7)(sj[1][0], sj[1][1]) 
                    )
                    
                    for sj in 
                        [   (
                                '商家{}'.format(i+1) ,
                                (lambda rndlow,add: 
                                    (rndlow,rndlow+add)
                                ) (随机.randint(40,300), 随机.randint(10,600))
                            )
                            for i in range(cnt)
                        ]
                ] 
            ] , 
            
            ## first of adding
            '' )  )   )
exec(xforexec(3))

```

为什么不用 `eval` ？因为这个东西最后没有返回，而是副作用生成一个文件。

其实这里去掉字符串里的俩 `import` 的话，就也能用 `eval` 了，不过它毕竟不是函数而是副作用命令集合（虽说生效的只有一条）。

想要看看这个文件的话，可以**退出 Python REPL** 然后这样：

```bash
python -m http.server 2341

```

然后访问 `节点IP:2341/shangjias.html` 就能看到结果了。

这是个 HTTP 代理，一般用于调试。生产中建议使用 Httpd 或者 Nginx 来做这个事。

## 备忘

本文一共有两个 `import` ：

```python
import random as 随机
from functools import reduce as 汇总

```

----

`分享注明来源: https://segmentfault.com/a/1190000040317072`  
