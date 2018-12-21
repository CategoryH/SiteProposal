# SiteProposal

### 注册流程
填邮箱，密码，确认密码，**邀请码**（详见下文），然后填邮箱里收到的验证码，然后注册。

昵称可以改，但全站唯一，不带头像。

#### 公私钥保证溯源
用户在网站注册的时候，网站应给用户分配一对公私钥。公钥对所有人可见，私钥只有用户本人和网站可见。高级玩家可以**自己生成公私钥对**，在注册时提交自己的公钥，自己保管私钥（这种情形私钥不可找回）。

#### 登录
1. 用邮箱和密码登录
2. 用密钥签名登录

#### 密码找回和修改
1. 用邮箱验证
2. 用密钥验证

### 用户的属性

#### public:
* 昵称（全站唯一）
* 公钥
* 一句话简介
* 角色（详见下文）

#### private:
* 私钥（可能不存在）
* 邮箱
* 密码
* 投票权重：`vote_powers` 是一个数值向量，每个版面一个数值，决定了用户在每个版的投票权重，默认值在每个版都是 `100.0`. 最低可以低到 `1.0`, 最高 `2000.0` （详见下文）
  
TODO: 投票权重要不要公开？
TODO: 关注问题、版面、收藏夹是公开还是私密？

### 用户角色
用户有三种角色，是递进关系
1. 普通用户
2. 版面维护者
3. 管理员

#### 通用操作：
登录/注销/找回密码/修改密码

#### 专有操作
**普通用户**：以版面 `/r/HelloWorld` 为例

1. 提问/编辑提问（用户把问题提交给版面如 `/r/HelloWorld`）
2. 回答/编辑回答（用户把回答提交给问题如 `/question/X`，生成回答 `/question/X/answer/Y`）
3. 赞/踩问题（赞：`upvote += user.vote_powers("HelloWorld")`, 踩: `downvote += user.vote_powers("HelloWorld")`, 注意两者是分开计数的，每个问题计算 `vote = (upvote, downvote)`.）
4. 赞/踩答案（同上）
5. 问题/答案提交的同时默认提交者给自己 upvote（用权重算）
5. 关注问题（+取消）
6. 关注版面（+取消）
7. 关注收藏夹（+取消）
8. 编辑个人资料（TODO: 个人资料放哪些内容？一句话简介？）

投票权重比较高的用户（例如 `vote_power >= 200`）获得关闭问题的权力。
	
**版面维护者**：

1. 给优质答案（详见下文）的作者发放邀请码
2. TODO: 版面维护者看到的信息要不要比普通用户多一些，多哪些？

**管理员**：

1. 创建版面（TODO: 创建版面的规则。“子版面可以由某个版面的高质量用户（标准待定）提出，提出后，获得全站级运营的通过，成为”试验“版面，维持一段时间(1~3个月)以后，全站级运营根据版面质量决定是否转正，如果不转正，那么试验版面下的所有内容都归入原父版面。”）
2. 任免版面维护者

### 邀请码
邀请码是注册的必需品，邀请码由系统生成，发放渠道：
1. 版面维护者发给高质量答案的作者（每个高质量回答两个邀请码，每个人可用的邀请码不能超过五个）。
2. 初期，管理员定向邀请。

### 版面
定义：版面是一个固定主题的讨论场所，是固定主题下问题和答案的总体。hello 话题的页面记作 `site.com/r/hello`

#### 版面分栏：优质内容/问题列表/最新回答/精华

1. 版面首页: `r/hello#popular`
2. 问题列表: `r/hellp#questions`
3. 最新回答: `r/hello#newest`
4. 精华内容: `r/hello#best`

**版面首页**，预览一系列答案，根据热度 `hotness` 排序。其中热度是质量和时间的函数：

     hotness = f(quality, date)
 
