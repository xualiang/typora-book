**要求****：**

**剪辑**

**全文检索**

**纯md**

**git同步**

 

 

**Typora**

挺好的，输入即得

支持多级目录

支持mac、win、linux

支持全文检索

 

剪辑网站内容不方便，也无法直接拷贝粘贴

 

 

Boostnote

软件大

和quiver功能差不多，但不如quiver强大，不如quiver美观

也没有集成git同步

 

永久免费

支持win、mac、linux、ios、andrio

 

Vnote

不支持直接粘贴网站内容，放弃

 

**Quiver**

仅支持mac和IOS

存储不是md文件，但可以导出为md

没有git同步功能

收费、可破解

 

界面美观

支持全文检索

支持多级结构

支持markdown、text、code、lat 、diagram

软件比较小巧

 

bear

 

**Gitnote****（跟踪一下）**

赵鹏一个人还在开发，bug较多

同步github很方便

使用md文件保存

有作图插件（脑图、ER、流程图等）

 

ia writer

不好，保存后，文件名后缀居然是txt

yu writer

###### ################

用过的一些Markdown编辑器

开篇

买了MacBook Pro之后的一段时间里，为了打造适合自己的知识管理体系，折腾起了笔记类软件（题外话，我还挺喜欢尝试新软件的，尤其在接触macOS后发现许多软件都长得很漂亮）。其实在入手Mac之前，我已经试用过不少笔记类软件和服务了，包括Evernote（还有印象笔记）、有道云笔记、为知笔记，等等。再后来，改用Emacs的org-mode来写笔记——主要是将一些常常搜索的内容或经验记录在了多个.org文件中，算是一份自己的FAQ。后来想看看在macOS的世界中有没有更好的工具，同时渐渐觉得Markdown是一个更好的笔记内容载体，便尝试了一些知名的笔记类软件暨Markdown编辑器。

 

大致上尝试了下列这些：

 

Emacs

Boostnote

Quiver

Typora

Visual Studio Code

Yu Writer

本文并不是一篇完整的、专业的软件评测报告，只是我兴趣使然的对各个软件的吐槽和赞美，各位权当打发时间吧。下面我按顺序说一下上面提及的各款软件。

 

Emacs

Emacs并不仅仅是一款Markdown编辑器，我用得最多的是用它来做计划（之前还用来写Node.js代码，不过现在交给VSCode了）。用Emacs来写Markdown，坏处是没有live preview的功能。在Emacs中打开了一个.md文件，只会原原本本地显示着井号、星号，三个反引号等Markdown语法的关键字——并且还是白底黑字的模样，而不带有丝毫不同的样式。为了让它们好看点，你还需要安装一个叫做markdown-mode的Emacs扩展。但几遍安装了markdown-mode，也无法实时预览。markdown-mode的菜单栏中有一个叫做“Preview”的功能，它依赖一个名为markdown的命令行工具（用brew install markdown可以安装）。当一切安装完毕点击“Preview”菜单项时，才发现是在网页浏览器中查看的方式——虽然有preview了，但并不live。

 

Emacs在写Markdown方面也并非一无是处。对程序员而言，在一篇Markdown写就的文章中插入代码是再正常不过的事情了。在Emacs中将光标定位到Markdown语法的代码块内，按下control和c的组合键，再敲一下单引号键，Emacs便会另起一个相应模式的buffer，并将代码块中的内容复制到新buffer中供继续编辑。如下图所示

 

 

 

在上面的GIF中，代码块以GitHub Flavored Markdown的语法在开头的三个反引号后附上了模式的名字，即lisp，Emacs便会打开lisp-mode的buffer。在这个buffer中可以继续使用Emacs的完整功能编辑对代码，包括语法高亮、自动补全，等等——如果启动了SLIME，甚至可以运行里面的Common Lisp代码。

 

Emacs和VSCode用于在编写代码的同时写写项目的README.md文件应当是绰绰有余的了。

 

Boostnote

Boostnote自诩为“程序员的笔记本”，它并不是我在Emacs之外寻找的第一款笔记软件，在它之前，我还尝试了Notion、Quiver来着。上手后发现，Boostnote简直就是Quiver的开源免费版本，相当的喜爱。

 

Boostnote当然让我格外喜欢的有几点：首先，Boostnote可以实时预览键入的Markdown源文档。会有一列跟编辑区域差不多宽的区域被用来展示Markdown渲染后的效果。（刚刚发现，原来这个区域的宽度是可以拖动调节的）

 

