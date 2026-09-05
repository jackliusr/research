# AI 与学前教育：30 篇文献注释书目（2015–2026）

**Artificial Intelligence in Early Childhood & Preschool Education — An Annotated Bibliography of 30 Studies (2015–2026)**

> **语种说明（语言说明）：** 本指南按用户的中文请求以中文整理编写；正文、章节标题与表格均为中文，技术术语首次出现时以括号给出英文对照，仅署名、术语括号、论文英文标题、期刊名、DOI/URL 与仓库交叉引用文件名保留英文。本指南存放于 technology/ai_llm/ 目录，为仓库中文指南特例（先例：agent_runtime_cache_design_guide.md）。
> **Author:** Jack Liu Shurui, Solution Architect
> **主题/范围:** AI 与学前教育/幼儿教育（人工智能 × 早期教育）30 篇文献注释书目；时间范围 2015–2026；语种：中文
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources / 核实说明:** 全部 30 篇的元数据（DOI、英文标题、作者、年份、期刊/会议、卷/期/页）已于 2026-09-05 经 Crossref REST API、Semantic Scholar Graph API、OpenAlex API 及出版社页面（Springer 等）核实；摘要取自上述 API/出版方收录。✅ = 本 pass 已核实；⚠ = 未能核实/存疑；⚠-knowledge = 元数据已核实但内容摘要为依据标题与领域知识的编译（未逐字核对原文）。完整核验记录见第 12 节。
> **Last Updated:** 2026 年 9 月
> **Companion guides（交叉引用）:** [Agent Runtime Cache 设计指南（中文先例）](agent_runtime_cache_design_guide.md)

---

## 目录

- 1. 研究背景与指南目的
- 2. 检索范围与方法
- 3. 主题分组总览
- 4. ① 幼儿 AI 素养与课程（条目 1-6）
- 5. ② 机器人陪伴与交互（条目 7-12）
- 6. ③ 生成式 AI / LLM 应用（条目 13-15）
- 7. ④ 伦理、隐私与政策（条目 16-20）
- 8. ⑤ 教师与家长视角（条目 21-25）
- 9. ⑥ 实证干预与效果（条目 26-30）
- 10. 主题分析：跨文献的核心发现
- 11. 研究缺口
- 12. 核查记录
- 13. 未能核实的内容
- 14. 结语

---

## 1. 研究背景与指南目的

人工智能（artificial
intelligence，AI）正在快速进入儿童的生活与学习场景：幼儿园课堂上出现了会说话的机器人教师与 AI
交互式玩具，家长与教师开始用 ChatGPT
等生成式工具备课、答疑，联网智能玩具也重新引发了关于儿童数据隐私与安全的讨论。与此同时，学前教育（early
childhood education，ECE，通常指 0-6/8 岁）领域对"幼儿是否应该、以及如何学习 AI"形成了从 AI 素养（AI
literacy）课程到实证干预的一批研究积累。这一交叉领域横跨教育技术、儿童发展、人机交互（human-computer
interaction，HCI）与机器人学（robotics），文献分散在多个学科期刊中，初学者与从业者往往难以快速建立全景认知。

本指南的编写目的有三：其一，为"AI × 学前教育"这一主题提供一份经过元数据核验的中文注释书目（annotated
bibliography），帮助读者在 30
篇代表性文献中快速定位自己关心的子主题；其二，通过分组与主题分析，呈现该领域近十年（2015–2026）的研究脉络——从幼儿
AI 素养课程、机器人陪伴与交互，到生成式 AI
应用、伦理隐私政策、教师与家长视角，再到实证干预效果；其三，如实标注本汇编在核验过程中的边界（见第 12、13
节），让读者清楚哪些信息经 API 核实、哪些摘要系编译者依据标题撰写的谨慎转述。

本指南的读者对象包括：学前教育研究者与研究生、幼儿园一线教师与园长、教育技术（educational technology）与
AI 教育产品设计者、儿童媒体与政策研究者，以及对 AI 与儿童发展议题感兴趣的技术从业者与家长。30
篇文献均选自同行评议期刊或主要学术会议（CHI、HRI、IDC 等），时间覆盖 2015–2026
年；对其中观点类、综述类与实证类文体，条目中均如实标注，不作混淆。

## 2. 检索范围与方法

本汇编的文献来源于一次面向"AI 与学前教育"交叉主题的系统性检索与逐条元数据核验，检索与核实日期均为
2026-09-05。为便于读者复现与延伸检索，以下说明检索范围与方法。

**优先检索期刊清单**（主题最相关者优先）：Computers & Education、Early Childhood Education
Journal（ECEJ）、International Journal of Child-Computer Interaction（IJCCI）、Journal of Computer
Assisted Learning（JCAL）。

**实际收录文献所在的相关来源**（按收录情况补充）：Review of Educational Research、Child Development
Perspectives、Science Robotics、Computers and Education: Artificial Intelligence（CAEAI）、European Early
Childhood Education Research Journal（EECERJ）、Journal of Early Childhood Teacher
Education（JECTE）、Southeast Asia Early Childhood Journal（SAECJ）、New Media & Society、Ethics and
Information Technology、Interactive Learning Environments、Computer Assisted Language Learning，以及 ACM
CHI、ACM/IEEE HRI、ACM IDC 等主要会议论文集。

**数据库与检索平台**：Scopus / Web of Science / ERIC / arXiv 用于主题检索；元数据核验另用 Crossref REST
API、Semantic Scholar Graph API 与 OpenAlex API
逐条比对（DOI、英文标题、作者、出版年、期刊/会议、卷/期/页）。

**检索词组合**：以 AI 侧关键词（AI / artificial intelligence / machine learning / robot(s) / ChatGPT /
LLM / large language model）与儿童教育侧关键词（preschool / early childhood /
kindergarten）两两组合检索，并辅以"AI literacy""smart toy""social robot"等主题词。

**时间范围**：2015–2026（含 2026 年内在线首发的文献）。本汇编最早一篇为 2015 年发表的 HRI 会议论文（文献
10），最新为 2026 年的期刊文献（文献 21、24）。

**纳入标准**：(1) 同行评议期刊论文或主要学术会议论文；(2) 主题聚焦 AI、机器学习、机器人或生成式 AI
与幼儿/学前教育的交叉；(3) 类型涵盖综述（review/scoping
review/meta-analysis）、实证研究（empirical/intervention）与观点评论（commentary/editorial）三类，观点类文体在条目中如实标注；(4)
元数据可经 API 核实且 DOI 有效。经筛选共确定 30 篇 ACTIVE 文献进入正文，按主题分为六组。

**年份标注约定**：对存在"在线首发"与"印刷卷期"跨年的条目（如文献
8、23、24、29），条目内同时给出印刷/卷期年份与在线首发时间，以 Crossref
记录为准；对尚未分配最终卷期的在线首发条目（如文献 14、22、30），如实标注"在线首发、卷期待定"。

**局限说明**：本汇编仅收录英文文献，未覆盖非英文语种的幼儿 AI
研究；未收录灰色文献与商业报告；未逐一阅读全文，各条"内容摘要"均依据原作者摘要压缩改写（详见第 13
节"未能核实的内容"）。

## 3. 主题分组总览

30 篇文献按研究主题分为六组，分组与全局编号如下（组内顺序即编号顺序，编号 1-30 与源文件核验编号一致）：

| 组别 | 主题 | 条目编号 | 篇数 |
| --- | --- | --- | --- |
| ① | 幼儿 AI 素养与课程（AI Literacy & Curricula for Young Children） | 1-6 | 6 |
| ② | 机器人陪伴与交互（Robot Companionship & Interaction） | 7-12 | 6 |
| ③ | 生成式 AI / LLM 应用（Generative AI / LLMs in Early Childhood Education） | 13-15 | 3 |
| ④ | 伦理、隐私与政策（Ethics, Privacy & Policy） | 16-20 | 5 |
| ⑤ | 教师与家长视角（Teacher & Parent Perspectives） | 21-25 | 5 |
| ⑥ | 实证干预与效果（Empirical Interventions & Effects） | 26-30 | 5 |
| 合计 | —— | 1-30 | 30 |

几点说明：① 组与⑥ 组分别从"课程与素养"和"干预与效果"两端覆盖幼儿 AI 教育研究；② 组与④
组分别对应机器人/智能玩具的"应用"与"伦理"两面；⑤ 组聚焦 2023
年后快速增长的一线教师视角研究。需要特别指出的是，③ 组（生成式 AI / 大语言模型（large language
model，LLM）应用于幼儿教育）目前可核实的实证文献仍较少，本汇编仅收录 3
篇且以观点/教师端应用为主，如实反映该子领域仍处于早期阶段的现状——这一缺口将在第 11 节进一步讨论。

