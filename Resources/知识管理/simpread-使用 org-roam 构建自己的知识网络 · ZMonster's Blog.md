> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zmonster.me](https://www.zmonster.me/2020/06/27/org-roam-introduction.html)

> 2020-06-27

2020-06-27

前言
--

最近 [Roam Research](https://roamresearch.com/) 一类的以网状结构来关联笔记、并以 backlink 的形式来展现笔记上下文的工具非常热门。所谓网状结构，是认为知识和知识是互相关联的，并通过这种互联形成复杂的网络，就像我们的大脑一样；所谓 backlink，是指对单条笔记，展示出链接到这条笔记的其他笔记，这样有助于更好地理解这条笔记的意义。本质上，网状结构和 backlink 其实是一回事，说的都是知识之间的互相链接，不过网状结构着眼于整体结构，而 backlink 则呈现局部形态。

Roam Research 这类工具中的理念，叫做卡片盒笔记法，本文无意对这一想法做过多介绍，如果想进一步了解，可以参考下列文章：

*   [Roam Research 到底好在哪儿？ - 少数派](https://sspai.com/post/60787)
*   [从卡片链接到大脑联想，基于 Obsidian 的卡片盒笔记法实践 - 少数派](https://sspai.com/post/60802)
*   [真正的思考技术：来自德国社会学 Niklas Luhmann 的 Zettelkasten 方法](https://mp.weixin.qq.com/s?__biz=MzI3NDEzMjIyMQ==&mid=2649474090&idx=1&sn=bf2fcc5c3a909e939795eb36b1ec33ae&chksm=f307d9b8c47050ae1f88ed976deef2e7e51b44fd4eac768b5541743a9876cc1a505df11a3a71#rd)

在 Emacs 中，很早就有一个工具 [org-brain](https://github.com/Kungsgeten/org-brain)，想要以 org-mode 为基础让人能建立自己的知识网络，本质上思想是类似的，但交互并不算特别友好，所以我以前稍微用了下就没有继续下去了，而最近出现的 [Org-roam](https://www.orgroam.com/) 则对标 Roam Research，实现了非常友好的交互，并提供了 [org-roam-server](https://github.com/org-roam/org-roam-server) 这样非常棒的知识网络可视化界面，经过短时间的使用后，我推荐所有使用 org-mode 来记录自己笔记的人都用一下 org-roam，理由如下：

1.  org-mode 本来就提供了极其强大的链接能力，可以链接到文件、headline 甚至文件的随便一行，也支持了对大量不同类型的外部链接，而 org-roam 为这种能力提供了友好而高效的交互操作
2.  org-roam 复用了 org-capture 的强大功能，使得我们可以自定义各种笔记模板来更好地表示、呈现知识
3.  org-roam-server 提供的笔记网络可视化界面和 org-roam 深度集成，点击界面上的笔记节点就能在 Emacs 中打开对应的笔记

环境说明
----

*   操作系统： Ubuntu 16.04
*   Emacs 版本： GNU Emacs 26.1
*   org-mode 版本： 9.3.7
*   org-roam 版本： 开发版 20200615
*   org-roam-server 版本： 开发版 20200621
*   浏览器： Firefox/Chrome
*   GIF 录制工具： [byzanz-record](https://linux.die.net/man/1/byzanz-record)

安装及初步配置
-------

直接从 [MELPA](https://melpa.org/) 安装即可

```
(package-install 'org-roam)
(package-install 'org-roam-server)
```

安装完成后，首先需要设置 org-roam-directory 指向一个目录，用来存放使用 org-roam 创建的笔记。我把笔记都放在 Dropbox 里，所以设置如下

```
(setq org-roam-directory "~/Dropbox/org/roam")
```

然后让 org-roam 在 Emacs 启动后就启用

```
(add-hook 'after-init-hook 'org-roam-mode)
```

然后设置并启动 org-roam-server 来监听笔记的变化并进行可视化（见 [org-roam-server 安装说明](https://github.com/org-roam/org-roam-server#installation)）

```
(setq org-roam-server-host "127.0.0.1"
      org-roam-server-port 9090
      org-roam-server-export-inline-images t
      org-roam-server-authenticate nil
      org-roam-server-network-label-truncate t
      org-roam-server-network-label-truncate-length 60
      org-roam-server-network-label-wrap-length 20)
(org-roam-server-mode)
```

上面的配置生效后，会在本地启动一个网页服务，访问 [http://127.0.0.1:9090](http://127.0.0.1:9090/) ，会看到下面这样的界面：

![](https://www.zmonster.me/assets/img/org-roam-server-web.png)

由于刚开始并没有创建笔记，上面只会显示一片空白。

然后启用 org-roam-protocol，用来在笔记可视化网页上和 org-roam-server 通信

```
(require 'org-roam-protocol)
```

这个 org-roam-protocol 是使用 org-protocol 实现的，依赖操作系统的相关功能，相关设置参考[文档](https://www.orgroam.com/manual/Installation-_00281_0029.html)。

完成上述设置后，就可以开始体验 org-roam 了，执行 M-x org-roam-find-file 创建一条新的笔记，然后刷新笔记可视化页面，就能看到页面上多了一个新的节点了，如下图所示：

![](https://www.zmonster.me/assets/img/org-roam-new.gif)

org-roam 的基本使用
--------------

首先来看下 org-roam 的基本功能

<table><colgroup><col> <col> <col> </colgroup><thead><tr><th scope="col">函数</th><th scope="col">功能</th><th scope="col">备注</th></tr></thead><tbody><tr><td>org-roam-find-file</td><td>打开或新建笔记</td><td>&nbsp;</td></tr><tr><td>org-roam-capture</td><td>新建笔记</td><td>&nbsp;</td></tr><tr><td>org-roam-insert</td><td>插入一个指向其他笔记的链接，如果不存在会新建一个笔记</td><td>&nbsp;</td></tr><tr><td>org-roam-insert-immediate</td><td>类似 org-roam-insert，但新建笔记后不打开这个笔记</td><td>需要 org-roam 1.2.1</td></tr><tr><td>org-roam</td><td>显示 backlink</td><td>&nbsp;</td></tr></tbody></table>

核心的功能就这么多，没有太多概念、操作要学习，这也是我推荐大家使用它的原因。其基本工作流也很简单，下面是一个示例：

1.  打开已有笔记，或新建笔记
    *   使用 org-roam-find-file 来新建一个笔记
        
        ![](https://www.zmonster.me/assets/img/org-roam-new.gif)
        
    *   或者，使用 org-roam-find-file 来打开已有的笔记
        
        ![](https://www.zmonster.me/assets/img/org-roam-open-note.gif)
        
    *   或者，在笔记可视化网页上浏览，点击想要查看或编辑的笔记节点，在 Emacs 中打开这个笔记
        
        ![](https://www.zmonster.me/assets/img/org-roam-open-note-2.gif)
        
2.  选中笔记内容中的某些关键词，使用 org-roam-insert-immediate，创建新的笔记并链接过去，并继续编辑当前的笔记
    
    ![](https://www.zmonster.me/assets/img/org-roam-insert-immediate.gif)
    
    从图上右侧可以看到产生了一个名为 org-mode 的新节点，并和 Emacs 这个节点关联起来了。
    
3.  或者，选中笔记内容中的关键词，使用 org-roam-insert，创建新的笔记并链接过去，同时打开新的笔记进行编辑
    
    ![](https://www.zmonster.me/assets/img/org-roam-insert-new.gif)
    
    从图上右侧可以看到产生了一个名为 calc 的新节点，并和 Emacs 这个节点关联起来了。
    
4.  或者，用 org-roam-insert-immediate/org-roam-insert 插入一个指向已有笔记的链接
    
    ![](https://www.zmonster.me/assets/img/org-roam-link-to.gif)
    
    上图和步骤 3 一样执行的是 org-roam-insert，但从图上右侧可以看到，只是已有的两个节点之间产生了一条关联，并没有新的节点产生。
    
5.  使用 org-roam 展示笔记的 backlinks
    
    ![](https://www.zmonster.me/assets/img/org-roam-show-backlinks.gif)
    
6.  用 org-roam-capture 在已有笔记中新增内容
    
    ![](https://www.zmonster.me/assets/img/org-roam-append.gif)
    
    org-roam-capture 也可以用于新建笔记，实际上 org-roam-find-file 的逻辑就是：先检查笔记文件是否存在，如果存在就打开，否则就调用 org-roam-capture 来新建笔记。但 org-roam-capture 除了用于新建笔记文件，还可以快捷地在已有笔记中新增内容，且新增内容时可以利用模板来提高效率，比用 org-roam-find-file 打开笔记文件再手工新增会更高效一些。
    
7.  重复上述过程

掌握上述工作流后，剩下的事情就是把自己所学到的东西用 org-roam 来进行记录、整理了。

org-roam 进阶
-----------

### 定制笔记模板

用 **org-roam-find-file/org-roam-capture** 新建笔记的时候，会要求我们输入笔记标题，假如我们输入的笔记标题是 "org-roam"，那么会在新建这个笔记后发现这个笔记只在第一行把我们输入的标题写上去了，别的什么都没有：

在实际使用中，我们可能会有不同的笔记需求，比如说：当我为一个专业术语记录笔记时，我想写下这个术语所属的领域以及它的含义；当我记录一个观点时，我会想写上这个观点是谁提出来的、论据是什么、我自己是支持还是反对；当我读一篇深度学习的论文时，我要记录这篇论文的相关工作、要解决的问题、使用了什么方法、进行了怎么样的实验……

这些需求用 org-roam 是能够满足的，因为 org-roam 通过 org-roam-capture-templates 这个变量提供了定制笔记模板的能力。

具体来说，默认的笔记模板是这样的

```
'(("d" "default" plain (function org-roam-capture--get-point)
   "%?"
   :file-name "%<%Y%m%d%H%M%S>-${slug}"
   :head "#+title: ${title}\n"
   :unnarrowed t))
```

每个模板都由 8 个部分组成，我这里以上面的默认模板来进行说明

<table><colgroup><col> <col> <col> </colgroup><thead><tr><th scope="col">模板组成</th><th scope="col">对应默认模板中的内容</th><th scope="col">描述</th></tr></thead><tbody><tr><td>key</td><td>"d"</td><td>用来选择模板的快捷键</td></tr><tr><td>description</td><td>"default"</td><td>展示用的模板描述</td></tr><tr><td>type</td><td>plain</td><td>新增内容的类型</td></tr><tr><td>target</td><td>(function org-roam-capture–get-point)</td><td>新增内容的位置， <b>不可更改</b></td></tr><tr><td>template</td><td>"%?"</td><td>新增内容的模板</td></tr><tr><td>file-name</td><td>:file-name "%&lt;%Y%m%d%H%M%S&gt;-${slug}"</td><td>新增笔记文件的文件名模板</td></tr><tr><td>head</td><td>:head "#+title: ${title}\n"</td><td>新增笔记的初始化内容，仅新建时生效</td></tr><tr><td>properties</td><td>:unnarrowed t</td><td>新增笔记的其他属性</td></tr></tbody></table>

下面对这 8 个模板元素分别说明一下

*   用来选择模板的 key：
    
    对应默认模板里的 "d"，一个字符的情况下用来直接选择模板，两个字符的情况下用第一个字符表示模板分组、第二个字符用来选择这个分组下的实际模板。
    
    下面的配置设置了四个模板，其中第二个 ("g" "group") 用来指明一个模板分组，后面的 "ga" 和 "gb" 是这个模板下的子模板。
    
    ```
    (setq org-roam-capture-templates
          '(
            ("d" "default" plain (function org-roam-capture--get-point)
             "%?"
             :file-name "%<%Y%m%d%H%M%S>-${slug}"
             :head "#+title: ${title}\n#+roam_alias:\n\n")
            ("g" "group")
            ("ga" "Group A" plain (function org-roam-capture--get-point)
             "%?"
             :file-name "%<%Y%m%d%H%M%S>-${slug}"
             :head "#+title: ${title}\n#+roam_alias:\n\n")
            ("gb" "Group B" plain (function org-roam-capture--get-point)
             "%?"
             :file-name "%<%Y%m%d%H%M%S>-${slug}"
             :head "#+title: ${title}\n#+roam_alias:\n\n")))
    ```
    
    上面的模板生效后，首先在执行 org-roam-find-file 新建笔记时，就会看到一个选择界面，如下图所示：
    
    ![](https://www.zmonster.me/assets/img/org-roam-capture-select-template.gif)
    
    如果输入 d 会直接打开新建笔记的编辑窗口，如果输入 g 会展开分组模板要求我们再输入一次来选择具体的模板，如下图所示：
    
    ![](https://www.zmonster.me/assets/img/org-roam-capture-select-group-template.gif)
    
*   用来描述模板的 description：这个元素就是起到单纯的描述作用，没有功能上的意义
*   用来说明新增内容类型的 type：本来有 plain/entry/item/checkitem/table-line 五种取值，但在 org-roam 中作用都是一样的，建议一律使用 plain
*   用来说明新增内容位置的 target：这一项在 org-roam 中不可更改
*   设置新增内容模板的 template：
    
    这个元素是整个模板中的核心，其中的内容可以分为两类：
    
    *   普通的文本，将会原样出现在新增内容中
    *   以 % 开头的特殊标记，如默认模板中的 "%?"，将会在最后根据类型自动扩展成不同的内容
    
    对于第一类内容没啥可说的，唯一值得一提的是，如果需要模板是多行的文本，需要在模板中用 "\n" 来指明要换行，如模板内容 "第一行 \ n 第二行" 最后就会在笔记中显示为：
    
    这里说一下以 % 开头的特殊标记，由于这块内容很多，这里只列举一些常用的供读者参考：
    
    <table><colgroup><col> <col> </colgroup><thead><tr><th scope="col">标记</th><th scope="col">描述</th></tr></thead><tbody><tr><td>%&lt;…&gt;</td><td>自定义格式的时间戳，如: %&lt;%Y-%m-%d&gt;，会得到 &lt;2018-03-04 日&gt;</td></tr><tr><td>%t</td><td>当前日期，展开后的格式固定为 &lt;2018-03-04 日&gt; 这样</td></tr><tr><td>%T</td><td>当前日期和时间，展开后的格式固定为 &lt;2018-03-04 日 19:26&gt; 这样</td></tr><tr><td>%u</td><td>当前日期，展开后的格式固定为 [2018-03-04 日] 这样</td></tr><tr><td>%U</td><td>当前日期和时间，展开后的格式固定为 [2018-03-04 日 19:26] 这样</td></tr><tr><td>%<sup>prompt</sup></td><td>用 prompt 作为提示要求我们输入并填充在这个模板元素所在的位置</td></tr><tr><td>%?</td><td>其他所有特殊标记填充完毕后，光标将停留在这个元素的位置等待我们编辑</td></tr></tbody></table>
*   指定新增笔记文件名的 file-name：
    
    org-roam 中新建笔记一般都是以一个新文件的形式来创建的，支持用 file-name 来设置这个新文件的文件名，默认模板中的这块设置为 "%<%Y%m%d%H%M%S>-${slug}"，分为两部分
    
    *   **%<%Y%m%d%H%M%S>** ：参考上一节 template 部分的特殊标记
    *   **${slug}** ：将笔记标题文字做处理后得到的文本，这些处理包括大写字母转小写、去除一些特殊字符等
    
    这块建议使用默认模板就好。
    
*   设置新增笔记初始内容的 head：
    
    这个设置用来在新建笔记文件时设置初始内容，只会执行一次，也就是说之后如果使用 org-roam-capture 新增内容到已有笔记中时，这个设置的内容是不会再写入到文件中的。
    
    默认模板这块的内容是 "#+title: ${title}\n"，只设置了笔记文件的标题，建议改为如下内容：
    
    ```
    :head "#+title: ${title}\n#+roam_alias: \n#+roam_tags: \n"
    ```
    
    这样设置后新建笔记文件的初始内容将会是：
    
    ```
    #+title: 示例标题
    #+roam_alias:
    #+roam_tags:
    ```
    
    "#+roam_alias" 可以给这条笔记设置别名，这样在其他笔记中引用的时候会更方便；"#+roam_tags" 可以用来为这条笔记添加标签，使得在用 org-roam-find-file 查找已有笔记时能根据 tag 来进行过滤。
    
*   设置新增内容其他属性的 properties：
    
    这些属性用于对新建笔记内容的行为做一些额外的控制，列举几个常用的：
    
    *   **:unnarrowed t**: org-roam 推荐的设置，表示现实整个笔记文件，如果不加这个设置，用 org-roam-capture 增加内容到已有笔记文件中时，仅会显示当前我们输入的内容，而不会显示这个笔记文件中已有的内容
    *   **:empty-lines 1**: 在新增的笔记内容前后加一个空行，使用 org-roam-capture 增加内容到已有笔记文件中时比较有用

org-roam 的笔记模板是利用 [org-capture](https://orgmode.org/manual/Capture.html) 实现的，上述模板元素中 file-name 和 head 是 org-roam 在 org-capture 模板的基础上增加的新元素；其他六个部分，target 在 org-roam 中不可更改，key、description、type 和 template 的更详细说明可以参考我之前写的一篇介绍 org-capture 文章中的相关内容，[capture 模板的五个部分](https://www.zmonster.me/2018/02/28/org-mode-capture.html#org1a3d856)。

为了能更直观地理解模板的工作机制，这里给几个模板的示例：

*   用于记录专业术语的模板
    
    ```
    (add-to-list 'org-roam-capture-templates
                 '("t" "Term" plain (function org-roam-capture--get-point)
                   "- 领域: %^{术语所属领域}\n- 释义:"
                   :file-name "%<%Y%m%d%H%M%S>-${slug}"
                   :head "#+title: ${title}\n#+roam_alias:\n#+roam_tags: \n\n"
                   :unnarrowed t
                   ))
    ```
    
    将上面的配置拷贝到你的 Emacs 配置中，并置于所有 org-roam 相关配置的后面，就可以在你的 org-roam 中使用这个模板，后面的示例模板也是一样，但要注意不同模板的 key 不要有冲突。
    
    ![](https://www.zmonster.me/assets/img/org-roam-new-term.gif)
    
*   用于记录论文笔记的模板
    
    ```
    (add-to-list 'org-roam-capture-templates
                 '("p" "Paper Note" plain (function org-roam-capture--get-point)
                   "* 相关工作\n\n%?\n* 观点\n\n* 模型和方法\n\n* 实验\n\n* 结论\n"
                   :file-name "%<%Y%m%d%H%M%S>-${slug}"
                   :head "#+title: ${title}\n#+roam_alias:\n#+roam_tags: \n\n"
                   :unnarrowed t
                   ))
    ```
    
    ![](https://www.zmonster.me/assets/img/org-roam-new-paper-note.gif)
    

另外，org-roam-insert-immediate 不使用 org-roam-capture-templates，而是使用一个专门的 org-roam-capture-immediate-template 来设置新建内容的模板，且只能有一个模板，所以设置这个模板的配置是这样的（以默认配置为例）

```
(setq org-roam-capture-immediate-template
      '("d" "default" plain (function org-roam-capture--get-point)
        "%?"
        :file-name "%<%Y%m%d%H%M%S>-${slug}"
        :head "#+title: ${title}\n"
        :unnarrowed t))
```

### 实现网页内容摘录

这部分内容需要 org-protocol，后续内容是在 org-protocol 已经设置好的基础上展开的，如果 org-protocol 设置存在问题，请查阅[文档](https://www.orgroam.com/manual/Installation-_00281_0029.html)，或这评论区留言来讨论。

利用 org-protocol 这样的外部程序和 Emacs 进行通信的机制，我们可以使用 javascript 来抓取网页上的信息发送到 Emacs 中，而 org-roam 也支持了这种机制。在 org-roam 中可以通过 org-roam-capture-ref-templates 来设置网页捕获相关的模板，默认的设置是这样的：

```
(setq org-roam-capture-ref-templates
      '(("r" "ref" plain (function org-roam-capture--get-point)
         ""
         :file-name "${slug}"
         :head "#+title: ${title}\n#+roam_key: ${ref}\n"
         :unnarrowed t)))
```

可以看到，模板本身和前面的笔记模板是一样的，没有什么特别。但我们可以创建一个[小书签](https://zh.wikipedia.org/zh-cn/%E5%B0%8F%E4%B9%A6%E7%AD%BE)，来利用这个模板，抓取网页标题和链接然后新建一个笔记到 org-roam 中，如下图所示：

![](https://www.zmonster.me/assets/img/org-roam-store-link.gif)

上图中小书签的内容来自 org-roam 的[文档](https://www.orgroam.com/manual/The-roam_002dref-Protocol.html#The-roam_002dref-Protocol)，具体内容为：

```
javascript:location.href = 'org-protocol://roam-ref?template=r&ref=' + encodeURIComponent(location.href) + '&title=' + encodeURIComponent(document.title)
```

添加的方法是在浏览器中新增一个书签，书签的名字随意（上图中我设置为了 “网页抓取”），书签的 URL 填上上面的 javascript 代码。下图是 Firefox 中创建这样的小书签的示意图：

![](https://www.zmonster.me/assets/img/create-bookmarklet-in-firefox.gif)

这个小书签的内容分成几部分：

*   第一部分说明小书签要访问的地址，这个就是 org-roam-protocol 的通信地址
    
    ```
    javascript:location.href='org-protocol://roam-ref'
    ```
    
*   第二部分指定要使用的笔记模板，从 org-roam-capture-ref-templates 中匹配
    
*   第三部分获取一些网页的信息，并设置到变量中，供模板填充使用
    
    ```
    '&ref=' + encodeURIComponent(location.href) + '&title=' + encodeURIComponent(document.title)
    ```
    
    前面的默认模板中，head 部分内容为
    
    ```
    :head "#+title: ${title}\n#+roam_key: ${ref}\n"
    ```
    
    需要填充 "title" 和 "ref" 两个变量，小书签中第三部分内容就是获取了当前网页的链接赋值给 "ref" 变量，并获取网页标题文本赋值给 "title" 变量了，这样这个模板就能自动填充好了。
    

不过这个模板和小书签过于简单，只能记录网页链接，我设计了一个模板和对应的小书签，可以做到进行网页标注、摘录，效果见下图：

![](https://www.zmonster.me/assets/img/org-roam-annotate-web.gif)

要达到上图的效果，首先，在 org-roam-capture-ref-templates 中新增一个模板

```
(add-to-list 'org-roam-capture-ref-templates
             '("a" "Annotation" plain (function org-roam-capture--get-point)
               "%U ${body}\n"
               :file-name "${slug}"
               :head "#+title: ${title}\n#+roam_key: ${ref}\n#+roam_alias:\n"
               :immediate-finish t
               :unnarrowed t))
```

然后新建一个小书签，内容为

```
javascript:location.href = 'org-protocol://roam-ref?template=a&ref=' + encodeURIComponent(location.href) + '&title='+encodeURIComponent(document.title) + '&body='+encodeURIComponent(function(){var html = "";var sel = window.getSelection();if (sel.rangeCount) {var container = document.createElement("div");for (var i = 0, len = sel.rangeCount; i < len; ++i) {container.appendChild(sel.getRangeAt(i).cloneContents());}html = container.innerHTML;}var dataDom = document.createElement('div');dataDom.innerHTML = html;['p', 'h1', 'h2', 'h3', 'h4'].forEach(function(tag, idx){dataDom.querySelectorAll(tag).forEach(function(item, index) {var content = item.innerHTML.trim();if (content.length > 0) {item.innerHTML = content + '
';}});});return dataDom.innerText.trim();}())
```