其次，它不仅支持Markdown、带语法高亮的代码块，甚至还支持表格和流程图的绘制！当然我以为，用竖线和连字符绘制表格的功能仅在Emacs的org-mode中存在（孤陋寡闻了汗颜），刚开始用Boostnote制作表格的时候可是相当兴奋。而text-based的绘制流程图的方式也是让我大开眼界（后来才知道原来有flowchart.js这样的工具）——尽管后来我渐渐发现，绘制流程图其实挺少用。

 

然后Boostnote具备在多份笔记中搜索的功能，这对于一款笔记软件而言倒是真的非常重要，因为有时候只能想到一些只言片语，而并不能确定所要查阅的内容究竟在哪一份笔记中。

 

但Boostnote也有一些缺点。首先，Boostnote是用自有的文件格式（而不是纯文本的.md文件）来存储输入的内容的——打开~/Boostnote/notes/可以看到这些后缀为.cson的文件。这样一来，假设我日后发现了一款更优秀的Markdown编辑器，那就不能无痛迁移了，还得先从Boostnote中将这些笔记逐一导出成.md文件才行。

 

其次，Boostnote只支持三层的组织结构——最外层是storage，然后是folder，最后就是笔记本身。当初有道云笔记特别让我喜欢的，就是它支持非常多层级的目录结构。尽管目录不是越多越好，但有这种灵活性总是更好的。否则，笔记的使用者就只能在命名和标签上下功夫了

 

最后一点，就是Boostnote在我的系统上非常容易崩溃。有时候一翻起盖子，看到的就是Boostnote崩溃的提示。

 

不过Boostnote支持往其中粘贴图片，当我需要快速记录一些图文内容时，我还是很喜欢用它的。

 

Yu Writer

某一天偶然遇到了Yu Writer，它官网上的截图看着很吸引人，于是我便试用了一下。第一印象是，Yu Writer is awesome！首先它很人性化。它的预览区域是一个minimap——就是Sublime Text最右侧的那一列。在做到实时预览的时候，也不会占用太多的横向空间。其次，它支持大纲视图

 

 

 

即上图左侧的目录。恰逢当时我在用Boostnote写一篇比较长的设计文档，深刻地体验到了一个大纲视图的重要意义——对在长文档内的多个标题间跳转非常有帮助。再次，Yu Writer还准备了工具栏，方便不懂得Markdown语法的用户；支持标签页，便于在多个文档间切换；甚至可以把一个Markdown文档作为幻灯片来播放。

 

但Yu Writer也有它自己的劣势。第一，在Yu Writer内，原本在macOS系统中全局可用的Emacs风格快捷键——即control+b往左移动光标、control+f往右移动光标——居然不生效！这些快捷键对我个人还是非常重要的。

 

第二，在Yu Writer中，不能直接插入磁盘上的图片文件的绝对地址，既没有在预览区域显示出来，也没有在文档列表显示成功。

 

据说Yu Writer的作者的主业是厨师，感觉好强

 

Typora

Typora is best。不同于前面提到的几款Markdown编辑器，Typora是“所见即所得”的编辑器。你敲入两个井号，加一个空格，再敲入你的标题内容，最后回车，那么标题内容就会被渲染为二级标题的形式，如下图所示

 

 

 

这么一来，屏幕上的空间基本都可以被用来写作，不需要担心被预览用的列给占据了。

 

然后，Typora没有自定义它的存储结构，它直接打开磁盘上的.md文件进行编辑，这些Markdown源文件可以随心所欲地放在任何喜欢的目录下，只要能打开就行。再加上它文件树视图，就实现了不受限制的笔记组织方式了，如下图

 

 

 

不过一个可以想到的缺点，就是Typora不支持在所有的Markdown文件上搜索关键字——毕竟它也不知道要去哪个目录下寻找这些待搜索的源文件。

 

尽管Typora外观很简洁，但Boostnote有的功能它一个也没有落下，就像它的官网所说的那样

 

 

 

现在我的博客的文章基本都是用Typora来写的，冥冥中感受到了一股乐趣。但Typora毕竟没有搜索功能，所以我又开始摸索额外的搜索笔记的方式了（比如把记录在.org文件中的FAQ导入到ElasticSearch中再借助全文搜索的力量来找到自己要的内容）。

 

后记

没有最好的，只有最适合的，祝各位都能找到最适合自己的Markdown编辑器。

 

赞