# 6个Agent的任务分配 等细则（自 SKILL.md 下沉）

> 主文件只留指针；改动需与 SKILL.md 同步。

#### 6个Agent的任务分配

| Agent | 搜索目标 | 提取重点 | 输出文件 |
|-------|---------|---------|---------|
| 1 著作 | 书、长文、论文、newsletter | 反复出现的核心论点（≥3次=真信念）、自创术语、推荐书单 | `01-writings.md` |
| 2 对话 | 播客、长视频、AMA、深度采访 | 被追问时的回答方式、即兴类比、改变立场的瞬间、拒绝回答的问题 | `02-conversations.md` |
| 3 表达 | Twitter/X、微博、即刻、短文 | 高频用词句式、争议立场、幽默方式、公开辩论 | `03-expression-dna.md` |
| 4 他者 | 他人分析、书评、批评、传记 | 外部观察到的模式、批评与争议、与同行对比 | `04-external-views.md` |
| 5 决策 | 重大决策、转折点、争议行为 | 决策背景与逻辑、事后反思、言行一致/不一致案例 | `05-decisions.md` |
| 6 时间线 | 出生/出道到现在的完整时间线 | 关键里程碑、思想转折点、**最近12个月动态**（防过时） | `06-timeline.md` |

#### 每个Agent的硬性要求
- 调研结果必须写入 `references/research/0X-xxx.md`
- 注明信息来源和可信度（一手>二手>推测）
- 区分「他说过的」vs「别人说他的」vs「我推断的」
- 发现矛盾时保留矛盾，不要和稀泥

#### Agent prompt模板

spawn subagent时，用以下结构给任务（以Agent 1著作为例）：

```
你的任务：调研[人名]的著作和系统性长文。

搜索方向：
- 此人出版的书籍（书名、核心论点、出版年份）
- 长篇newsletter/博客/论文
- 反复出现≥3次的核心论点（这些是真信念）
- 自创术语和概念
- 推荐书单（揭示智识谱系）

输出要求：
- 写入 [skill目录]/references/research/01-writings.md
- 每条信息标注来源URL和可信度
- 区分一手（此人写的）vs 二手（别人总结的）
- 发现矛盾直接记录，不要调和

信息源黑名单：不使用知乎、微信公众号、百度百科。
```

其他5个Agent按同样结构调整搜索方向和输出文件名即可。

#### 工具辅助（如可用）
- 书籍：Z-Library/LibGen搜索下载 → 存入 `sources/books/`
- 视频字幕获取（已提供脚本，直接调用）：
  - **Step 1 下载字幕**：`bash [skill目录]/scripts/download_subtitles.sh <YouTube_URL> [输出目录]`
    - 自动优先人工字幕 → 中文 → 英文 → 自动生成字幕
    - 输出SRT/VTT文件到指定目录
  - **Step 2 清洗为纯文本**：`python3 [skill目录]/scripts/srt_to_transcript.py <input.srt> [output.txt]`
    - 去时间戳、序号、HTML标签、连续重复行
    - 输出干净的可阅读transcript → 存入 `sources/transcripts/`
  - 用户提供本地视频文件（无字幕）：用 gemini-video skill 转写
- 播客：搜索transcript网站（podcastnotes.org等）
- 调研摘要生成（Phase 1.5用）：`python3 [skill目录]/scripts/merge_research.py <skill目录>`
  - 自动扫描 `references/research/01-06.md`，统计来源数、一手/二手占比、关键发现
  - 输出Phase 1.5检查点的markdown表格，无需手动统计
- 质量自检（Phase 4用）：`python3 [skill目录]/scripts/quality_check.py <SKILL.md路径>`
  - 自动检查6项通过标准：心智模型数量、局限性、表达DNA、诚实边界、内在张力、一手来源占比
  - 输出逐项PASS/FAIL和总结

#### 利用已安装的信息获取Skill

Phase 1启动前，**主动扫描当前 runtime 的 skills 目录**，检查是否有可用于信息获取的skill。如果有，在调研中优先调用，比WebSearch更稳定高效：

| 已安装Skill | 用途 | 调用场景 |
|------------|------|---------|
| `gemini-video` | 分析本地视频文件，提取transcript | 用户提供了视频文件但没有字幕 |
| `web-article-reader` | 精确读取网页文章全文 | 找到重要文章URL时，精确提取而非依赖搜索摘要 |
| `agent-reach` | 多渠道信息获取（17个平台） | 需要从X/Reddit/YouTube等平台获取信息 |
| `huashu-research` | 结构化深度调研 | 需要对某个维度做深度调研而非广撒网 |
| `pdf` | 读取PDF书籍/论文 | 用户提供了PDF格式的一手素材 |

**执行方式**：在spawn subagent时，把可用skill的名称和用途告知agent，让agent在调研中按需调用。这比让agent自己用WebSearch摸索效率高得多。

**信息源优先级/黑名单/中文渠道** → 按需读 [references/source-policy.md](references/source-policy.md)（知乎/公众号/百度百科永远排除）。


