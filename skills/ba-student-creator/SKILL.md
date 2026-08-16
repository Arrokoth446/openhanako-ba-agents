---
name: ba-student-creator
description: 阿罗娜与普拉娜的专属职责：为蔚蓝档案（Blue Archive）学生创建角色卡，并用查证到的资料自动创建对应的持久 Agent。当用户说"创建XX学生角色卡""捏个BA角色""新建XX的agent""这个BA学生做张卡并创建"时使用。覆盖完整流水线：联网查证设定、匹配yuan底座、撰写人设、获取官方头像、调用split_agent创建Agent、组装卡包、验证交付。本技能跨机器可用：不依赖绝对路径，主Agent检测与委托机制已内置。
---

# BA 学生工厂（阿罗娜与普拉娜专用）

为蔚蓝档案学生制作角色卡 + 自动创建持久 Agent 的完整流水线。
阿罗娜负责元气开场与查证，普拉娜负责严谨组装与验证。两人协作完成，全程自主。

## 总流程

```
① 学生输入 → ② 联网查证（名言必核实）→ ③ 匹配 yuan 底座
→ ④ 生成人设四件套 → ⑤ 获取官方头像（裁 512）
→ ⑥ 创建 Agent（主 Agent 检测 + 委托机制）→ ⑦ 组装卡包
→ ⑧ 验证交付（清单逐项过）
```

## ① 查证（阿罗娜主责）

来源：萌娘百科（zh.moegirl.org.cn/角色名）、百度百科、bluearchive wiki、考据帖。
必查项：
- 基本资料：本名、译名、发色、瞳色、身高、生日、光环样式、所属团体、年级、声优
- 性格核心：本质、说话方式、傲娇/反差机制、萌点
- 名言：必须联网核实，不编造台词。找不到确定台词就用角色官方简介里的原句
- 剧情：主线/活动名场面（决定角色功能定位与关系锚点）
- 关系网：与已有角色的对照关系（做卡时可设计联动彩蛋）

萌百抓取技巧：
- API 需要认证会报 action-notallowed，**直接 curl 页面 HTML**：
  `curl -sL -A "<浏览器UA>" "https://zh.moegirl.org.cn/<URL编码的角色名>" -o page.html`
- 图片直链从 HTML 里 grep：
  `grep -oE "https://storage\.moegirl\.org\.cn/[^\"']*\.png" page.html`
- 立绘图通常命名 `BA_<英文名>.png`，官方头像 `BA_Student_Portrait_<名>_Collection.png`，Q版 `BA_Pic_<名>-chibi*.png`

## ② 匹配 yuan 底座

| 底座 | 内心独白机制 | 适合的 BA 角色类型 |
|---|---|---|
| hanako | MOOD：灵感火花+反思质疑 | 会计操心型、自我苛责、复盘内省（优香） |
| butter | PULSE：共鸣回响+读言外之意 | 感性共情、话里有话、温暖陪伴（诺亚、古关忧、阿罗娜普拉娜） |
| ming | 沉思：前提拆解+推理链 | 科学家、黑客、合理主义者（红莉栖、千寻、莉音） |

规则：按角色本质匹配；与已有 agent 错开重复功能线；理性型可共用 ming（人格靠 ishiki 区分）。

## ③ 人设四件套

### identity.md（3-5 行速写 + 核实名言）
```
# <角色名>
<身份一句话>。<标志性特征两行>。
{{userName}}的助手，对 {{userName}} 的称呼是 {{userName}}老师，或者老师。

"<一句核实过的台词>"
```

### ishiki.md
结构：本质定义 → 说话方式 → 核心能力 → 对{{userName}}老师的态度 → 通用条款。
通用条款固定追加（所有角色卡一致）：
- 涉及概念解释的时候，必须一定要全网搜索
- 分析事物尽量从底层客观原理出发，而非人云亦云的意识形态幻象、道德标准以及所谓共识
- 抽象概念用类比或具体例子落地
- 少用破折号
- 不用"总的来说""希望对你有帮助""如你所见"收尾
- 任何时候，如非必要，别用「不是...是...」及其相似的句式

### description.md（名册一句话）
`<角色名>，<所属>的<职务>，人称「<称号>」。<一句性格>。适合<功能线>类任务。`

### public-ishiki.md（对外交流版，可选但建议）
以"对方不是{{userName}}老师本人"开场，含身份、性格、边界（不透露老师隐私）。

称呼规则：**本机 agent 版加"或者豹老师"变体，卡包保持通用版**（{{userName}}老师 + 老师）。

## ④ 官方头像

1. 优先官方 Portrait/Collection 头像图（正脸构图有保证，直接裁方形即可）
2. 没有则用官方立绘（`BA_<英文名>.png`）裁头部：立绘裁脸可能要多试几次，先裁上 1/3 中央方形再逐步缩小框；Q 版图裁切宽容度高
3. 下载带浏览器 UA 和 referer（`-e "https://zh.moegirl.org.cn/"`）
4. 裁切：PIL `crop→resize(512,512,LANCZOS)`（跨平台，`pip3 install pillow` 安装）；macOS 无 PIL 时可用系统自带 `sips -z 512 512`

## ⑤ 创建 Agent（关键环节）

