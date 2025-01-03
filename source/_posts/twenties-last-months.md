---
title: 二十多岁即将结束的感想
date: 2024-12-28 22:00:00
categories: 生活
tags: [年终总结]
---

已经狂飙了好几年，没有时间和心态用文字记录心情，但若想变回创作者，就必须使用文字写下所感所想，以此篇文章热身，让自己回到创作者的姿态。

2024年终于是我20多岁的最后一年，明年的3月5日正式步入30岁，再也不能以年轻人自居了。

转瞬即逝，一眨眼就变成了两个孩子的爸爸，甚至还做了几年老板，而外公和奶奶已经过世。如果回到20岁告诉自己在30岁前将要扮演哪些角色，20岁的自己肯定不会相信。

<!--more-->

## Memory

2017年中旬，尚在22岁，除了接外包和兼职程序员以外，发现了以太坊显卡挖矿这个新玩意。当时买了两张1050Ti接上实验室的PC连星火矿池挖矿，思考能不能做个类似星火矿池一样的产品赚钱。

2017年底发现了 Nimiq 网页挖矿的 demo，链的源码刚好是我接项目最常用的 Node.js，于是我进行了长达三个月的开发，实现了一套完整的自研 Nimiq 矿池程序和客户端。虽然有诸多漏洞，但在18年4月 Nimiq 主网上线首发时是唯一矿池程序，并在头一个月都垄断了全网90%以上的算力。

当年的 PoW 矿池通常会在超过 51% 算力的时候砍自己一刀，为此我还有彻夜在 Discord 和人用英文对喷两个小时的经历。Look back，虽然遭到各种协议攻击、DDoS攻击、社区人生攻击，但那几个月是个人真正意义上飞速成长的几个月，上线第一天矿池总产币折合人民币60万，直接改变了我的价值观，毕竟当年在学校苦哈哈的做一年外包收入也就20万元。而当时对区块链的技术理想和社区还有一丝天真的想法，在垄断的行情下只抽水 2% 也让我真正意义上的第一桶金大大缩水，这也许就是穷人出生的后遗症。

正是从无到有的 Nimiq 矿池开发和运营，让我对生产关系有了更深层次的认识，Nimiq 矿池让我在硕士毕业前就赚到了200万元，而当年的百度阿里应届生程序员月收入也仅有1万元出头，池子的抽水等于我做十年外包的收入。但矿池也让我错过了加入明日方舟初始团队的机会。

最开始的 Nimiq 是基于 CPU 进行挖矿，后续有国外开发者实现了 GPU 挖矿并被我逆向重打包使用。尝到甜头后，我开始了 PoW 的路径依赖，对于散户来说 CPU 和 GPU 是最容易使用的挖矿设备，因此我疯狂探索 CPU 和 GPU 挖矿币种，接下来就做了门罗矿池并且一直运营至今。

2018年开始 Crypto 经历了一轮很长的熊市，ETH 从最高1万元跌到了几百元，经历了2018年初期的高收益后，Nimiq 和 Monero 矿池不温不火，所以我当时为了户口选择了在支付宝工作一年。与此同时，上班的税后2万和矿池上线初期的日流水60万让我深刻的怀疑打工的意义，2019年的上班收入甚至低于我大三大四爆肝做外包的收入。与此同时，2019年5月明日方舟上线爆火流水20亿，也让我坚定了“打工是不可能打工”的想法。

支付宝正常的写代码和迭代交付反倒没有什么印象，因为都太过于简单和平淡。在支付宝工作印象最深刻的一件事是让高职位的员工给新人进行谈心培训，言传身教。而早年阿里的员工有很多是大专生，作为元老经过快速发展已经成为 P9 P10，所以当一个大专生P9对一众985交大浙大本硕传授经验时，让我感受到这个世界的荒诞，对于做题家而言。另一件印象较深的事是整个大组60+号人飞到张家口徒步三天，晚上零下气温住自带的帐篷。这种活动对于95后员工可能过于古早和陌生，对于销售基因的公司保留江湖味却是必不可少的一环。那三天我徒步走了60公里。

2020年5月离职正式全职创业的时候，公司最初的目标是做手游，但又缺乏做游戏的经验，过于匆忙没有明确的立项目标。此时 Crypto 市场开始逐步转牛，缺乏明确目标的我又希望快速积累一波原始资金，穷是原罪，为此在熊市行情下一边把 PoW 矿池模式复制到各个链上，一边做各种 Unity Demo 寻找玩法。Look back，创业大忌是三心二意，会对心性造成严重的损伤同时也会错过最佳的爆发时机。

