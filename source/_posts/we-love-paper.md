title: 如何收集和整理论文（面向CS专业）
category: misc
date: 2016-09-28
tags:
---

论文（Paper）是每个研究生读研路上挥之不去的“阴云”。
无论是否已经有了一个好的课题或想法，都首先要收集某个研究方向一定数量的论文，来了解相关的工作和最新进展(State of the art & practice）。
本文介绍了如何检索、收集计算机科学（CS）专业的论文，还介绍了与相关的机构，学术会议和论文数据库。
文末有 [**Bonus**](#hosts) 哦;-)
本文原发在 https://ying-zhang.github.io/misc/2016-09-we-love-paper/

<!--more-->

---

# tl;dr
+ 从[CCF推荐目录](/misc/2017-02-ccf-all-in-one/)中自己感兴趣的方向的 **A类会议及期刊** 中找论文即可。
+ 我关注的云计算，程序分析方向的[会议和期刊列表](#tldr)
+ [**Bonus**](#hosts) 修改Hosts

# 引子
按理说，开篇应该先要强调一下Paper对于科研的重要性的，直接把前辈的经验拿来吧：
+ 周志华老师的一篇关于[做研究与写论文的ppt](/doc/research_and_paper_zhou_zhihua_2007_ppt.pdf)
+ 凌晓峰和杨强的[《学术研究 - 你的成功之道》](http://item.jd.com/11127141.html)，这本书的英文原版是[Crafting Your Research Future - A Guide to Successful Master's and Ph.D. Degrees in Science & Engineering](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=6813064)

首先要从前辈的经验中端正对Paper的态度：Paper不是科研的原因，而是结果(的一部分)，结果是必要的，自然也就少不了Paper或者总结报告；
再者要借鉴前辈的学习方法和技巧，所谓“工欲善其事，必先利其器”，除了科研课题本身，养成一套高效的科研方法和习惯也是重要的。
重要的是要 *有意识地探索和总结适合自己的科研方法*，既要低头苦干，又要抬头看路，还要回头总结。

## 论文发表的过程

<pre>
                                  / 期刊，特辑/专刊 → 多轮审稿 → 上线 \
Idea -> 编程、实验、写Paper、投稿 <                                    ->出版，检索
                                  \ 会议           → 审稿 → 赴会报告 /
</pre>

简单介绍一下发表论文的过程：
+ 首先投稿的Paper作者，一般是高校的的研究生，也有教师，比如[UC Berkeley 的AMPLab](https://amplab.cs.berkeley.edu/)；还有一些公司的研究院，比如[微软](https://www.microsoft.com/en-us/research/)，[谷歌](https://research.google.com/pubs/papers.html)。显然，论文的出身对质量有很大影响。
+ 期刊是传统的研究成果发表方式，一般期刊是季度，少有月度出版的，每期称为一个Issue，一年的各期集结为一个Volume。一般可以随时投稿给期刊，没有截稿日期(deadline)的压力。投稿后一般要经过同行评审(Peer Review)，针对审稿人的建议做大修，小修(Major，Minor Revision)等两三轮修改才能被接收，期间跨度一年多是常有的情况。不过如果有的期刊安排了专刊(Special Issue)计划，会公布一个截稿日期，审稿的进度会稍快些。
期刊分为 **Transaction, Journal 和 Magazine** 。这三者的学术严肃性依次降低，这可以从它们的封面上直观地看出来。严格来说Magazine不算是学术期刊了，上面很少发表新的原创性的内容，而是对当前进展的简介和综述，也会转发一些已经发表过的重要的Paper。 *不过对于新手来说，先浏览一下Magazine，建立一个基本的概念还是很有必要的，特别是* [Communications of the ACM, CACM, ACM通讯](http://dl.acm.org/citation.cfm?id=J79) *值得关注*。
+ 对快速发展的CS专业来说，期刊的节奏有点慢了，而且期刊版面有限，每期只能发布几篇到十几篇不等，所以上面发表的一般是一些理论性比较强的论文，或者综述性的论文；很多新成果转而发表在学术会议上来，这跟传统学科不太一样。
很多会议每年举行一次，时间上也是比较固定的，会提前在会议网站上发布下一年的call for paper（cfp，征稿启事）和deadline（截稿日期）。投稿后，一般经过两三个月的审稿就会通知作者是否录用（Notifications）。被接收的Paper会要求按格式和Reviewers的意见稍修改后提交正式的最终版，称为Camera Ready。最后需要作者赴会做Presentation。
会议录用的所有Paper会结集出版，称为 *Proceedings* 。有的会议还会推荐一些优秀的Paper到合作的期刊，扩展后作为期刊论文出版。
会议分为 *Symposium , Conference 和 Workshop*。这三者的学术严肃性依次降低，大部分会议都称为 *Conference*。一般来说 *Workshop* 是随某个 *Conference* 一同举办，可能没有固定的主题，Paper质量与主会议有所差别。

通过这个过程，我们还可以知道如何**尽快**找到一篇感兴趣的文章：
+ 对于期刊，一般投稿是很多的，编辑部会把已经接收但还没有排到出版期号的文章先放到网上在线出版，称为Early Access；
+ 对于会议，在确定了接收的文章后，会在会议网站的Program/Accepted Papers/Schedule等类似链接下给出列表，同时会Email通知作者准备提交Camera Ready版。这时有的作者就会把Camera Ready版放到自己的主页上。之后会议组委会将论文集结提供给所有作者，还会将论文集发布到ACM或IEEE(这两个机构直接参与了很多会议的组织)的论文数据库中。不同的会议组委效率不同，有的在开会前就上线出版了，有的在会议结束后后还要等一段时间。

# CS论文数据库

## ACM, IEEE Computer等

一般会议和期刊都有自己的网站，但很少能在上面获取到论文全文。又因为来源分散，直接从它们那里检索Paper很不方便。有几个大型的论文数据库，它们与期刊或者会议主办方合作，或者自己组织会议或编辑出版期刊，比如下面的表格及[图书馆页面截图](#lib)。
> 注意， DL和机构的网站有很多介绍性的内容是重复的，下载论文全文要去DL的页面。

| 机构                                     | Digital Library （DL）                              | 机构首页                   |
|------------------------------------------|----------------------------------------------------|----------------------------|
| Association for Computing Machinery, ACM | ACM Digital Library  https://dl.acm.org/           | https://www.acm.org/       |
| IEEE Computer Society                    | IEEE Xplore DL http://ieeexplore.ieee.org/         | https://www.computer.org/  |
| Elsevier ScienceDirect                   | http://www.sciencedirect.com/                      | https://www.elsevier.com/  |
| Springer                                 | Springer Link http://link.springer.com/            | http://www.springer.com/   |
| Wiley                                    | Wiley Online Lib http://onlinelibrary.wiley.com/   | http://www.wiley.com/      |

ACM 和 IEEE Computer Society（计算机学会，IEEE还有电气、电子、通信等其它多个学会） 的网址后缀是 *.org*，这两个是CS领域最重要的学术组织，很多的CS学术会议都是由它们组织的。
Elsevier，Springer，Wiley的网址后缀则是 *.com* ，这些是学术出版商，内容以期刊为主，涵盖了CS及其它多个学科。
上面这几个数据库是 **主要的论文全文来源**。它们各自收录的会议和期刊基本没有重叠，从它们的数据库下载的Paper也都有各自的排版样式。
> ACM作为最“正统”的计算机学术组织，它的DL除了收录ACM组织的会议和期刊全文之外，还会索引其它几家数据库的 **元数据**，但没有全文，不过可以通过DOI链接跳转到这几家数据库的全文页面。
> IEEE出版的一些论文在 computer.org （实际是[CSDL](https://www.computer.org/csdl/)）和 Xplore DL 都可能搜到，但这两个数据库是 *分别* 收费的，能在Xplore DL下载的不一定能在Computer.org下载。

### ACM SIGs
ACM之下针对CS多个子方向的“分舵”，称为Special Interest Group，SIG，目前有三十多个[ACM SIGs](http://www.acm.org/sigs/)（或参考DL的这个链接[SIGs ACM DL](http://dl.acm.org/sigs.cfm)），比如
+ 体系结构方向的[SIGARCH](http://www.sigarch.org/)、[SIGHPC](http://www.sighpc.org/)、[SIGMETRICS](http://www.sigmetrics.org/)、[SIGMICRO](http://www.sigmicro.org/)、[SIGMOBILE](http://www.sigmobile.org/)，
+ 网络方向的[SIGCOMM](http://www.sigcomm.org/)，
+ 数据库方向的[SIGMOD](http://www.sigmod.org/)，
+ 系统方向的[SIGOPS](http://www.sigops.org/)，
+ 软件工程方向的[SIGPLAN](http://www.sigplan.org/)、[SIGSOFT](http://www.sigsoft.org/)

这些SIGs除了组织一系列的学术会议，还会评选本方向的一些奖项，包括 **最佳论文**，**优秀博士论文** 等(在DL中一般没有哪篇是Best Paper的信息)。此外，
+ 有网站维护了一个[部分会议的最佳论文列表](http://jeffhuang.com/best_paper_awards.html)，
+ 还有下面要介绍的USENIX的[各会议最佳论文](https://www.usenix.org/conferences/best-papers)。

除此之外，有的SIG会选择一些高质量的文章，以Review，Newsletter 或Notes的形式重新发表。

## [USENIX](https://www.usenix.org/)
要是学校的图书馆不差钱，把所有有价值的论文数据库都买买买来，那么下面截图中的电子资源列表应该很全了吧？然而还是少了一个重要的数据库：USENIX —— 它是免费的。
话说**[USENIX](https://www.usenix.org/)** 实在是个良心组织。USENIX最初称为Unix User Group。它组织了OSDI 、ATC、FAST、NSDI、LISA等会议，不但学术水平很高，贴近工业界，而且免费提供全文下载，还提供一些论文作者在会议上的slices及演讲视频。slices是对文章的提炼，读论文时可以参考。拿slices和视频来学习做Presentation，练习英语听力和口语也不错。

## [arXiv](http://arxiv.org/)
[arXiv](http://arxiv.org/)， 是archive(归档)的意思，是一个由康乃尔大学维护的免费的多学科论文**预**出版(preprint)数据库。所谓**预**出版，就是说论文还没有经过同行评审，文责自负，文章质量参差不齐，所以一般不会作为正式的学术成果。不过有的学科习惯上先把文章公开到arXiv上，然后再提交到会议上。

<a name="lib" />![图书馆电子资源](/images/paper-lib.png)

## EI和SCI
分别搜索上面的数据库还是有点麻烦，于是就有了一些聚合数据库，又称为索引。想必很多同学在读研之前早就听说EI和SCI，
+ EI *Engineering Index* https://www.engineeringvillage.com/
+ SCI *Science Citation Index* http://apps.webofknowledge.com/

只看 *URL* 还以为是 *山寨网站*，它们的Web界面体验也不太友好，而且它们不止有CS一个学科，直接通过关键词搜索经常会给出不相关的内容。其实这两个数据库通常是在 **已知文章标题的情况下** 检索是不是被它们收录了，而 **不是** 用来收集文章的。

要确定某个会议论文集或者期刊[是否被EI或SCI收录](http://www.philippe-fournier-viger.com/links.php)，
+ 在[EI收录列表](https://www.elsevier.com/solutions/engineering-village/content) 页内搜索Compendex Source List，会找到一个Excel表格的链接，下载下来会发现这个表格是受保护的，但可以筛选标题（而且最后一个WorkSheet有中文翻译哦，满满的土洋结合，中国特色）。嘘~~ 也许你可以参考[这个脚本解除保护](/doc/crack_xls_vb.txt)，还要建议把title列中每个单元格开头的`=`替换掉。这个Excel的title是排好序的，方便顺序浏览，比如有个杂志名叫`Computing`，真是起的好名字，如果直接搜索是肯定搜出一堆结果，所以，即便找到名字一样的期刊，最好也要再确认一下ISSN号。**但是！**，这个Excel表格并不完整，如果没有在表中搜索到，还是需要在EI的网站上搜索文章标题才能最终确认。对了，EI的数据库叫 **Compendex**。
+ 在`webofknowledge`的网站查询之前，**一定** 要选择数据库为`检索 Web of Science 核心合集`，等自动刷新候，还要在页面下部展开“更多设置”，只选中`Science Citation Index Expanded (SCIEXPANDED) 1900年至今`这一项，然后才能查询出根正苗红的`SCI（E）`。请**务必**参考[这个截图](/doc/SCI_E_Web_of_Science.pdf)。可以在[SCI收录列表](http://ip-science.thomsonreuters.com/cgi-bin/jrnlst/jlsearch.cgi?PC=K)直接输入期刊的名称来查询该期刊是否被SCI收录，但感觉这个查的也不全。还是要充分利用SCI的中国特色了，因为还有一个国内整理的SCI期刊列表：[中国科学技术信息研究所SCI（E）论文期刊分区列表(2016年)](http://scit.nju.edu.cn/Item/1162.aspx)，这是一个有13.8k多行的Excel表格，简洁粗暴。

----
上面的这些数据库可以免费检索标题和摘要，购买全文则价格不菲。如果学校的图书馆购买了这些数据库，一般会识别用户的IP地址，在学校网络范围内可以直接下载PDF全文。
校外就没有这么方便了，好在很多作者在Paper被录用后会在自己的主页上挂出PDF全文，从Google Scholar上可以搜索到这些PDF全文的链接，非常方便。
话说只要是能花钱买到的东西，去万能的 **淘宝** 肯定能找到，就看是买 *VPN/代理*，*单篇文章*，还是 *整个数据库* 了。

## dblp
dblp [http://dblp.org] ，或[http://dblp.uni-trier.de]， 是专注于CS学科的文献 **元数据索引** 数据库，优势是收集得相当完整，链接也很有规律，比如特定会议的 FSE 2016 http://dblp.org/db/conf/sigsoft/fse2016.html 或者某个作者的全部论文列表(dblp对重名作者处理得很好)，但只能搜索标题或作者等元数据，用来初步筛选论文非常方便，需要获取全文时还是要跳转到上面的几个数据库，数据更新也稍微滞后一点。
2015版CCF目录中的会议和期刊都是dblp的链接。

dblp 列出了关于CS论文的一些统计数据，比如（2016年10月查询）
+ [累计论文记录数量](http://dblp.dagstuhl.de/statistics/recordsindblp.html)，
+ [每年发表的论文数量](http://dblp.dagstuhl.de/statistics/publicationsperyear.html)，
+ [论文发表的类型](http://dblp.dagstuhl.de/statistics/distributionofpublicationtype.html)，其中会议论文占53%，
+ 论文总数 3,587,354 ， 作者人数 1,825,286，会议数4,912，期刊数 1,491。

另外，ACM DL也有一个[类似的统计](http://dl.acm.org/contents_guide.cfm)。
![每年发表的CS论文数量](/images/paper-pubs.png)

而且dblp整站的数据都可以下载为一个[xml文件](http://dblp.dagstuhl.de/xml/)，以供进一步挖掘。

## DOI
在查找或引用论文时经常会遇到DOI(Digital Object Identifier)，[wikipedia介绍DOI](https://zh.wikipedia.org/wiki/DOI)“是一套识别数字资源的机制，涵括的对象有视频、报告或书籍等等。它既有一套为资源命名的机制，也有一套将识别号解析为具体地址的协议”。

## 其它
从Google Scholar搜索全文时可能会跳转到下面这几个网站，因为它们会保存一些别人分享的全文。
+ Semantic Scholar [https://www.semanticscholar.org]
+ CiteSeerX [http://citeseerx.ist.psu.edu/]
+ ResearchGate [https://www.researchgate.net/] ，这是一个学术社交网络

# CCF目录

EI和SCI只是两个论文数据库，但能够发表被EI和SCI收录的文章变成了能够毕业，能否获得奖学金，能否获得基金的指标。由于时代的限制，EI和SCI被赋予了不相称的地位和意义，而且短期看还是如此。
更 “*不幸*” 的是，对于CS的学生，还有一个[CCF目录](http://history.ccf.org.cn/sites/ccf/paiming.jsp)（[2015版的PDF](http://history.ccf.org.cn/sites/paiming/2015ccfmulu.pdf)）摆在面前。其实并非是不幸，而是十分幸运，因为CCF目录不像是一个紧箍咒，而更像是一个入门指南。

首先说[中国计算机学会 CCF](http://www.ccf.org.cn/)是国内的类似于ACM的计算机学术组织。也许某位同学的导师就是CCF会员。相比EI和SCI收录的成百上千的会议和期刊，CCF维护的目录显然[精简得多](/misc/2017-02-ccf-all-in-one/)。
考虑到对EI和SCI指标要求的实际情况，目录选取的 **大多** 是被EI或SCI收录的，具体划分为10个子方向，并区分出A，B，C三个等级。
A，B类的会议和期刊的文章学术质量较高，但这个质量不是简单通过所谓影响因子等机械的数据来评价的，而是综合了多种因素。如果翻一下本科的操作系统等骨干专业课教程，会发现其中引用的有不少A类的文章，比如SOSP，OSDI等。所以建议从A类文章开始研究会。

> 如果只看标题和摘要，有的期刊/会议文章看起来非常值得一读，比如 [FGCS	Future Generation Computer Systems](http://dblp.org/db/journals/fgcs/)上的文章。但如果仔细读一下文章全文，经常是大失所望。好在CCF只给了FGCS C类的评级。

上面提到ACM有三十多个SIGs，而CCF则只划分了10个子方向，不同的视角有不同的划分结果，这里有ACM的更详细的[分类系统CCS](http://dl.acm.org/ccs/ccs.cfm)，以及有重叠的[SIGs大类划分](https://www.acm.org/special-interest-groups/sigs-by-knowledge-area)，还有[wikipedia上的一个划分](https://en.wikipedia.org/wiki/Outline_of_computer_science)。

# Google Scholar（谷歌学术）
[Google Scholar](https://scholar.google.com/)非常强大又简单易用。虽然它不只收录CS专业的文献，但很容易搜索到准确的结果。我习惯先在谷歌学术上搜索，如果搜不到就改用 **高级搜索**，实在不行再去ACM DL、IEEE Xplore。
![谷歌学术高级搜索](/images/paper-scholar_adv.png)

## 创建快讯
与Google网页搜索一样，可以在Google Scholar创建某个关键词或某篇文章的快讯（发送邮件通知最新的搜索结果）；
此外，注意搜索到的论文的作者是否有链接，打开即是Google Scholar创建的作者个人资料页，上面一般有作者的单位、联系方式，文章列表等，在 **作者的个人资料页** 可以创建关于他的新文章或新引用的快讯，及时获取动态。

> 话说体验一下[必应学术](http://www.bing.com/academic)、[百度学术](http://xueshu.baidu.com/)和[搜狗学术](http://scholar.sogou.com/)也未尝不可。

<a name="tldr" />

# tl,dr：链接列表

## 体系结构，系统，存储，分布式
+ ASPLOS@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE178&tab=pubs) , [DBLP](http://dblp.org/db/conf/asplos/) Architectural Support for Programming Languages and Operating Systems
+ FAST@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE425&tab=pubs) , [DBLP](http://dblp.org/db/conf/fast/) Conf. on File and Storage Technologies
+ HPCA@**A**   [IEEE](http://ieeexplore.ieee.org/servlet/opac?punumber=1000335) , [DBLP](http://dblp.org/db/conf/hpca/) High-Performance Computer Architecture
+ ISCA@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE239&tab=pubs) , [DBLP](http://dblp.org/db/conf/isca/) Int. Symposium on Computer Architecture
+ MICRO@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE203&tab=pubs) , [DBLP](http://dblp.org/db/conf/micro/) Microarchitecture
+ PPoPP@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE241&tab=pubs) , [DBLP](http://dblp.org/db/conf/ppopp/) Principles and Practice of Parallel Programming
+ SC@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE207&tab=pubs) , [DBLP](http://dblp.org/db/conf/sc/) Int. Conf. for High Performance Computing, Networking, Storage, and Analysis
+ ATC@**A**   [USENIX](https://www.usenix.org/conferences/byname/131) , [DBLP](http://dblp.org/db/conf/usenix/) USENIX Annul Technical Conf.
+ CGO@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE256&tab=pubs) , [DBLP](http://dblp.org/db/conf/cgo/) Code Generation and Optimization
+ EuroSys@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE101&tab=pubs) , [DBLP](http://dblp.org/db/conf/eurosys/) European Conf. on Computer Systems
+ HotCHIPS@**B**   [HotChips](http://www.hotchips.org/) Symposium on High Performance Chips
+ HPDC@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE300&tab=pubs) , [DBLP](http://dblp.org/db/conf/hpdc/) High-Performance Distributed Computing
+ LISA@**B**   [USENIX](https://www.usenix.org/conferences/byname/5) , [DBLP](http://dblp.org/db/conf/lisa/) Large Installation system Administration Conf.
+ PODC@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE221&tab=pubs) , [DBLP](http://dblp.org/db/conf/podc/) Symposium on Principles of Distributed Computing
+ SIGMETRICS@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE187&tab=pubs)，[ACM](http://dl.acm.org/citation.cfm?id=J618) , [DBLP](http://dblp.org/db/conf/sigmetrics/) Int. Conf. on Measurement and Modeling of Computer Systems
+ VEE@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE287&tab=pubs) , [DBLP](http://dblp.org/db/conf/vee/) Virtual Execution Environments
+ TC@**A**   [IEEE](https://www.computer.org/web/tc) , [DBLP](http://dblp.org/db/journals/tc/) Trans. on Computers
+ TOCS@**A**   [ACM](http://dl.acm.org/pub.cfm?id=J774&tab=pubs) , [DBLP](http://dblp.org/db/journals/tocs/) Trans. on Computer Systems
+ TOS@**A**   [ACM](http://dl.acm.org/pub.cfm?id=J960&tab=pubs) , [DBLP](http://dblp.org/db/journals/tos/) Trans. on Storage
+ TPDS@**A**   [IEEE](https://www.computer.org/web/tpds) , [DBLP](http://dblp.org/db/journals/tpds/) Trans. on Parallel and Distributed Systems
+ ISMM   [ACM](http://dl.acm.org/event.cfm?id=RE149&tab=pubs) , [DBLP](http://dblp.org/db/conf/ismm/) Int. Conf. on Memory Management

## 软件工程（软件分析）
+ Github上的[软件工程方向会议的数据](https://github.com/tue-mdse/conferenceMetrics)
+ UIUC的[谢涛老师](http://taoxie.cs.illinois.edu/)维护的[软件工程方向的会议统计列表](http://taoxie.cs.illinois.edu/seconferences.htm)
+ ASE@**A**   [DBLP](http://dblp.org/db/conf/kbse/) Int. Conf. on Automated Software Engineering
+ FSE/ESEC@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE201&tab=pubs) , [DBLP](http://dblp.org/db/conf/sigsoft/) SIGSOFT Symposium on the Foundation of Software Engineering / European Software Engineering Conf.
+ ICSE@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE228&tab=pubs) , [DBLP](http://dblp.org/db/conf/icse/) FOSE会议：七年一届的展望 Int. Conf. on Software Engineering
+ OOPSLA/SPLASH@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE181&tab=pubs) , [DBLP](http://dblp.org/db/conf/oopsla/) Onward会议：创新(Nao Dong) Conf. on Object-Oriented Programming Systems, Languages, and Applications
+ OSDI@**A**   [USENIX](https://www.usenix.org/conferences/byname/179) , [DBLP](http://dblp.org/db/conf/osdi/) USENIX Symposium on Operating Systems Design and Implementations，**双数** 年份召开
+ SOSP@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE208&tab=pubs) , [DBLP](http://dblp.org/db/conf/sosp/)，Symposium on Operating Systems Principles， **单数** 年份召开，另，[SOSP 2015 History Day](http://sigops.org/sosp/sosp15/history/index.html)
+ PLDI@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE200&tab=pubs) , [DBLP](http://dblp.org/db/conf/pldi/) SIGPLAN Symposium on Programming Language Design and Implementation
+ POPL@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE180&tab=pubs) , [DBLP](http://dblp.org/db/conf/popl/) SIGPLAN&SIGACT Symposium on Principles of Programming Languages
+ ECOOP@**B**   [ECOOP](http://www.ecoop.org/) , [DBLP](http://dblp.org/db/conf/ecoop/) European Conf. on Object-Oriented Programming
+ HotOS@**B**   [USENIX](https://www.usenix.org/conferences/byname/155) , [DBLP](http://dblp.org/db/conf/hotos/) USENIX Workshop on Hot Topics in Operating Systems
+ ICSME@**B**   [IEEE](http://conferences.computer.org/icsm/) , [DBLP](http://dblp.org/db/conf/icsm/) Int. Conf. on Software Maintenance
+ ISSTA@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE222&tab=pubs) , [DBLP](http://dblp.org/db/conf/issta/) Int. Symposium on Software Testing and Analysis
+ TOPLAS@**A**   [ACM](http://dl.acm.org/pub.cfm?id=J783&tab=pubs) , [DBLP](http://dblp.org/db/journals/toplas/) Trans. on Programming Languages and Systems
+ TOSEM@**A**   [ACM](http://dl.acm.org/pub.cfm?id=J790&tab=pubs) , [DBLP](http://dblp.org/db/journals/tosem/) Trans. on Software Engineering Methodology
+ TSE@**A**   [IEEE](https://www.computer.org/web/tse) , [DBLP](http://dblp.org/db/journals/tse/) Trans. on Software Engineering

## 云计算，网络，大数据
+ SIGMOD@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE227&tab=pubs) , [DBLP](http://dblp.org/db/conf/sigmod/) Conf. on Management of Data
+ SoCC@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE227&tab=pubs) , [DBLP](http://dblp.org/db/conf/cloud/) Symposium on Cloud Computing
+ PODS@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE227&tab=pubs) , [DBLP](http://dblp.org/db/conf/pods/) SIGMOD Conf. on Principles of DB Systems
+ VLDB Endowment@**A**   [ACM](http://dl.acm.org/pub.cfm?id=J1174&tab=pubs) , [DBLP](http://dblp.org/db/conf/vldb/) Int. Conf. on Very Large Data Bases
+ VLDB Journal@**A**   [ACM](http://dl.acm.org/pub.cfm?id=J869&tab=pubs) , [DBLP](http://dblp.org/db/journals/vldb/) Int. Journal on Very Large Data Bases
+ NSDI@**B**   [USENIX](https://www.usenix.org/conferences/byname/178) , [DBLP](http://dblp.org/db/conf/nsdi/) Network System Design and Implementation
+ IEEE 云计算系列@**B/C**  [IEEE](http://cloudcomputing.ieee.org/conferences) , [DBLP](http://dblp.org/db/conf/IEEEcloud/)
+ HotCloud   [USENIX](https://www.usenix.org/conferences/byname/1) , [DBLP](http://dblp.org/db/conf/hotcloud/) Hot Topics on Cloud Computing
+ TCC   [IEEE](https://www.computer.org/web/tcc) , [DBLP](http://dblp.org/db/journals/tcc/) IEEE Trans. on Cloud Computing
+ TSC@**B**   [IEEE](https://www.computer.org/web/tsc), [DBLP](http://dblp.org/db/journals/tsc/) IEEE Trans. on Services Computing
上面体系结构中的 ASPLOS、FAST、ISCA、Micro、ATC、EuroSys、HPDC、LISA、SIGMETRICS、TPDS等，及软工的OSDI、SOSP等也都有不少云计算相关的文章。

## 移动计算
+ MobiCom@**A**   [ACM](http://dl.acm.org/event.cfm?id=RE366&tab=pubs) , [DBLP](http://dblp.org/db/conf/mobicom/) Mobile Computing and Networking
+ MobiSys@**B**   [ACM](http://dl.acm.org/event.cfm?id=RE191&tab=pubs) , [DBLP](http://dblp.org/db/conf/mobisys/) Mobile Systems, Applications, and Services
+ HotMobile   [ACM](http://dl.acm.org/event.cfm?id=RE142&tab=pubs) , [DBLP](http://dblp.org/db/conf/wmcsa/) Mobile Computing Systems and Applications

## ACM DL列表
+ [收录会议和期刊的完整列表](http://dl.acm.org/contents_dl.cfm)
+ [会议](http://dl.acm.org/events.cfm)
+ [会议历次论文集](http://dl.acm.org/proceedings.cfm)
+ [期刊和学报](http://dl.acm.org/pubs.cfm)
+ [杂志](http://dl.acm.org/mags.cfm)

关于ACM的杂志，特别推荐
+ [Communications of the ACM, CACM](http://dl.acm.org/citation.cfm?id=J79)， [in dblp](http://dblp.org/db/journals/cacm/)
+ [CACM中国版](http://dl.acm.org/toco_arch.cfm?id=J79&lang=chinese)，CCF曾经 选译 一部分CACM文章为中文并出版，但2016年后没有继续下去
+ [Queue](http://dl.acm.org/citation.cfm?id=J882)也值得一看，不过它与CACM有很多重叠的文章

期刊中，推荐[ACM Computing Surveys, CSUR](http://dl.acm.org/citation.cfm?id=J204)， [in dblp](http://dblp.org/db/journals/csur/)

> 看了ACM DL的列表页面，应该能体会到：1000项以内的列表还是不要分页的好。

##  IEEE Computer列表

+ [会议日历](https://www.computer.org/web/conferences/calendar/)
+ [期刊和学报](https://www.computer.org/web/publications/transactions)
+ [杂志](https://www.computer.org/web/publications/magazines)

## [USENIX组织的会议列表](https://www.usenix.org/conferences/byname)
[USENIX组织的会议列表](https://www.usenix.org/conferences/byname)，其中包括ATC，FAST，LISA，MobiSys，NSDI，OSDI，VEE及HotCloud，HotOS等一系列 HotXXXX 的Workshop。

## 国内三个学报

+ [软件学报](http://www.jos.org.cn/ch/index.aspx)，[CNKI RSS](http://rss.cnki.net/KNS/rss.aspx?journal=RJXB&amp;Virtual=KNS)
+ [计算机学报](http://cjc.ict.ac.cn/)，[CNKI RSS](http://rss.cnki.net/KNS/rss.aspx?journal=JSJX&amp;Virtual=KNS)
+ [计算机研究与发展](http://crad.ict.ac.cn/CN/volumn/home.shtml) ，[CNKI RSS](http://rss.cnki.net/KNS/rss.aspx?journal=JFYZ&amp;Virtual=KNS)

## 国内论文数据库

+ [知网CNKI](http://www.cnki.net/)
+ [万方数据](http://www.wanfangdata.com.cn/)

## 其它链接
+ [微软研究院](https://www.microsoft.com/en-us/research/)
+ [谷歌研究院](https://research.google.com/pubs/papers.html)
+ [The morning paper](https://blog.acolyer.org/), an interesting-influential-important paper from the world of CS every weekday morning
+ [IEEE Technical Committee on Data Engineering](http://sites.computer.org/debull/bull_issues.html)
+ [YouTube](https://www.youtube.com)，
+ **[Suggested Guidelines for Finding Materials to include in the "Related Work" Sections of Conference Papers](http://www1.cs.columbia.edu/~kaiser/relatedwork.htm)**
+ [YOCSEF专题论坛：从LNCS事件反思中国学术论文的发表](http://www.yocsef.org.cn/sites/yocweb/yocltzw.jsp?contentId=2658502145582)  

# 如何读论文
+ [Efficient Reading of Papers in Science and Technology(.pdf)](http://www.cs.columbia.edu/~hgs/netbib/efficientReading.pdf)
+ [How to Read a Paper(.pdf)](http://blizzard.cs.uwaterloo.ca/keshav/home/Papers/data/07/paper-reading.pdf)
+ [How to Read a Technical Paper](https://www.cs.jhu.edu/~jason/advice/how-to-read-a-paper.html)
+ [《学术研究 - 你的成功之道》第3章](http://item.jd.com/11127141.html)

# [Todo]辅助工具
+ [会伴](http://myhuiban.com)
+ Trans. on BigData的学术文献处理专刊 [Vol. 2 Issue 1]](https://www.computer.org/csdl/trans/bd/2016/01/index.html)，[Vol. 2 Issue 2]](https://www.computer.org/csdl/trans/bd/2016/02/index.html)
+ [Sciplore](http://www.sciplore.org/)
+ [Scopus](https://www.scopus.com/)
+ [Docear](http://www.docear.org/)
+ [Mendeley](https://www.mendeley.com/)
+ [Zotero](https://www.zotero.org/)
+ [Teambition](https://www.teambition.com/)
+ Todo，如何整理文献，如何管理时间，[科研小组里有哪些有效的组会形式 - 知乎](https://www.zhihu.com/question/27956707)

如果所在的实验室没有什么积累，暂时没有好的idea/topic，不妨去浏览一下感兴趣的大方向的A类会议近三年或五年的文章列表，**总会觉得** 有几篇比其它文章更有意思，这就把范围缩小一些了；然后再从这几篇文章里提炼出关键词，去[wikipedia](https://en.wikipedia.org/)上搜索一下这个关键词，再从Related Work再扩展出去，体验一下这个小领域涉及的问题。找出来这个小领域里发文章比较多的作者和研究小组（实验室），去作者和小组的主页看看。这个前期工作其实花不了一周的时间，然后就收集一些相关的论文，粗览一遍，筛选出值得仔细研读的。我们不妨通过相关论文的数量来定义一个小的领域，武断地说100篇或200篇，这个具体的大小不是关键，但一个领域能发的文章必定是有限的，太多的话说明问题太复杂，还要细分，太少的话，如果不是幸运地发现了新的方向，就是问题太Trivial了。而这其中，又只有几篇或十几篇是开创性的，非常值得仔细研读的。
如果读完核心的几篇文章后还是没有新的想法，那就只好重复上面的过程，重新寻找另外感兴趣的领域了。

> PS，相比上面列出来一堆链接，其实这几句话才是这篇文章的重点啊 ;-)

一些标题有`A systematic review on ...`综述文章，其中会介绍收集相关论文的过程，方法都类似，可以找一篇当作论文搜集方法的教程来看。

再列出知乎上的几个相关问题吧
+ [如何总结和整理学术文献？](https://www.zhihu.com/question/26901116)
+ [如何高效管理文献？](https://www.zhihu.com/question/26857521)
+ [如何写好一篇高质量的IEEE/ACM Transaction级别的计算机科学论文?](https://www.zhihu.com/question/22790506)

<a name="hosts" />
# [Bonus] 如何访问Google Scholar

## **改hosts**
+ IP v4， https://raw.githubusercontent.com/racaljk/hosts/master/hosts  或短网址 https://git.io/vWE1N
+ IP v6， https://raw.githubusercontent.com/lennylxx/ipv6-hosts/master/host 或短网址 https://git.io/vMjCk

## **hosts文件的路径**
+ Windows：`C:\Windows\System32\drivers\etc\hosts`
+ Linux，Mac，Android(均需要root权限)：`/etc/hosts`

---

飞鸟集

> 第83
> 那想做好人的，在门外敲着门，那爱人的，看见门敞开着。

> 第142
> 让我设想，在群星之中，有一颗星是指导着我的生命通过不可知的黑暗的。