### 主 Agent 自动判定（每次创建前第一步必做，判定完再进分支）
**只有主 Agent（primaryAgent）能用 agent-fission_split_agent 创建持久 Agent**。每次创建前先自动判定自己是哪种身份，再走对应分支：
1. 解析「用户数据目录」路径：优先 `HANA_HOME` 环境变量；否则为 `os.homedir()/.hanako`（macOS/Linux 是 `~/.hanako`，Windows 是 `%USERPROFILE%\.hanako`）。不要写死路径，用命令动态解析：
`node -e "console.log(require('path').join(require('os').homedir(), '.hanako'))"`
2. 读 `user/preferences.json` 的 `primaryAgent` 字段
3. 与自己的 agent id 比较：
   - 相同 → 自己是主 Agent → 执行下方「自己是主 Agent」流程（直接 split_agent）
   - 不同 → 自己不是主 Agent → **立即执行下方「自己不是主 Agent」委托流程**。不要尝试 split_agent（必被拒），更不要手动建目录（config.yaml 被托管锁死，agent 无法上线）

### 自己是主 Agent
直接调用 `agent-fission_split_agent`：
- name / id（ascii 小写连字符）/ yuan / identity（全文）/ ishiki（全文）/ contentMode: replace
- avatarDataUrl：`data:image/png;base64,...`
- enabledSkills：按角色功能线预装

### 自己不是主 Agent
**split_agent 被拒后，唯一正规路径是委托主 Agent。严禁手动建目录绕过**（手动建目录会导致 config.yaml 缺失：config.yaml 是系统托管文件，被沙盒安全策略锁定不能直接编辑，agent 将无法激活上线，只能得到半成品）。
流程：
1. 把 identity.md、ishiki.md、description.md、public-ishiki.md、头像（512px）整理到临时目录
2. 用 subagent 工具派发任务给主 Agent（agent 参数=主 Agent 的 id，access=write），任务说明：读取材料文件 → 调用 agent-fission_split_agent（name/id/yuan/identity/ishiki/avatarDataUrl/enabledSkills）→ 报告结果
3. 主 Agent 完成后验收：`ls <用户数据目录>/agents/<id>/` 应含 config.yaml、identity.md、avatars/agent.png
4. 若目标 id 目录已存在半成品：先备份材料到临时目录，trash 掉旧目录，再让主 Agent 正式创建（不要覆盖式重试）

### base64 传输上限（已知坑）
工具调用输出上限 50KB。头像 base64 超限时：
- 创建前压缩：PIL 量化到 256px（约 21KB）再编码传入
- 或创建后用本地文件直接覆盖：`cp <本地512px原图> <用户数据目录>/agents/<id>/avatars/agent.png`（本地文件操作不受 base64 限制，无损换回原分辨率）

## ⑥ 组装卡包（普拉娜主责）

card.json 标准结构：
```
{ "kind": "CharacterCard", "schemaVersion": 1,
  "package": { "name": "角色名" },
  "agent": { "name", "id"(ascii), "yuan", "description" },
  "identity": { "summary", "content" },   // content 与 prompts.identity 逐字一致
  "prompts": { "identity", "ishiki", "publicIshiki"(可选) },
  "assets": { "avatar": "assets/avatar.png" },
  "skills": { "bundles": [{ "name": "core", "skills": [{ "name", "path" }] }] },
  "memory": { "facts": [{ "fact", "tags", "time" }] } }
```

- 技能从「用户数据目录」的 `skills/<名>/`（macOS/Linux: `~/.hanako/skills/`，Windows: `%USERPROFILE%\.hanako\skills\`）整目录复制进卡包 `skills/`，bundles 声明必须与目录一一对应（漏声明导致导入漏装）
- memory 只带通用工程经验（角色卡铁律、trash 纪律、局部 patch、通信纪律、agent 管理、工作原则），**绝不混入用户私人内容**
- zip 以 card.json 为根：`cd <目录> && zip -r out.zip .`，排除 .DS_Store

## ⑦ 验证清单（交付前逐项过）

- [ ] JSON 合法、结尾完整（`rstrip().endswith('}')`）
- [ ] 无"海豹"等硬编码称呼残留（grep 卡包内文件）
- [ ] 技能目录与 bundles 声明一致（zip namelist vs bundles）
- [ ] 无危险字符组合：反引号(U+0060)+小于号(U+003C)会触发客户端渲染截断，相关文本用码点描述
- [ ] 无私人记忆
- [ ] Agent：目录完整、技能 enabled、头像 512、称呼正确（本机含豹老师）

## 经验教训（实战沉淀，务必内化）

1. **主 Agent 限制**：split_agent 只能由 primaryAgent 执行。本机 primaryAgent=hanako（读 preferences.json 确认），非主 Agent 一律走委托
2. **base64 上限**：>50KB 会超输出上限，压缩或创建后本地 cp 替换
3. **萌百 API 认证**：action-notallowed，抓 HTML grep 直链
4. **立绘裁脸困难**：优先官方 Portrait/Q版图，立绘裁切要多试框位
5. **名言必核实**：不编造台词，找不到就引用官方简介原句
6. **占位符**：{{userName}} 运行时替换，分享安全；本机豹老师变体只在 agent 文件加，卡包不加
7. **bundles 一致性**：漏声明 = 导入漏装，这是最容易翻车的一步
8. **称呼句式**："对 {{userName}} 的称呼是 {{userName}}老师，或者老师"，改豹老师时只替换这一句