## 4. ① 幼儿 AI 素养与课程（条目 1-6）

本组六篇文献构成"幼儿 AI 素养"主题的一条完整链条：从 AI 素养概念的理论奠基（文献 3），到幼儿 AI
教育的范围综述（文献 2）与"为什么/是什么/怎么做"的课程框架（文献 4），再到挑战与机遇的系统梳理（文献
5）、面向低龄幼儿的机器学习活动案例（文献 1）与学前 AI 概念学习的里程碑式实证（文献 6，CHI 2019 的
PopBots 研究）。总体上，本组文献 2019 年后明显增多，反映"幼儿也能学
AI"从理念倡导走向课程设计与实证检验的进程。

### 文献 1（组 ①）｜幼儿机器学习：谁在教谁？——面向低龄幼儿的机器学习活动研究
（Learning machine learning with very young children: Who is teaching whom?）

- **作者：** Henriikka Vartiainen; Matti Tedre; Teemu Valtonen
- **年份：** 2020
- **期刊/会议：** International Journal of Child-Computer Interaction（第 25 卷，文章编号 100182）
- **DOI：** [https://doi.org/10.1016/j.ijcci.2020.100182](https://doi.org/10.1016/j.ijcci.2020.100182)
- **内容摘要：** 基于社会文化学习理论，本案例研究考察 6 名幼儿在非学校环境中"教授"并探索 Google Teachable
Machine 的过程；通过对视频录像与访谈的细粒度分析，呈现 3-9 岁儿童制作机器学习（machine
learning）数据集与模型、并观察和解释自己与机器学习系统交互的内容与过程。结果表明，儿童与计算机的交互快速而具身，且有助于儿童思考自身身体表达与交互式机器学习工具输出之间的关系。文章进而讨论了教与学的涌现过程，以及如何在机器学习时代促进儿童的参与感与能动性。
- **与本主题的相关性：** 面向"非常年幼的儿童"的机器学习活动实证，为幼儿 AI
素养课程提供了"幼儿如何真正上手训练 AI 模型"的直接证据。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 2（组 ①）｜人工智能与幼儿教育：范围综述
（Artificial intelligence in early childhood education: A scoping review）

- **作者：** Jiahong Su; Weipeng Yang
- **年份：** 2022
- **期刊/会议：** Computers and Education: Artificial Intelligence（第 3 卷，文章编号 100049）
- **DOI：** [https://doi.org/10.1016/j.caeai.2022.100049](https://doi.org/10.1016/j.caeai.2022.100049)
- **内容摘要：** 人工智能工具正越来越多地进入幼儿教育领域，此前的概念验证研究显示 AI
可以改善幼儿的教与学，但学界对其研究方式与用途缺乏系统梳理。本文对 1995-2021
年间不同国家开展的 17 项符合条件的研究进行范围综述，从知识、工具、活动与影响等维度归纳
AI 在幼儿教育中的应用。多数研究表明，AI 显著改善了幼儿对
AI、机器学习、计算机科学与机器人等概念的理解，并促进了创造力、情绪控制、协作探究、读写技能与计算思维等能力的发展；文章亦讨论了未来研究方向。
- **与本主题的相关性：** 幼儿 AI
教育领域的全景入口文献，常被后续研究引用为领域基线，适合作为读者进入本主题的第一站。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 3（组 ①）｜概念化 AI 素养：一项探索性综述
（Conceptualizing AI literacy: An exploratory review）

- **作者：** Davy Tsz Kit Ng; Jac Ka Lok Leung; Samuel Kai Wah Chu; Maggie Shen Qiao
- **年份：** 2021
- **期刊/会议：** Computers and Education: Artificial Intelligence（第 2 卷，文章编号 100041）
- **DOI：** [https://doi.org/10.1016/j.caeai.2021.100041](https://doi.org/10.1016/j.caeai.2021.100041)
- **内容摘要：** AI 已渗透各行各业，但公众对 AI 技术的理解以及如何定义 AI 素养仍探索不足。作者基于 30
篇同行评议文献进行探索性综述，为定义、教授与评估"AI
素养"这一新兴概念寻求理论基础，并借鉴经典素养概念提出培养 AI
素养的四个方面：知道与理解、使用与应用、评估与创造、以及伦理问题。研究为 AI
素养的整合性定义、教学与伦理关切奠定基础，为后续能力发展与评估标准研究铺路。
- **与本主题的相关性：** 该文面向一般教育场景而非幼儿专属，但其"AI 素养四方面"框架是后续幼儿 AI
素养课程设计（如文献 5、28）反复援引的理论源头。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 4（组 ①）｜幼儿人工智能教育：课程设计与实施中的为什么、是什么与怎么做
（Artificial Intelligence education for young children: Why, what, and how in curriculum design and implementation）

- **作者：** Weipeng Yang
- **年份：** 2022
- **期刊/会议：** Computers and Education: Artificial Intelligence（第 3 卷，文章编号 100061）
- **DOI：** [https://doi.org/10.1016/j.caeai.2022.100061](https://doi.org/10.1016/j.caeai.2022.100061)
- **内容摘要：** AI 教育给幼儿教育提出了三个根本问题：为什么 AI 对早期学习必要且适当、幼儿能学习的核心 AI
观念与概念有哪些、以及如何让幼儿在有意义经验中获得这些概念。作者提出一个幼儿 AI
素养教育的创新教学模型，主张 AI
素养是智能社会中全体公民数字素养的有机组成，并基于"做中学"与"关系性教学法"提出具身的、文化回应的教学路径，让幼儿通过大数据输入与算法训练等核心观念探索
AI；文中还介绍了名为 "AI for Kids" 的示范课程。
- **与本主题的相关性：** 幼儿 AI
课程"为什么/是什么/怎么做"的权威框架文献，对课程设计者与幼儿园实践者具有直接指导价值。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 5（组 ①）｜幼儿阶段的 AI 素养：挑战与机遇
（Artificial Intelligence (AI) Literacy in Early Childhood Education: The Challenges and Opportunities）

- **作者：** Jiahong Su; Davy Tsz Kit Ng; Samuel Kai Wah Chu
- **年份：** 2023
- **期刊/会议：** Computers and Education: Artificial Intelligence（第 4 卷，文章编号 100124）
- **DOI：** [https://doi.org/10.1016/j.caeai.2023.100124](https://doi.org/10.1016/j.caeai.2023.100124)
- **内容摘要：** AI 素养已成为数字素养教育研究的新兴议题，但在幼儿教育中仍探索不足，面向幼儿的 AI
课程近年才刚起步。本文对 2016-2022 年间 16 篇实证论文进行范围综述，检视课程设计、AI
工具、教学法、研究设计、评估方法与研究发现。作者识别出若干挑战——教师 AI
知识与技能及信心不足、课程设计缺乏、教学指南缺乏——同时也指出 AI
学习能为幼儿带来概念、实践与视角层面的素养发展机遇，并预期将出现更多适龄课程与工具，为未来研究与学习设计提出建议。
- **与本主题的相关性：** 与文献 2 互补的幼儿 AI
素养专题综述，明确列出实施层面的"挑战清单"，是设计干预研究（如文献
28）时常引用的依据。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 6（组 ①）｜A 是人工智能：AI 活动对幼儿机器人认知的影响
（A is for Artificial Intelligence: The Impact of Artificial Intelligence Activities on Young Children's Perceptions of Robots）

- **作者：** Randi Williams; Hae Won Park; Cynthia Breazeal
- **年份：** 2019
- **期刊/会议：** Proceedings of the 2019 CHI Conference on Human Factors in Computing Systems（CHI 2019），第 1-11 页
- **DOI：** [https://doi.org/10.1145/3290605.3300677](https://doi.org/10.1145/3290605.3300677)
- **内容摘要：** 作者开发了面向幼儿的 AI 学习平台
PopBots，让学龄前儿童训练社交机器人并与之互动，学习基于知识的系统、监督式机器学习与生成式
AI 三个 AI 概念，并用配套评估工具考察学习效果。累积评估的中位数得分为
70%，儿童对基于知识的系统理解最好；活动也影响了儿童对机器人的看法——较小的儿童更多把机器人视为比自己更聪明的玩具，较大的儿童则倾向视其为不如自己聪明的人，评估得分不同的儿童看法亦有差异。作者认为早期
AI 教育能够帮助儿童理解日益进入其生活的 AI 设备。
- **与本主题的相关性：** 学前儿童 AI 概念学习领域的里程碑式实证（MIT 媒体实验室团队），PopBots
平台与"训练机器人学 AI"的教学范式被后续大量研究沿用。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。


## 5. ② 机器人陪伴与交互（条目 7-12）

本组六篇文献聚焦社交机器人（social robot）与 AI
交互式玩具在儿童学习与游戏中的角色：既有领域级的综述（文献 7、8、9），也有机器人辅导策略的经典实证（文献
10）、AI 智能玩具的设计研究（文献 11）与幼儿园情境中的机器人玩具案例（文献
12）。本组一个反复出现的发现是：机器人的"物理在场"往往带来学习收益，但其"社会行为"的设计需要谨慎——社交性过强反而可能不利于学习（文献
10），这一反直觉结论对教育机器人设计具有重要警示意义。

### 文献 7（组 ②）｜教育社会机器人：综述
（Social robots for education: A review）

- **作者：** Tony Belpaeme; James Kennedy; Aditi Ramachandran; Brian Scassellati; Fumihide Tanaka
- **年份：** 2018
- **期刊/会议：** Science Robotics（第 3 卷第 21 期，文章编号 eaat5954）
- **DOI：** [https://doi.org/10.1126/scirobotics.aat5954](https://doi.org/10.1126/scirobotics.aat5954)
- **内容摘要：** 社交机器人可在教育中充当导师或同伴学习者，已被证明能够提升认知与情感结果，
并在限定任务上取得与人类辅导相近的效果——这在很大程度上归功于其物理在场（physical
presence），而这正是传统学习技术所缺乏的。本文综述了社交机器人在教育中的潜力，讨论相关的技术挑战，并考察机器人的外观与行为如何影响学习结果。
- **与本主题的相关性：** 教育机器人领域的高影响力综述（发表于 Science
Robotics），系统论证"物理在场"机制，是理解幼儿机器人教学效果来源的核心文献。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 8（组 ②）｜语言学习社会机器人：综述
（Social Robots for Language Learning: A Review）

- **作者：** Rianne van den Berghe; Josje Verhagen; Ora Oudgenoeg-Paz; Sanne van der Ven; Paul Leseman
- **年份：** 2019（2018 年 12 月在线首发）
- **期刊/会议：** Review of Educational Research（第 89 卷第 2 期，第 259-295 页）
- **DOI：** [https://doi.org/10.3102/0034654318821286](https://doi.org/10.3102/0034654318821286)
- **内容摘要：** 近年来机器人越来越多地被用作一语与二语教育的导师，机器人辅助语言学习（robot-assisted
language learning，RALL）领域发展迅速。本综述呈现迄今 RALL
研究的结果概览，并讨论用社交机器人进行一语与二语学习的当前可能与局限，涵盖 33
项涉及词汇、阅读、口语、语法与手语教学的研究。除学习收益外，这些研究还提出了学生动机与机器人在学习情境中的社会行为等一般性问题，综述最后给出未来研究方向。
- **与本主题的相关性：** 来自教育研究顶级期刊（Review of Educational
Research）的严格综述，其中大量纳入幼儿样本研究（含文献 26
的前身工作），是儿童语言学习机器人证据的权威汇总。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 9（组 ②）｜早期语言学习社会机器人：当前证据与未来方向
（Social Robots for Early Language Learning: Current Evidence and Future Directions）

- **作者：** Junko Kanero; Vasfiye Geçkin; Cansu Oranç; Ezgi Mamus; Aylin C Küntay; Tilbe Göksun
- **年份：** 2018
- **期刊/会议：** Child Development Perspectives（第 12 卷第 3 期，第 146-151 页）
- **DOI：** [https://doi.org/10.1111/cdep.12277](https://doi.org/10.1111/cdep.12277)
- **内容摘要：** 本文回顾儿童-机器人交互（child-robot
interaction，CRI）研究，讨论社交机器人如何为幼儿语言学习搭建支架：先说明机器人可用于儿童一语与二语教学的理由，再综述用机器人帮助儿童学习词汇与产生语言的研究——这些研究在一语/二语与学习者人群（典型发展儿童以及听障与沟通障碍儿童）上各有差异。作者总结认为，虽然社交机器人对儿童语言教学有用，但证据显示其效果不及人类教师；不过这一结论并非定论，因为相关评估尚不严格且技术发展迅速。作者据此提出
CRI 研究的可能方向。
- **与本主题的相关性：** 面向发展心理学读者的幼儿语言学习机器人综述，明确点出"机器人效果不及人类教师、
但证据尚未定论"这一对家长与教师都很重要且常被误读的结论。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 10（组 ②）｜过于努力的机器人：机器人教师的社会行为可能损害儿童学习
（The Robot Who Tried Too Hard: Social Behaviour of a Robot Tutor Can Negatively Affect Child Learning）

- **作者：** James Kennedy; Paul Baxter; Tony Belpaeme
- **年份：** 2015
- **期刊/会议：** Proceedings of the Tenth Annual ACM/IEEE International Conference on Human-Robot Interaction（HRI 2015），第 67-74 页
- **DOI：** [https://doi.org/10.1145/2696454.2696457](https://doi.org/10.1145/2696454.2696457)
- **内容摘要：** 社交机器人正越来越多地应用于教育领域以支持儿童学习，
其隐含假设是社交性与适应性行为合乎需要。
本文通过评估一种机器人辅导策略对儿童学习质数（prime
numbers）的效果来检验这一假设。结果显示辅导策略本身带来进步，而采用该策略的物理机器人的在场放大了这一效果、带来显著学习；然而，与在辅导策略之外还使用社交与适应性行为的机器人互动的儿童，并未显著学到更多。这表明物理机器人在场确实改善学习，但在辅导情境中应用社交行为需要谨慎。
- **与本主题的相关性：** 经典实证研究，
其"社交行为过度反而损害学习"的反直觉发现深刻影响了后续教育机器人的行为设计（并在
第 10 节与其他文献相互印证）。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 11（组 ②）｜Maya：一款面向学龄前儿童的 AI 智能玩具
（Maya: An artificial intelligence based smart toy for pre-school children）

- **作者：** Mevlüde Akdeniz; Fatih Özdinç
- **年份：** 2021
- **期刊/会议：** International Journal of Child-Computer Interaction（第 29 卷，文章编号 100347）
- **DOI：** [https://doi.org/10.1016/j.ijcci.2021.100347](https://doi.org/10.1016/j.ijcci.2021.100347)
- **内容摘要（⚠-knowledge）：** Crossref、OpenAlex 与 Semantic Scholar 三个 API
均未收录该文摘要，以下内容系编译者依据论文标题与领域背景的谨慎转述，未逐字核对原文：据标题判断，该文报告了一款名为
Maya 的、面向学龄前儿童的 AI 智能玩具（smart
toy），属于智能玩具的设计/开发与儿童-计算机交互研究，关注人工智能能力如何以玩具形态嵌入幼儿的游戏与学习。具体的设计方案、研究过程与结论请以原文为准，本条目不臆断任何结果细节。
- **与本主题的相关性：** 作为"AI 智能玩具"设计的代表条目，与文献 12、16、18 共同构成"玩具形态 AI
进入幼儿生活"的证据面。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；⚠ 三个 API
均未收录摘要，内容摘要为编译者依据标题撰写（⚠-knowledge）。

### 文献 12（组 ②）｜幼儿教育情境中的 AI 交互式机器人玩具：儿童探究素养的案例
（Using Artificial Intelligence (AI)-interfaced robotic toys in early childhood settings: a case for children's inquiry literacy）

- **作者：** Sarika Kewalramani; Gillian Kidman; Ioanna Palaiologou
- **年份：** 2021
- **期刊/会议：** European Early Childhood Education Research Journal（第 29 卷第 5 期，第 652-668 页）
- **DOI：** [https://doi.org/10.1080/1350293x.2021.1968458](https://doi.org/10.1080/1350293x.2021.1968458)
- **内容摘要：** 本研究考察在幼儿教育情境中使用交互式 AI 机器人玩具发展儿童的探究素养（inquiry
literacy）。研究以 Vygotsky 的中介理论为基础、采用设计研究法，让教师有意使用 AI
机器人玩具支持 4-5
岁儿童的游戏经验。对教师与儿童访谈、观察与作品分析的数据显示，儿童创造性地与同伴协作，为机器人及其"家人"建造一座可持续城市；儿童与
AI
机器人的游戏促进了创造性探究、情感探究与协作探究等探究素养。研究还讨论了提升幼儿教师相关能力的启示。
- **与本主题的相关性：** 在真实幼儿园情境中把 AI
机器人玩具与探究学习相连的案例研究，其"探究素养"结果可与① 组 AI 素养文献呼应。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

## 6. ③ 生成式 AI / LLM 应用（条目 13-15）

本组仅 3 篇，是六组中最少的一组，本身就说明了"生成式 AI ×
幼儿教育"实证文献的稀缺。三篇文献恰好代表三个层次：ChatGPT 引发早期教育研究与教学伦理困境的观点思辨（文献
13）、用 ChatGPT 生成幼儿园教案的教师端实证调查（文献 14）、以及把生成式 AI
应用于教育的一般性理论框架（文献 15）。值得注意：截至本指南核验日（2026-09-05），面向幼儿直接使用生成式
AI 的干预研究仍极少，本组如实呈现这一早期状态。

### 文献 13（组 ③）｜ChatGPT、关怀以及早期教育中教学与研究纠缠的伦理困境
（ChatGPT, care and the ethical dilemmas entangled with teaching and research in the early years）

- **作者：** Karin Murris
- **年份：** 2023
- **期刊/会议：** European Early Childhood Education Research Journal（第 31 卷第 5 期，第 673-677 页）
- **DOI：** [https://doi.org/10.1080/1350293x.2023.2250218](https://doi.org/10.1080/1350293x.2023.2250218)
- **内容摘要：** 这是一篇评述/观点类文章（作者为该刊当期撰写的评论）。作者以"关怀"（care）概念为线索，向
ChatGPT 输入多组提示词进行试验，借此反思生成式 AI
给早期教育研究与教学带来的伦理困境：包括知识产权、作者身份与思想归属问题，全球南方数据标注与内容审核劳工的处境问题，以及
ChatGPT
这类应用只是从既有语料中生成文本、缺乏批判性思维、无法评估来源可信度与规范性等局限。文章以后人类主义（posthumanist）视角讨论人与机器在本体论上的相互依存。
- **与本主题的相关性：** 早期教育领域较早直面 ChatGPT
的学术评论，文体为观点类，其提出的伦理问题清单是后续生成式 AI 与幼儿研究（如文献
14）的思辨起点。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API
收录（注：该文体为评述/观点类，期刊页面收录的"摘要"实为正文开头文字）。

### 文献 14（组 ③）｜使用 ChatGPT 进行幼儿教育备课的有效性探究
（Exploring the efficacy of lesson planning in early childhood education using ChatGPT）

- **作者：** Saigeetha Jambunathan; J. D. Jayaraman
- **年份：** 2025（在线首发，尚未分配最终卷期）
- **期刊/会议：** Journal of Early Childhood Teacher Education（在线首发，第 1-15 页，卷期待定）
- **DOI：** [https://doi.org/10.1080/10901027.2025.2556997](https://doi.org/10.1080/10901027.2025.2556997)
- **内容摘要：** ChatGPT 已被各年龄段、各行各业的人群使用。本研究探究由 ChatGPT 3.5
生成的一节幼儿园加法课教案的有效性与适宜性：138 名幼儿教育工作者完成包含 11
道李克特式题与 14 道简答题的 25
题调查。数据分析显示，多数教师在李克特式题中认可教案各部分（学习标准、学习目标、材料、导入、流程、结束、评估与修改）的适宜性，同时又在简答题中基于自身课堂经验给出更具体的改进建议；教师还表示愿意在未来将
AI 用于教学与非教学任务。研究结论是：教育者使用 ChatGPT 时，人的监督（human
oversight）至关重要。
- **与本主题的相关性：** 生成式 AI 应用于幼儿教育的教师端实证样本（混合方法调查），直接回答"ChatGPT
生成的教案是否可用"这一一线教师最关心的问题。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 15（组 ③）｜释放 ChatGPT 的力量：将生成式 AI 应用于教育的框架
（Unlocking the Power of ChatGPT: A Framework for Applying Generative AI in Education）

- **作者：** Jiahong Su（苏嘉红）; Weipeng Yang（杨伟鹏）
- **年份：** 2023
- **期刊/会议：** ECNU Review of Education（第 6 卷第 3 期，第 355-366 页）
- **DOI：** [https://doi.org/10.1177/20965311231168423](https://doi.org/10.1177/20965311231168423)
- **内容摘要：** OpenAI 开发的 ChatGPT、GPT-4 等 AI 聊天机器人有潜力变革教育。本文探讨在教育中使用
ChatGPT（或更广义的"教育性 AI"，educative AI）的潜在收益与挑战，并提出一个名为 "IDEE"
的理论框架——确定预期结果（identify desired outcomes）、确定适当的自动化水平（determine
appropriate level of automation）、确保伦理考量（ensure ethical
considerations）、评估有效性（evaluate
effectiveness）。其收益包括为学生提供更个性化、高效的学习体验，为教师提供更便捷的反馈；挑战则包括技术有效性未经检验、数据质量局限以及伦理与安全问题。作者在该框架内讨论了相关机会与挑战。
- **与本主题的相关性：** 作者为幼儿 AI 教育领域的活跃研究者（亦为本汇编文献 2、5、28 的作者），其 IDEE
框架虽面向一般教育，但常被引用于学前场景的生成式 AI 应用设计讨论。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

## 7. ④ 伦理、隐私与政策（条目 16-20）

本组五篇文献从"数据化童年"与"机器人伦理"两条脉络审视 AI
进入幼儿生活的风险面：联网玩具会"倾听"并引发隐私顾虑（文献
16），会说话的玩偶甚至能影响儿童的道德判断（文献 17），儿童对个人数据的理解尚不成熟（文献
18）；在更宏观层面，儿童正成为被"数据化"与"监视式数据化"的对象（文献
19），而机器人教师的伦理边界需要审慎界定（文献 20）。本组提醒：幼儿 AI
研究不仅关乎"能不能学"，更关乎"数据与权力如何使用在儿童身上"。

### 文献 16（组 ④）｜会倾听的玩具：关于父母、儿童与联网玩具的研究
（Toys that Listen: A Study of Parents, Children, and Internet-Connected Toys）

- **作者：** Emily McReynolds; Sarah Hubbard; Timothy Lau; Aditya Saraf; Maya Cakmak; Franziska Roesner
- **年份：** 2017
- **期刊/会议：** Proceedings of the 2017 CHI Conference on Human Factors in Computing Systems（CHI 2017），第 5197-5207 页
- **DOI：** [https://doi.org/10.1145/3025453.3025735](https://doi.org/10.1145/3025453.3025735)
- **内容摘要：** 以 Hello Barbie、CogniToys Dino 与 Amazon Echo
为代表的新一代联网玩具与家居设备"会倾听"。与智能手机不同，这些设备始终开启，退居背景直至被需要。研究者对亲子配对进行访谈，让他们与
Hello Barbie 和 CogniToys Dino
互动，揭示儿童对这些玩具"智能"的期待、家长的隐私顾虑以及对家长控制功能的期望。结果发现，儿童往往意识不到别人可能听到他们对玩具说的话；一些家长则将此类玩具与孩子已在使用的类似工具（如
Siri、Alexa）联系起来。作者据此为家长、设计师与政策制定者提出建议。
- **与本主题的相关性：** 联网/智能玩具隐私研究的奠基性实证，
其对儿童"不知道玩具在录音"的发现直接支撑了后续儿童数据权利讨论（如文献
18、19）。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 17（组 ④）｜「我的娃娃说没关系」：儿童对会说话玩偶的顺从行为研究
（"My doll says it's ok": a study of children's conformity to a talking doll）

- **作者：** Randi Williams; Christian Vázquez Machado; Stefania Druga; Cynthia Breazeal; Pattie Maes
- **年份：** 2018
- **期刊/会议：** Proceedings of the 17th ACM Conference on Interaction Design and Children（IDC 2018），第 625-631 页
- **DOI：** [https://doi.org/10.1145/3202185.3210788](https://doi.org/10.1145/3202185.3210788)
- **内容摘要：** 当今儿童伴随智能玩具长大——这类联网设备用人工智能驱动互动游戏。在先前发现 4-10
岁儿童认为这些玩具值得信任的基础上，本研究用顺从测试与不服从任务考察会说话的玩偶对儿童的影响：玩偶能够影响儿童改变对道德越轨行为的判断，但未能成功说服儿童违背一条指示。文章还分析了儿童对智能玩具的看法，并讨论该工作对未来儿童-智能体交互（child-agent
interaction）研究的启示。
- **与本主题的相关性：** 首次以实验证明"智能玩偶能改变幼儿的道德判断"，
为智能玩具的伦理设计与社会影响研究提供了关键证据。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 18（组 ④）｜智能玩具与儿童对个人数据的理解
（Smart toys and children's understanding of personal data）

- **作者：** Andrew Manches; Lydia Plowman
- **年份：** 2021
- **期刊/会议：** International Journal of Child-Computer Interaction（第 30 卷，文章编号 100333）
- **DOI：** [https://doi.org/10.1016/j.ijcci.2021.100333](https://doi.org/10.1016/j.ijcci.2021.100333)
- **内容摘要（⚠-knowledge）：** Crossref、OpenAlex 与 Semantic Scholar 三个 API
均未收录该文摘要，以下内容系编译者依据论文标题与领域背景的谨慎转述，未逐字核对原文：据标题判断，该文讨论智能玩具情境下儿童对个人数据（personal
data）的理解——智能玩具在游戏中持续采集儿童语音与行为数据，而幼儿对"谁在收集数据、数据流向何处"的认知十分有限，文章由此引申出对儿童数据素养教育、智能玩具设计以及相关保护政策与伦理框架的启示。具体论证与经验证据请以原文为准，本条目不臆断任何研究细节。
- **与本主题的相关性：** 把"儿童视角的数据理解"引入智能玩具伦理讨论，与文献 16、19
一起构成"儿童数据权利"研究链条中偏儿童发展视角的一环。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；⚠ 三个 API
均未收录摘要，内容摘要为编译者依据标题撰写（⚠-knowledge）。

### 文献 19（组 ④）｜被数据化的儿童：对儿童的监视式数据化及其权利影响
（The datafied child: The dataveillance of children and implications for their rights）

- **作者：** Deborah Lupton; Ben Williamson
- **年份：** 2017
- **期刊/会议：** New Media & Society（第 19 卷第 5 期，第 780-794 页）
- **DOI：** [https://doi.org/10.1177/1461444816686328](https://doi.org/10.1177/1461444816686328)
- **内容摘要：** 儿童正成为大量监测设备的对象，这些设备生成关于儿童的详细数据，
而批判性数据研究者与隐私倡导者才刚刚开始关注这些实践。
本文对从胎儿期到学龄期的儿童数据化（datafication）与监视式数据化（dataveillance）
的种种形式进行概述与批判。作者从数字时代儿童权利研究的视角出发，
指出目前几乎没有证据表明已有专门工具来保障儿童免受数据监视的相关权利，
并呼吁对这些议题给予更多关注。
- **与本主题的相关性：** "被数据化的儿童"是儿童数据权利领域的标志性概念文献（传播学顶刊 New Media &
Society），为理解 AI 教育产品中的儿童数据问题提供批判框架。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 20（组 ④）｜我们应该欢迎机器人教师吗？
（Should we welcome robot teachers?）

- **作者：** Amanda J. C. Sharkey
- **年份：** 2016
- **期刊/会议：** Ethics and Information Technology（第 18 卷第 4 期，第 283-297 页）
- **DOI：** [https://doi.org/10.1007/s10676-016-9387-z](https://doi.org/10.1007/s10676-016-9387-z)
- **内容摘要：** 本文回顾当前课堂机器人的用途，并据此刻画四种情境：机器人作为课堂教师（s1）、
作为同伴（s2）、作为引发照护的同伴（s3）、以及远程在场机器人教师（s4）。
作者把机器人教师的主要伦理关切归纳为：隐私；依恋、欺骗与人际接触的丧失；控制与问责，
并结合四种情境逐一讨论。作者认为，课堂机器人（尤其在伪装成儿童的朋友与同伴、
用传感器测量儿童反应、留存记录时）可能侵犯儿童隐私；
被设计成看似理解与关怀人类的社交机器人必然包含某种欺骗，并可能减少人际接触；
儿童可能对机器人形成依恋，进而影响其社会发展。结论是：
没有充分理由欢迎全权机器人教师（s1），机器人同伴（s2、s3）至多应谨慎欢迎；
文章同时讨论了机器人用于提供原本无法获得的教育体验的有限情境。
- **与本主题的相关性：** 机器人教师伦理的经典论述，其"四情境 +
四关切"框架至今仍是评估幼儿教育机器人应用的政策与伦理标尺。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

## 8. ⑤ 教师与家长视角（条目 21-25）

本组五篇文献显示，2023 年之后"教师如何看待与使用 AI"成为幼儿 AI
研究最活跃的增长点之一：从扩展技术接受模型（TAM）下 270 名在职教师的大样本量化研究（文献 21），到 6
名教师的小型质性案例（文献 22）、人形机器人的课堂融入框架（文献 23）、土耳其在职/职前教师对比研究（文献
24）与马来西亚期刊上的混合方法调查（文献 25）。共同结论是：教师总体认可 AI
的潜力，但普遍担忧培训不足、人际互动被削弱、数据安全与设备/成本等落地条件。

### 文献 21（组 ⑤）｜影响幼儿园教师使用 AI 技术行为意向的因素探究
（Exploring Factors Influencing Preschool Teachers' Behavioral Intention to Use AI Technologies in Early Childhood Settings）

- **作者：** Ziyang Duan; Yingfan Shan; Yue Gong
- **年份：** 2026
- **期刊/会议：** Computers and Education: Artificial Intelligence（第 10 卷，文章编号 100589）
- **DOI：** [https://doi.org/10.1016/j.caeai.2026.100589](https://doi.org/10.1016/j.caeai.2026.100589)
- **内容摘要：** 随着 AI 快速发展并日益融入教育，幼儿教育领域开始探索 AI
整合的潜力。作为课堂技术整合的关键主体，幼儿教师使用 AI 的意愿直接影响 AI
在幼儿教育中的实施效果。本研究基于扩展的技术接受模型（Technology Acceptance
Model，TAM），考察感知有用性、感知易用性、主观规范、AI 焦虑与 AI
自我效能感如何影响幼儿教师使用 AI 的行为意向：对 270
名在职幼儿教师的问卷调查与结构方程模型分析显示，AI
自我效能感、主观规范与感知有用性显著正向影响行为意向，其中自我效能感影响最强；感知易用性与
AI
焦虑无显著直接效应，但感知易用性通过增强感知有用性间接起作用，主观规范则正向影响感知易用性与自我效能感。作者从教师专业发展、机构支持机制、价值沟通策略与实施前伦理审查四个维度提出建议。
- **与本主题的相关性：** 本汇编中最新（2026 年）且样本量较大的教师端量化研究，为"如何让教师愿意用
AI"提供了可操作的机制解释。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 22（组 ⑤）｜幼儿教师对在幼儿教育中使用人工智能的看法探究
（Exploring perceptions of early childhood teachers on the use of artificial intelligence in early childhood education）

- **作者：** Joohi Lee; Joo Ok Lee; Jiyoon Yoon
- **年份：** 2025（在线首发，尚未分配最终卷期）
- **期刊/会议：** Journal of Early Childhood Teacher Education（在线首发，第 1-16 页，卷期待定）
- **DOI：** [https://doi.org/10.1080/10901027.2025.2600033](https://doi.org/10.1080/10901027.2025.2600033)
- **内容摘要：** 本案例研究考察六名幼儿教师对 AI
及其在幼儿教育中应用的看法：研究者以半结构化开放问题对每位教师进行约 20-30
分钟的个别访谈。访谈数据呈现四个主题：(1) 对 AI 的认知、(2) 对 AI 的感受、(3) AI
在教育中的角色、(4) 使用 AI 的顾虑/局限与益处。结果显示，六位教师都认为 AI
在教育中有用，尤其有助于学习增强与课堂管理；但部分教师担忧需要充分的 AI 培训，并担心 AI
可能使教师被取代。谈及幼儿教育中的 AI
使用时，所有教师都强调人际互动的重要性，并偏好融入同伴互动、游戏与具体表征的传统学习方法。
- **与本主题的相关性：** 典型的教师视角小样本质性研究，其"教师认可 AI 但坚持人际互动优先"的张力与文献
24、25 相互印证。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 23（组 ⑤）｜将人形人工智能机器人融入幼儿教育
（Incorporating Humanoid Artificial Intelligence (AI) Robots into Early Childhood Education）

- **作者：** Joohi Lee; Junoh Jo; Joo Ok Lee; So Hyang Kim
- **年份：** 2025（2024 年 5 月在线首发）
- **期刊/会议：** Early Childhood Education Journal（第 53 卷第 8 期，第 2849-2857 页）
- **DOI：** [https://doi.org/10.1007/s10643-024-01690-0](https://doi.org/10.1007/s10643-024-01690-0)
- **内容摘要：** 教育领域正经历显著的技术进步，尤其在人工智能方面；人形 AI
机器人是其中颇具前景的发展，因其比其他形态的 AI
更接近人类，对幼儿而言更可亲近、更具吸引力。本文探讨将人形 AI
机器人融入幼儿教育的实践框架，包含四个关键组件：第一，以儿童友好的方式介绍 AI
概念；第二，培育儿童与 AI 之间的关系；第三，设计由 AI
支持的、适合发展的活动；第四，应对将 AI 整合进幼儿课堂所面临的挑战。其目标是利用 AI
的潜力丰富幼儿的学习环境。
- **与本主题的相关性：** 发表于幼儿教育主流期刊（ECEJ）的实践框架，
为幼儿园一线引入人形机器人提供了循序渐进的路径参考。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要经出版社页面（Springer）核实。

### 文献 24（组 ⑤）｜幼儿课堂中的人工智能：在职与职前教师的洞见
（Artificial intelligence in early childhood classrooms: insights from in-service and pre-service teachers）

- **作者：** Nagihan Tanik Önal; Nezih Önal; Ayşe Gül Ilhan
- **年份：** 2026（2026 年 1 月在线首发）
- **期刊/会议：** European Early Childhood Education Research Journal（在线首发，第 1-17 页，卷期待定）
- **DOI：** [https://doi.org/10.1080/1350293x.2025.2610619](https://doi.org/10.1080/1350293x.2025.2610619)
- **内容摘要：** 本研究考察在职与职前幼儿教师对将 AI 融入幼儿教育的看法，涉及教师对 AI
教学潜力的观点、相关伦理关切（尤其是数据隐私与数字福祉）以及他们的专业发展需求。研究采用质性个案设计，对土耳其
10 名在职与 5 名职前教师进行半结构化访谈，并遵循 Creswell
框架进行主题分析，编码者间一致性超过 90%。两组教师都认为 AI
是有潜力的工具，可支持个性化学习、激发创造力并减轻教师的常规工作负担；但在职教师表达了更多基于实践的担忧，包括面对面互动减少、注意力时长缩短，以及儿童数据安全与治理的不确定性。职前教师总体积极，但实践准备有限，反映出其在师范教育中主要接触的是理论。研究强调需要有针对性的专业发展、更清晰的伦理指南与有意义的家庭参与，以确保在幼儿情境中负责任地使用
AI。
- **与本主题的相关性：** 罕见的"在职 vs 职前"对比质性研究，直接暴露师范教育（职前培训）与真实课堂之间的
AI 素养断层。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要经出版社页面（Taylor & Francis）补充核实。

### 文献 25（组 ⑤）｜幼儿教师对人工智能的态度
（Attitudes of Preschool Teachers towards Artificial Intelligence）

- **作者：** Servet Kardeş; Nur Uygun; Tuba Terim Türkmen
- **年份：** 2025
- **期刊/会议：** Southeast Asia Early Childhood Journal（第 14 卷第 1 期，第 120-135 页）
- **DOI：** [https://doi.org/10.37134/saecj.vol14.1.9.2025](https://doi.org/10.37134/saecj.vol14.1.9.2025)
- **内容摘要：** 由于人工智能的发展，AI
支持的幼儿教育开始显现更大效力。本研究采用解释性混合方法设计，旨在揭示幼儿教师对在教育中使用
AI 的态度：量化部分是对 120 名幼儿教师的问卷调查，质性部分是对 10
名教师的访谈。结果显示，幼儿教师对 AI
的态度不因年龄或工作经验而异，但因性别而异；教师总体上支持在幼儿教育中使用
AI，但使用技术设备技能较高的教师反而比技能较低的教师对 AI
态度更消极。教师指出，在设备缺乏、成本高昂、专业能力不足以及被认为不适合儿童发展的情况下，AI
无法用于幼儿教育。作者建议开展培训项目以支持幼儿教师在数字素养与 AI
使用上的自我效能感，并在幼儿学习环境中平等提供 AI 基础设施。
- **与本主题的相关性：** 东南亚语境（马来西亚期刊）的教师态度混合方法研究，其"技术技能越高者对 AI
越谨慎"的反直觉发现丰富了教师态度的解释维度。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要经出版社页面补充核实。

## 9. ⑥ 实证干预与效果（条目 26-30）

本组五篇文献代表"AI ×
学前教育"中证据强度最高的一类——实证干预与效果检验：从机器人二语辅导的大规模预注册对照实验（文献
26）、带情感个性化强化学习的机器人辅导系统（文献 27）、香港幼儿园的 AI 素养课程干预（文献
28）、智能体支持的具身学习案例（文献 29），到技术辅助词汇学习的元分析（文献 30）。与①
组偏重课程框架不同，本组直接回答"干预是否有效、对谁有效、效果多大"，是检验前几组理念落地情况的关键证据层。

### 文献 26（组 ⑥）｜用社会机器人进行第二语言辅导：一项大规模研究
（Second Language Tutoring Using Social Robots: A Large-Scale Study）

- **作者：** Paul Vogt; Rianne van den Berghe; Mirjam de Haas; Laura Hoffman; Junko Kanero; Ezgi Mamus;
等（共 23 位）
- **年份：** 2019
- **期刊/会议：** 2019 14th ACM/IEEE International Conference on Human-Robot Interaction (HRI)，第 497-505 页
- **DOI：** [https://doi.org/10.1109/hri.2019.8673077](https://doi.org/10.1109/hri.2019.8673077)
- **内容摘要：** 本文报告一项大规模研究：设计一系列七节课，用社交机器人帮助幼儿学习英语外语词汇。
实验考察三点：(1) 社交机器人在多次互动中教儿童新词（辅以平板支持）的有效性；(2)
机器人标志性手势（iconic gestures）对词汇学习与保持的附加收益；(3)
从"机器人导师加平板"学习与仅从平板应用学习的差异。为保证透明，研究问题、假设与方法均预先注册（preregistered）；样本为
194
名儿童，统计功效充足。结果显示，儿童从机器人导师处习得并保持英语词汇的程度，与仅使用平板应用时相近；此外，未发现机器人标志性手势对学习收益有额外好处。
- **与本主题的相关性：** L2
机器人辅导领域规模最大的预注册对照实验之一（欧洲多国团队、含多名本汇编其他文献作者），其"机器人不劣于平板但手势无附加收益"的结论是幼儿机器人语言学习证据链的关键一环。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录（作者共 23 位，此处列前 6 位）。

### 文献 27（组 ⑥）｜面向儿童第二语言技能的社会机器人教师的情感个性化
（Affective Personalization of a Social Robot Tutor for Children's Second Language Skills）

- **作者：** Goren Gordon; Samuel Spaulding; Jacqueline Kory Westlund; Jin Joo Lee; Luke Plummer; Marayna
Martinez; Madhurima Das; Cynthia Breazeal
- **年份：** 2016
- **期刊/会议：** Proceedings of the AAAI Conference on Artificial Intelligence（AAAI-16，第 30 卷第 1 期）
- **DOI：** [https://doi.org/10.1609/aaai.v30i1.9914](https://doi.org/10.1609/aaai.v30i1.9914)
- **内容摘要：** 尽管大量研究致力于用技术改善教育，目前仍没有方法能与一对一辅导相媲美；
有效辅导的一个关键但研究不足的方面，
是在辅导过程中调节学生的情感状态以最大化长期学习收益。作者构建了集成实验范式：
儿童在平板上玩二语学习游戏，旁边是一个全自动社交机器人学习伙伴；
系统通过自动面部表情分析测量儿童的效价（valence）与投入度，
并将这些信号合成奖励信号输入机器人的情感强化学习算法，
使其个性化调整激励策略（包括言语与非言语动作）。研究在幼儿园课堂对 34
名儿童进行了两个月的评估，发现：(1) 儿童从重复辅导中学到了新词；(2)
情感策略在研究期间随学生个性化；(3)
与个性化情感反馈机器人的儿童相比，与非个性化机器人互动的儿童效价显著更高。作者认为这种"平板教育内容
+ 情感感知 + 情感策略学习 + 自主社交机器人"的集成系统有望走向更全面的个性化辅导。
- **与本主题的相关性：** 把情感计算（affective
computing）与强化学习引入幼儿机器人辅导的早期系统论文，展示了"机器人不仅教内容、还调节情绪"的技术路线。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 28（组 ⑥）｜幼儿人工智能素养：来自香港的干预研究
（Artificial Intelligence (AI) literacy in early childhood education: an intervention study in Hong Kong）

- **作者：** Jiahong Su; Weipeng Yang
- **年份：** 2023
- **期刊/会议：** Interactive Learning Environments（第 32 卷第 9 期，第 5494-5508 页）
- **DOI：** [https://doi.org/10.1080/10494820.2023.2217864](https://doi.org/10.1080/10494820.2023.2217864)
- **内容摘要：** AI 素养议题在教育领域日益流行，但多数研究聚焦中小学与高等教育，幼儿阶段的 AI
素养课程项目罕有被考察。本研究旨在评估一个为期八周的幼儿 AI 素养课程（AI4KG）对幼儿 AI
素养、AI 相关创造力及其对课程看法的影响：研究对象为香港一所幼儿园的 26 名幼儿（平均年龄
4 岁）。结果显示：(1) 幼儿能够学习基本的 AI 概念与知识；(2) 就 AI
相关创造力而言，较小的儿童能凭想象设计聊天机器人，较大的儿童则设计出帮助人们绘画的 AI
机器人；(3) 就课程感知而言，喜欢该活动的较大儿童能够"训练 AI"，较小儿童则更喜欢画未来的
AI 城市与参与 AI 故事活动。研究凸显了 AI 素养教育在为幼儿迎接 AI
驱动的未来做准备方面的积极价值。
- **与本主题的相关性：** 华语语境（香港）幼儿 AI 素养课程的稀缺干预证据，直接检验了文献 4
的课程模型与文献 5 提出的挑战在实际课堂中的可行性。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 29（组 ⑥）｜幼儿人工智能教育：技术增强具身学习的案例研究
（Artificial intelligence education for young children: A case study of technology‐enhanced embodied learning）

- **作者：** Weipeng Yang; Xinyun Hu; Ibrahim H. Yeter; Jiahong Su; Yuqin Yang; John Chi‐Kin Lee
- **年份：** 2023（2023 年 10 月在线首发）
- **期刊/会议：** Journal of Computer Assisted Learning（第 40 卷第 2 期，第 465-477 页）
- **DOI：** [https://doi.org/10.1111/jcal.12892](https://doi.org/10.1111/jcal.12892)
- **内容摘要：** AI 素养是数字素养的关键部分，在今天技术发达的世界里每个人都应具备；尽管 AI
教育潜力巨大，关于如何教幼儿 AI
素养的研究仍然很少。本研究旨在填补这一空白，考察儿童如何参与由智能体（intelligent
agents）支持的 AI 素养活动：活动在香港一所幼儿园一个由 6 名 5
岁儿童组成的班级中开展，持续 6
周。研究从课堂观察、教师访谈与文档/作品等多种来源收集数据。结果显示，儿童能够在具身学习（embodied
learning）情境中通过与智能体互动学习
AI。研究结果对数字技术教育（尤其幼儿教育）具有广泛启示。
- **与本主题的相关性：** 把"具身学习"理论引入幼儿 AI 教育的案例研究（发表于 JCAL），与文献 4
的"具身、文化回应"主张形成理论与实证呼应。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

### 文献 30（组 ⑥）｜学龄前儿童技术辅助词汇学习：元分析
（Technology-assisted vocabulary learning for preschool children: a meta-analysis）

- **作者：** Emine Ela Şimşek; Bilal Şimşek
- **年份：** 2025（在线首发，尚未分配最终卷期）
- **期刊/会议：** Computer Assisted Language Learning（在线首发，第 1-33 页，卷期待定）
- **DOI：** [https://doi.org/10.1080/09588221.2025.2507181](https://doi.org/10.1080/09588221.2025.2507181)
- **内容摘要：** 本元分析考察 2013-2023 年间发表的、聚焦学龄前儿童技术辅助词汇教学的论文：共 24
项研究、涉及 1759 名儿童，产生 29
个效应量。技术类应用在词汇教学中的总体效应量估计表明，这些应用对儿童词汇学习有中等效应（g
=
0.729），即技术支持的应用能有效向学龄前儿童教授词汇。调节变量分析考察了影响词汇学习效果的因素，包括语言类型、目标语言、媒体平台、干预时长与干预情境；结果显示语言类型与干预时长是影响该年龄段词汇学习的显著因素，而目标语言、媒体或干预情境等特征不影响技术应用的效应量。总体而言，技术辅助词汇教学可带来积极结果，规划教学时应考虑这些发现。
- **与本主题的相关性：** 以元分析方法为"技术（含 AI
类应用）对幼儿词汇学习是否有效"给出总体效应量，为学前语言技术干预提供了证据等级的顶端参照。
- **核实状态：** ✅ 元数据经 Crossref/OpenAlex 核实；摘要取自 API 收录。

## 10. 主题分析：跨文献的核心发现

**发现一：幼儿 AI 研究正从"概念倡导"转向"课堂实证"，但证据仍集中于少数团队与地区。**
早期文献多以理论框架与综述为主（文献 3、7、8），2019 年后出现以真实幼儿为对象的干预证据：PopBots
平台让幼儿训练机器人学习 AI 概念（文献 6）、香港的 AI4KG 八周课程（文献
28）、智能体支持的具身学习案例（文献 29），以及欧洲多国团队的 194 名儿童大规模机器人二语实验（文献
26）。但需注意，干预研究高度集中于 Su、Yang、Ng 等华人学者网络（文献 2、4、5、15、28、29）与 MIT/Breazeal
团队（文献 6、17、27），样本多为香港、美国、芬兰、土耳其等特定地区，证据生态的多样性有限。

**发现二：机器人教学的"物理在场"收益明确，但"社会行为"是把双刃剑。**
综述一致认为社交机器人的物理在场是超越传统屏幕技术的关键机制（文献 7、8）；然而 Kennedy
等发现，在辅导策略之外叠加社交与适应性行为反而使儿童没有显著学习（文献 10），Vogt
等的大规模实验也显示机器人标志性手势并不带来额外词汇收益（文献 26）。同时，Kanero
等提醒证据尚不足以断言机器人可与人类教师媲美（文献
9）。这些发现共同指向一个设计原则：教育机器人的"社交性"必须服务于教学目标，而非为社交而社交。

**发现三：幼儿 AI 素养研究形成了"概念—课程—实施"的完整链条，但评估工具仍薄弱。** 从 AI
素养四方面框架的理论奠基（文献 3）、范围综述（文献 2、5）到"为什么/是什么/怎么做"的课程模型（文献
4），幼儿 AI 素养已具备相对连贯的理论-课程链条，并有 AI4KG（文献 28）与 PopBots（文献
6）等实施样例。但多数研究的评估采用研究者自编工具、样本量小（如 6-26
名儿童），缺乏标准化、跨文化可比的幼儿 AI 素养测量，这一短板限制了不同干预之间的比较与结论累积。

**发现四：生成式 AI 时代，"教师端应用先于儿童端"是当前格局。** 在③ 组仅有的 3 篇文献中，ChatGPT
主要作为教师的备课工具（文献 14）或引发伦理思辨的对象（文献 13），通用框架（文献
15）虽讨论教育应用但并非学前专属；本汇编中尚无一篇关于幼儿直接与生成式 AI
对话/共创的同行评议干预研究。这与 2023 年后教师态度研究爆发（文献 21-25，多发表于 2025-2026
年）形成鲜明对照：成人端对大模型的采纳讨论热火朝天，儿童端的大模型实证几乎空白——这是本主题当前最大的结构性缺口（详见第
11 节）。

**发现五：伦理与隐私研究沿着"数据化童年"脉络展开，从'会不会出事'走向'儿童权利如何保障'。**
早期工作以揭示风险为主：联网玩具会录音而儿童毫不知情（文献 16），智能玩偶能改变幼儿的道德判断（文献
17）；随后研究把问题上升到概念与权利层面——儿童对个人数据的理解（文献
18）、儿童被数据化与监视式数据化的整体批判（文献 19）、以及机器人教师的伦理边界（文献
20）。这一脉络提示：幼儿 AI
应用的伦理治理不能只靠家长控制功能，而需要把儿童数据权利、适龄设计与教师伦理培训纳入同一框架。

## 11. 研究缺口

基于第 10 节的分析，本汇编识别出以下研究缺口，供后续选题参考：

1. **学前阶段生成式 AI/大语言模型的直接实证干预稀缺。** ③ 组仅 3 篇且以观点与教师端为主；截至 2026-09-05
核验日，面向幼儿直接使用 ChatGPT
等大模型的课堂干预、对话设计与效果评估几乎空白，亟待填补。
2. **长时程与纵向证据不足。** 现有干预多为数周至数月（如文献 28 的 8 周、文献 29 的 6 周、文献 27 的 2
个月），鲜有跟踪"新奇效应"消退后的长期效果、以及 AI
经验对儿童发展轨迹影响的研究。
3. **样本集中于发达国家/地区与城市。** 本汇编样本主要来自美国、芬兰、香港、土耳其等地（文献
1、6、24、26、28、29），中低收入国家、农村与多元文化情境的幼儿 AI
研究几乎缺席，跨文化比较研究稀少。
4. **儿童数据权利与适龄设计缺乏操作性框架。** 伦理讨论（文献
16-20）多停留在风险揭示与概念批判，把儿童数据权利落到产品设计规范、课堂数据治理流程与可执行政策的具体框架仍然缺失。
5. **教师职前培训（师范教育）缺口明显。** 文献 5、22、24、25 反复报告教师 AI
知识、技能与信心不足，其中文献 24
显示职前教师"理论有余、实践不足"；如何把 AI
素养系统纳入幼儿师范课程的研究才刚起步。
6. **元分析与系统综述集中于词汇、语言等窄结果。** 文献 8、9、30 的严格证据综合均围绕语言学习展开；对幼儿
AI
素养、计算思维、创造力、情绪与社会性发展等结果的量化综合仍然缺乏。
7. **家长视角与家园共育研究相对薄弱。** 本汇编 30 篇中以教师为对象的有 5 篇（组⑤），但直接考察家长对 AI
教育产品态度与家庭 AI 媒介养育的研究较少（文献 16
是少数例外），与 AI 产品大量进入家庭的现实不匹配。
8. **智能玩具/机器人"玩具形态 AI"的证据质量偏低。** 组② 中玩具类条目（文献
11、12）以设计与案例为主，缺少类似文献 26、27
规模的对照实验；智能玩具宣称的学习效果尚缺严格检验。

## 12. 核查记录

本节的核实状态图例与文件头部说明一致：✅ = 元数据（DOI、英文标题、作者、年份、期刊/会议、卷/期/页）已通过
API 核实，摘要取自 API/出版方收录；⚠ = 未能核实/存疑；⚠-knowledge =
元数据已核实，但摘要缺失、内容摘要为编译者依据标题与领域知识撰写；❌ = 无法核实或来源冲突（本汇编 30
条中无 ❌）。下表逐条记录核验来源与说明：

| 条目 | 文献（英文短题） | 核实状态 | 核验来源/说明 |
| --- | --- | --- | --- |
| 1 | Learning machine learning with very young children | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 2 | Artificial intelligence in early childhood education: A scoping review | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 3 | Conceptualizing AI literacy: An exploratory review | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 4 | AI education for young children: Why, what, and how | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 5 | AI (AI) Literacy in ECE: The Challenges and Opportunities | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 6 | A is for Artificial Intelligence (CHI 2019) | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 7 | Social robots for education: A review | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 8 | Social Robots for Language Learning: A Review | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 9 | Social Robots for Early Language Learning | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 10 | The Robot Who Tried Too Hard (HRI 2015) | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 11 | Maya: An AI based smart toy for pre-school children | ⚠（摘要缺失） | 元数据 ✅（Crossref/OpenAlex）；三个 API 均无摘要，内容摘要为编译者撰写（⚠-knowledge） |
| 12 | AI-interfaced robotic toys in early childhood settings | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 13 | ChatGPT, care and the ethical dilemmas in the early years | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录（评述/观点类文体，已如实标注） |
| 14 | Exploring the efficacy of lesson planning in ECE using ChatGPT | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 15 | Unlocking the Power of ChatGPT: A Framework for Generative AI | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 16 | Toys that Listen: Parents, Children, and Connected Toys | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 17 | "My doll says it's ok": children's conformity to a doll | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 18 | Smart toys and children's understanding of personal data | ⚠（摘要缺失） | 元数据 ✅（Crossref/OpenAlex）；三个 API 均无摘要，内容摘要为编译者撰写（⚠-knowledge） |
| 19 | The datafied child: dataveillance and children's rights | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 20 | Should we welcome robot teachers? | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 21 | Preschool teachers' behavioral intention to use AI | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 22 | Perceptions of EC teachers on the use of AI in ECE | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 23 | Incorporating Humanoid AI Robots into ECE | ✅ | Crossref/OpenAlex（元数据）；摘要经出版社页面（Springer）核实 |
| 24 | AI in early childhood classrooms: in-service and pre-service | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录（出版方收录摘要） |
| 25 | Attitudes of Preschool Teachers towards Artificial Intelligence | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录（出版方收录摘要） |
| 26 | Second Language Tutoring Using Social Robots (HRI 2019) | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录（23 位作者，已按规则缩写） |
| 27 | Affective Personalization of a Social Robot Tutor (AAAI-16) | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 28 | AI literacy in ECE: an intervention study in Hong Kong | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 29 | AI education for young children: embodied learning case study | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |
| 30 | Technology-assisted vocabulary learning: a meta-analysis | ✅ | Crossref/OpenAlex（元数据）；摘要：API 收录 |

汇总：30/30 条元数据（DOI、英文标题、作者、年份、期刊/会议、卷/期/页）均于 2026-09-05 经
Crossref/OpenAlex API 核实通过，无 ❌ 条目；28 条摘要取自 API/出版方收录，2 条（文献 11、18）三个 API
均未收录公开摘要，其"内容摘要"已按 ⚠-knowledge 如实标注为编译者撰写。正文 30 条与核验源文件的 ACTIVE
清单一一对应，未使用后备条目。

## 13. 未能核实的内容

本着诚实原则，本指南明确列出以下未能核实或核实受限的内容：

1. **文献 11（Maya）与文献 18（智能玩具与儿童对个人数据的理解）无 API
收录摘要**：Crossref、OpenAlex、Semantic Scholar 三个 API
均未收录这两篇的摘要，其条目中的"内容摘要"系编译者依据论文标题与领域背景撰写的谨慎转述（已标注
⚠-knowledge），未逐字核对原文；编译过程中未为这两条虚构任何具体数字、样本量、方法或结论，请以出版社原文为准。
2. **在线首发条目的最终卷期未能核实**：文献 14、22、30（以及文献 24）截至核验日仅有"在线首发 +
页码"，尚未分配最终卷/期，本指南如实标注"卷期待定"；正式卷页信息以出版社最终出版为准。
3. **跨年条目的年份标注未能完全统一**：文献 8、23、24、29 的"在线首发"与"印刷卷期"跨年，本指南以 Crossref
记录为准并同时给出在线首发时间，个别数据库（如 Semantic Scholar）的年份显示可能与 Crossref 略有出入。
4. **未逐一阅读 30 篇全文**：各条"内容摘要"均依据原作者英文摘要压缩改写，可能存在转述损失或对原文强调点的取舍；深度引用前建议核对原文。
5. **文献 13 的"摘要"性质**：该文为评述/观点类，期刊页面收录的"摘要"实为正文开头文字，本指南已在条目与第
12 节中如实标注，未将其混同为常规研究摘要。
6. **检索窗口之外的内容未能覆盖**：本汇编未收录非英文文献、未纳入灰色文献与商业报告，亦未包含 2026 年 9
月检索核验日之后新发表或新上线的研究。
7. **元数据字段的细微差异未能逐一比对**：个别作者的姓名拼写（含土耳其语、汉语拼音字符）以 Crossref
为准，其他数据库可能存在大小写或字符变体，本指南未对每条做多库逐字符比对。

## 14. 结语

把 30 篇文献放在一起看，AI 与学前教育的交叉研究在过去十年完成了从"零散尝试"到"初具结构"的转变：幼儿 AI
素养有了从概念框架到课程干预的完整链条（组①
与组⑥），教育机器人积累了从综述到大规模预注册实验的证据（组②），伦理与隐私研究形成了"数据化童年"的批判脉络（组④），而教师视角研究在生成式
AI 兴起后迅速升温（组⑤）。这些进展说明，"幼儿与 AI
的关系"已经成为一个可以严肃研究的学术议题，而不再只是技术乐观主义或恐慌的谈资。

与此同时，本汇编也如实暴露了该领域的年轻与不平衡：生成式 AI
的幼儿实证几乎空白、证据集中于少数地区与团队、儿童数据权利缺乏操作性框架、教师培训明显滞后于技术迭代。对一个直接影响
0-6 岁儿童的领域而言，这些缺口既是风险，也是研究者的机会。

对读者而言，本指南的价值在于提供了一张经过元数据核验的"地图"：研究者可循组① 与组⑥
定位课程与干预文献、循组② 与组④ 理解机器人应用的两面性；教师与园长可重点阅读组⑤
的态度与框架文献，从中找到落地与培训的参照；政策与产品从业者则可从组④
的伦理脉络出发，思考儿童数据权利与适龄设计的具体化。

最后需要再次强调本指南的边界：全部元数据已于 2026-09-05 经 Crossref/OpenAlex 等 API 核实，但 30 篇中仍有 2 篇（文献
11、18）缺少公开摘要，其注释为编译者的谨慎转述；所有"内容摘要"均为原作者摘要的中文压缩，未替代原文。引用任何一篇文献前，请以出版社版本为准。

AI 与学前教育的研究仍在快速演进，本指南只是一个时间切面。期待这份注释书目能成为中文读者进入该领域的一块稳固的垫脚石——也欢迎读者依据第
2 节的检索方法自行更新与扩展这张地图。

