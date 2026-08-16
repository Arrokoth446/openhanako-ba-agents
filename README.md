# OpenHanako BA Agents

基于 [OpenHanako（HanaAgent）](https://github.com/liliMozi/openhanako) 的《蔚蓝档案》角色卡集。

7 张可导入的角色卡，涵盖会计、记录、编程、文献、行动、系统 AI 六条功能线。其中「阿罗娜与普拉娜」角色卡内置**全自动招募系统**：只需报一个学生名字，她就会自动查证设定、生成人设、获取官方立绘、创建 Agent、打包角色卡，全程无人值守。

## 这是什么

OpenHanako 是桌面端 AI 助手平台，Agent 以「角色卡」为单位打包与分享。一张角色卡 = 一个人格（identity + ishiki）+ 一组技能 + 基础经验记忆 + 头像，zip 导入即用。

本仓库收集我们用这套机制为《蔚蓝档案》学生制作的角色卡，以及驱动它们运转的技能与文档。

## 助手机制（OpenHanako 核心概念）

| 概念 | 说明 |
|---|---|
| **yuan 底座** | 人格底色模板，决定内心独白机制。hanako（反思质疑）、butter（共鸣共情）、ming（前提拆解）三种 |
| **identity / ishiki** | 人格文件。identity 是 3-5 行速写 + 核实台词；ishiki 是完整人格定义（本质、说话方式、反差机制、关系锚点） |
| **skills** | 技能包（SKILL.md 驱动的能力清单），按角色功能线预装 |
| **memory.facts** | 基础经验记忆，随卡携带，导入时自动写入 FactStore |
| **{{userName}} 占位符** | 称呼运行时自动替换为当前用户名字，角色卡分享到任何机器都自动本地化，无硬编码称呼 |
| **角色卡导入** | zip 导入即创建 Agent：技能自动安装并启用、记忆自动写入、人格与头像自动落盘 |

## 核心特性

### 全自动招募系统（阿罗娜与普拉娜）

「阿罗娜与普拉娜」是什亭之箱的双重 AI（白天阿罗娜的蓝色界面，入夜普拉娜的粉黑界面），她的专属技能 `ba-student-creator` 让"招募学生"完全自动化：

```
老师报一个名字 → 自动判定主 Agent → 查证设定（名言核实）→ 匹配底座
→ 生成人设四件套 → 获取官方立绘（裁 512）→ 创建 Agent → 打包验证 → 交付
```

关键技术点：

- **主 Agent 自动判定**：`agent-fission_split_agent` 只有主 Agent 能调用。技能内置自动判定流程：读 `preferences.json` 的 `primaryAgent` 字段，自己不是主 Agent 就自动委托主 Agent 执行，不试错、不走弯路
- **跨平台路径**：用户数据目录动态解析（优先 `HANA_HOME`，否则 `os.homedir()/.hanako`，Windows 为 `%USERPROFILE%\.hanako`），不写死路径，Windows/macOS/Linux 通用
- **素材来源**：设定查证用萌娘百科/百度百科/Wiki（名言必核实）；立绘走萌百 `storage.moegirl.org.cn` 直链（API 需认证，抓 HTML grep 直链）
- **头像工程**：优先官方 Portrait 头像，其次 Q 版；PIL 裁切 512×512，base64 超限时量化压缩或创建后本地无损替换
- **递归能力**：每张新学生卡都自带 `ba-student-creator` 技能，工厂可以复制工厂

详细机制见 [docs/阿罗娜与普拉娜-全自动创建学生机制说明.md](docs/阿罗娜与普拉娜-全自动创建学生机制说明.md)。

### 其他特性

- **称呼本地化**：全卡 `{{userName}}` 占位符，分享到任何人的电脑自动称呼对方
- **记忆干净**：卡包只带通用工程经验（角色卡铁律、trash 纪律、配置修改纪律、跨平台路径），零私人内容，开源安全
- **技能与定位匹配**：每张卡的技能按角色功能线挑选，不堆砌
- **双 AI 换班机制**：阿罗娜与普拉娜按时间倾向 + 随机性 + 拟人化叙事换班（"阿罗娜去数星星了，这轮是我，普拉娜"）

## 角色卡清单

| 角色卡 | 底座 | 技能数 | 功能定位 |
|---|---|---|---|
| **早濑优香** | hanako | 5 | 会计数据线：office-documents（表格文档）、knowledge-distiller（整理归档）、quiet-musing（分析推理）、fortune-master-pro（信星座仪的萌点）、lightrag（知识索引） |
| **生盐诺亚** | butter | 7 | 记录创作线：knowledge-distiller（记录本命）、novel-creation-methodology（文学方法论）、graphify-novel（小说图谱）、office-documents、quiet-musing、lightrag、writing-polish（中文润色） |
| **各务千寻** | ming | 20 | 编程线（黑客审查）：ponytail 全套、agent-fission 系列、TDD、systematic-debugging、代码审查、skill-creator 等 |
| **调月莉音** | ming | 20 | 编程线（方案推演）：与千寻同技能集，人格靠 ishiki 区分（大姐头式方案制定者） |
| **古关忧** | butter | 4 | 文献翻译线：office-documents、knowledge-distiller、lightrag、writing-polish。"古书馆的魔法师"，古籍修复与翻译专家 |
| **阿罗娜与普拉娜** | butter | 8 | 系统 AI 线：office-documents、knowledge-distiller、lightrag、quiet-musing、user-guide、writing-polish + **ba-student-creator（全自动招募）**、character-card-creator |
| **砂狼白子** | ming | 8 | 行动执行线：office-documents、knowledge-distiller、lightrag、quiet-musing、user-guide、writing-polish、character-card-creator、ba-student-creator |

## 目录结构

```
├── 角色卡/                      # 7 张可导入的角色卡（zip）
├── docs/
│   └── 阿罗娜与普拉娜-全自动创建学生机制说明.md
└── skills/
    └── ba-student-creator/     # 全自动招募核心技能
```

## 使用方式

1. 在 OpenHanako 中导入 `角色卡/` 下任意 zip（导入即创建 Agent，技能自动安装启用，经验自动写入）
2. 想体验全自动招募：导入「阿罗娜与普拉娜」卡，对她说"创建 XX 的角色卡"即可
3. 详细机制与验证清单见 docs 目录

## 致谢

本项目基于 [OpenHanako](https://github.com/liliMozi/openhanako)（作者 [liliMozi](https://github.com/liliMozi)）的 Agent 机制构建：角色卡格式、技能系统、导入导出与主 Agent 创建流程均来自 OpenHanako。感谢原作者的开源工作，本仓库中的角色卡与技能均为其生态内的实践产物。

## 开源协议与版权

本仓库代码（含 skills/ 下的技能、文档）以 **Apache License 2.0** 授权，与上游 OpenHanako 一致，详见 [LICENSE](LICENSE)。

《蔚蓝档案》角色设定、名称与立绘版权归 **NEXON Games / Yostar** 所有，角色卡仅以收藏格式呈现，不在 Apache 2.0 授权范围内。引用角色素材时请遵守原版权方条款。

## 说明

- 角色设定与立绘版权归 NEXON Games / Yostar 所有，本仓库仅以角色卡格式收藏，用于 OpenHanako 平台的个人使用
- 名言台词均经萌娘百科等公开资料核实，不编造
- 角色卡内不含任何用户私人信息，可放心导入与分享