目标是做到优质内容分数高，新内容优先于老内容。其中 `quality` 可以这样定义（参数待定）

      vdiff = upvote - downvote // 每个 vote 带来的变化 ≈ 100
       sign = sgn(vdiff)
    quality = sign * log(1 + abs(vdiff/1000)) * 10 // 这是与时间无关的 quality, 
              // 这个 quality 的要点在于，vdiff 不大的时候基本上跟 vdiff 是线性的，
              // 较大的时候每次 vdiff 增加带来的 quality 增量较小
              // 里面的 1000 和 10 是可以调整的参数
    
而 `date` 是（答案的）诞生时间，定义为发文时刻的 `epochtime() - Constant`, 单位是秒。

如果定义

    hotness = quality + k * date
    
则可实现优质内容分数高，新内容优先于老内容，其中 `k` 是一个正的参数。（例如定义 `k = 1/(6 * 60 * 60) = 1/21600`, 相当于是说每 6 小时热度减一，因为六小时之后发的内容 `k * date` 比六小时前的多 1.）

（更自然的想法是用发帖至今的时间，帖子的 ”年龄“ 来计算 hotness, 但年龄是个一直变化的东西，`date` 是一个固定值，发文的那一刻就可计算，且后来者的 `date` 一定大于先来的，这是 reddit 的办法。）

**问题列表**（时间和 votes 的函数，`downvote >> upvote` 的关闭）

**最新回答**（时间逆序，**仅**按时间排序）

**精华内容**（与时间无关的，讨论沉淀下来的优质内容）
直接用 quality 进行排序？

### 投票权重
* 是一个数值向量，每个版面一个数值，决定了用户在每个版的投票权重
* 默认值在每个版都是 `100.0`. 
* 最低可以低到 `1.0`, 最高 `2000.0`.
* 提供一个优质答案，该版面投票权重增加 `(5 + 45 * r)/s`, 其中
	*  `r = min(1, 3 * q)`, 这里 `q` 是用户提交优质答案时，创作的优质答案和普通答案的比例。（逻辑: 奖励纯干货输出者，优质答案/普通答案 > 1/3 则无惩罚，但如果一个人能提供优质答案但是灌水太多，投票权重增长得就比较缓慢，反之则增长快）。
	*  `(5, 45)` 这对参数用来调节灌水带来的损失：前者越大，发优质内容的固定收益越多；后者越大，灌水损失越重。`(40, 10)` 就是鼓励创作内容，不太惩罚灌水；`(5, 45)` 的话水车损失较大。（TODO: 这个参数可以交给各个版面自行调节？）
	*  `s = max(0.9, sqrt(voting_power/100))`, 也就是说，如果权重已经较高，则权重增长应该放缓。`s` 的取值范围从 `0.9` 到 `sqrt(20) = 4.47`.
	* `max(0.9, -)` 是为了避免投票权重降得很低以后一个优质输出带来的权重增量爆炸。
	* 用上面这个数字的话，不灌水输出 `10` 个优质答案之后 `voting_power = 425.99`, 输出 `100` 个优质答案之后 `voting_power = 1803.10`. 如果输出优质答案的比例是 10%，输出 `31` 个优质答案之后 `voting_power = 455.15`. 
* 提供一个劣质答案或者被关闭的提问，该版面投票权重减 `9`. 最低 `= 1`.
* (TODO) 对于违反版面规则的用户，可以用降低权重处罚。
* (TODO) 高权重用户的权重是否应该适当随时间递减？比如权重超过 `400` 就每天 `-1` 直到权重 `= 400` 为止。

### 优质答案
TODO: 定义优质答案。

### 全站首页
TODO: 定义全站首页的排序

用户订阅了不同的版面、问题和收藏夹，全站首页给这些地方出现的新内容一个排序。

### 杂项
MathJax + XyJax 设置参见 https://stacks.math.columbia.edu/tag/0780 的 source code.

### Idea
vote 越多，每个 vote 的贡献越小？