2020年市场重新转牛，以太坊挖矿收益逐步抬升，当时看到一个新闻讲韩国网吧不营业只做 GPU 挖矿。当时我想到专门做一个针对国内网吧市场的 GPU 挖矿程序，可以在闲置的时候自动启动挖矿，而在有人上机的时候降低比例或者暂停挖矿。以太坊矿池涉及到高额的打包 MEV 收益所以技术相对复杂，所以放弃这一块的开发只做客户端，全部接入星火矿池。随后在上线初期又针对用户群里的反馈做了网维模式和网维抽水，这个功能随着以太坊挖矿收益的暴涨让用户数量彻底裂变（巅峰时一张原价12000元的3090一天收入140元），因为网维抽水能让各个城市的网维赚到自己数月甚至数年的收入，等于让我瞬间有了数百名遍布全国的免费销售推广员和客服。半年左右的巅峰期，天池网吧版客户端占据了以太坊网络2%的算力，近十万张显卡。

网吧版的收益随着以太坊暴跌和市场转熊而下降，最终在以太坊转向 PoS 后基本结束，百万级别的闲置 GPU 让显卡 挖矿开启了大逃杀模式，显卡挖矿这个市场市值也萎缩了 95%。在后 PoW 时代，也就是2022年6月风控结束我把熊掌工作室人员遣散后，有几个月我试图逆潮流而行，续命又上了好几个币种的矿池，但都因为利润太低相比当年的网吧矿工和 Nimiq、Monero 矿池忽略不计而在随后的两年陆续关闭。

随后2022年底到2023年又爆肝上的 DeFi 项目 AnimeSwap 和 CEX、DEX 套利等项目，但都因为这样或者那样的问题，无法获得网吧矿工那样的辉煌数据和回报，这些项目如果在我一无所有的时候（例如 Nimiq 矿池开发和运营时期）有可能因为强大的正反馈继续迭代而做大做强，但有时候人有了积累、实现了梦想，就会忽视小的项目正反馈，缺乏动力，从而错过更大的利润。归根到底，曾经做矿池的根源动力是技术兴趣和贫穷，而现在的 Crypto 领域更像金融领域，在做 AnimeSwap 期间和这些金融人士打交道让我对行业人群的价值观取向产生了极端的厌恶情绪，或许我本质上是个艺术家人格，就像《黑客与画家》所描述的。

现在想想不可思议，我竟然在网吧矿工项目开发运营期间，还带队开发上线了三款手游和一款单机，可以说是三心二意的巅峰代表，也是并行开发的极限。从纯理性的数据产品角度来看并行没有任何问题，但这对于2020年后的游戏内容创作行业是非常致命的，浮躁的切换是没有办法做出发自内心而能够打动人心的内容。这些游戏的开发和运营并非完全没有收获，让我理解了游戏测试反馈的重要性，2020年后手游运营的复杂程度和门槛。最重要的是获得了2年的单机游戏发行和营销体系实战，让我对后续的产品和市场更有理解。

## Dream

最初保送时选择计算机专业除了小时候沉迷装机、游戏、黑客故事就是比尔盖茨和扎克伯格的传说。为了赚取生活费和学费，我延续了高三保送后做家教的习惯，刚进大学时每周末去徐家汇给沪爷补习高中数学赚取两小时240元的生活费，然后在交大一餐食堂靠5元挂逼面续命。

曾经的我梦想十分渺小，只是希望在30岁前开发一个小产品赚到1000万，这样每天都能吃得起牛排。Look back，穷人出生的狭窄视野会影响到年轻时的选择，导致错过许多机会。彻底革新自己理解的世界需要机遇，我在大学期间没日没夜的写代码，疯狂折腾开发了上百个项目 repo 后，终于遇到了 Nimiq 矿池这个机遇而自我领悟了商业的法则。这种领悟只能靠刻骨铭心的经历，无法言传（数次天真的试图让人领悟某些法则，最终放弃）。我有几次机会能更早的获得这种机遇，但天生的性格让我难以作为陪衬完全信任他人，或许人的一生命运早已被性格决定。16岁领悟、23岁领悟和终身无法领悟的人境遇是完全不同的，而极端成功人士的聪明孩子能从小开悟。

比不能实现梦想更残酷的是梦想实现后该怎么办。实现了每天吃得起牛排的梦想，原本做事的初衷和动力会发生微妙的变化，这是穷人的梦想太小以至于太容易实现造成的问题。解决这个问题又需要残酷的自我更新迭代，我悟道终极的解决方案是 Dream Big，梦想实现后就需要设立一个发自内心的更大的梦想，这样才能一直走在前行的道路上而不成为行尸走肉，这并非一件容易的事情。

有次和一位流水百亿二次元手游的创始人取经，他的核心观点是做项目的钱并不重要，再多钱也不能把项目砸成功。而在经历了多个项目起起伏伏之后，我才终于明白这个观点的正确性。

创立一个新项目，最重要的是创始人的心性，只要大致路线是正确的，最后成功只看心性和愿力。心性足够，总能在三到五年的持续迭代中根据市场反馈找到正确的路线。中二的说，这是一种创始人与生俱来的愿力，我始终相信只要愿力充足到一定强度全世界都会给你开道。而心性不足，迭代只能高效维持一年甚至半年，就算走在正确的道路上，也会被时间抛弃。

黑神话的开发只用了3个亿，明日方舟只用了3000万。对于大部分公司甚至一些个人投资者，这并不什么大钱。创始人的心性和愿力才是新项目的核心，正确的人能把资源放大一万倍使用，人的力量是物质无法匹敌的。

## Future

作为碳基生物也需要迭代进化，30岁后给自己定下几个 key points。

（一）用钱维护好时间和精神状态

从小时候养成的习惯是时间不值钱，小学中学的时候经常为了省一块钱公交费暴走30分钟。这个习惯非常难改，至今还会为了节省打车费而暴走或骑车20分钟。

DeFi 的秒级利息充分量化了时间的金钱价值，最近几年正在逐步改善这种恶劣的习惯，可能一辈子也省不下一次大操作的手续费。用钱维护好时间和精神状态是非常重要的一笔小额投资，浪费的时间和精神状态会造成更严重的损失。

（二）花费时间保持健康

在正确的赛道上前进，过了入门门槛后最重要的就是体力和专注力，而维护长期战斗的基础就是身体健康。

得益于小学练习武术、中学篮球校队、大学踢足球，身体在20+岁的前中期还有一定的底子可以熬夜爆肝霍霍。而到了最近几年越发感觉到底子快被消耗殆尽，需要恢复运动保持身体健康。

（三）足够的作品输入

科幻是面向未来的作品，我喜欢幻想未来而不是拘泥于历史。虽然会玩会看一些中世纪和奇幻题材的作品，但我更多的时间还是选择科幻题材的ACGN。

富野由悠季、贰瓶勉、小岛秀夫是我最喜欢的创作者，在我看来是科幻题材里动画、漫画、游戏的巅峰。

富野的剧本大纲既有宏大的战争和政治场面，又能够对人物内心进行详细的刻画，所有人物的动机都是符合逻辑的，从不拖泥带水。巅峰时期的高达Z TV50集的容量做出了一部跌宕起伏的宇宙史诗。富野笔下所有女性的刻画性格鲜明，从来不落入俗套，这可能也得益于70、80年代的价值观，在我看来没有一个动画监督能与他的才能相提并论。

贰瓶勉天马行空的想象力让他作品的世界观和设定独一无二，建筑系出生的技能对科幻题材的巨物刻画全球独此一家，结合一点日本人物哀情节让宇宙尺度上的孤独感表现的玲离尽致。

小岛秀夫的作品横跨机甲ACT、科幻AVG，还开创了潜行和送快递两个玩法（开创一种玩法已经足以成为大师），他作品里的科幻设定跟游戏玩法结合的恰到好处，科幻题材的美术审美世不二出，50岁被裁后成功创业并运营自媒体的神迹，是近乎于马斯克的行为。很难解释世间竟有如此天才。

要产出有深度的作品需要足够的输入，源源不断的输入是创作者的根本源泉。儿子一岁生日那天看到小岛秀夫的X上发的工位照片有我刚看完的《希德尼娅的骑士》全套漫画，而其作者贰瓶勉（也是我最喜欢的漫画家）也是当天生日，非常精妙的巧合。小岛秀夫60岁至今还能在做制作人的同时进行大量的作品输入，比如连续6个小时看完18集番，这对于60岁的人是非常恐怖的活力，而很多人30岁已经如同一滩死尸。创作者一定需要保持高强度的作品输入。

（四）做自我认可的事情才有持续的动力

在正确且困难的道路上持续前进，总会获得正反馈，不会疲倦。反之在做不到自我认同的道路上前行只会充满痛苦，正常人很难扛过3年，扛过去了也会成为行尸走肉，收获毫无意义的人生。

一定要在自己认同的赛道上持续前进，长长的雪道厚厚的雪，5年10年后的雪球才能越滚越大。如果有什么形而上的道要传给儿子和女儿，“尽早找到自己认同的道路”一定是最关键的一条。

（五）AI 强化创作者的能力

在用过 GitHub Copilot 后，我写代码很少再去使用搜索引擎。而 Niji Journey 直接给我产出角色人设的原型。人类正处于硅基超越碳基的奇点，未来十年的工作和创作方式会发生巨变。如果 AI Agent 疯狂输入创作者本人的习惯，也许会拥有真正的 Ghost，Ghost in the shell。

在这个时间点，以小博大的创作游戏，一定要把工作流和 AI 工具结合，也许十年后创作者和 AI 融合会成为常态，美术、音乐、剧本、程序都将如此。题外话，若干个生产力爆炸的托拉斯企业将诞生在这一周期，如果不能参与其中或者拥有一部分股份，或许未来只能被托拉斯们圈养。
