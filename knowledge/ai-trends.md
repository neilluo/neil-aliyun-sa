# AI 趋势 — ai-trends.md

> **定位**：阿里云在 AI 维度的产品演进、行业落地、客户实践。
> **更新方式**：百炼/PAI/通义新动态、ATA AI 实践文章、行业 AI 案例蒸馏后回灌。
> **分级**：区分"已规模化生产" / "早期落地" / "概念阶段"。

**↔️ Cross-references**：
- AI 产品能力 → [aliyun-products.md](aliyun-products.md)(百炼/PAI/通义各产品卡)
- AI 方案 → [cloud-solutions.md](cloud-solutions.md)(S3 AI Agent 方案等)
- AI 竞品 → [competitor-cloud.md](competitor-cloud.md)(模型竞品矩阵/火山引擎对比)
- 客户 AI 实践 → [company-profiles.md](company-profiles.md)(客户 AI 落地案例)

## 关键追踪线

### 1. 百炼（Model Studio）[官方]
- 模型 / 路由 / 工具 / 工作流 / Agent / RAG 能力演进
- 与 PAI / 通义 的边界：**百炼 = MaaS 应用层**（API 直调、Agent 编排、RAG 知识库）；**PAI = 训练与推理基础设施**（DLC 训练、EAS 推理部署、GPU 资源管理）
- 客户接入模式（应用直调 / Agent 编排 / 私有部署）
- 模型矩阵（2026-06更新）：**Qwen3.7-Max**（最新旗舰，视觉+文本）/ Qwen3.7-Plus（多模态智能体）/ Qwen-Plus（日常平衡1M）/ Qwen-Turbo（轻量1M）/ Qwen-VL（视觉）/ Qwen-Audio / Qwen-Coder
- 旗舰模型演进：Qwen-Max → Qwen3-Max-Thinking(1月,1T+参数) → Qwen3.5系列(3-4月) → **Qwen3.7-Max**(5月20日云峰会,当前最强)
- 竞品定价对标：Qwen3.7-Max 12元/百万input vs GPT-4o ~$5/M → 价格相近；Qwen-Plus 0.8元/百万 vs GPT-4o-mini → 极致性价比

**百炼 API 限流表 [官方]**：

| 模型 | RPM(请求/分钟) | TPM(Token/分钟) | 等效 QPS |
| --- | --- | --- | --- |
| qwen-max | 1,200 | 1,000,000 | ~20 |
| qwen-plus | 30,000 | 5,000,000 | ~500 |
| qwen-turbo | 1,200 | 5,000,000 | ~20 |
| deepseek-v4-pro | 15,000 | 1,200,000 | ~250 |

- 限额按**主账号**维度（所有 RAM 子账号/工作空间/APIKey 共享）
- 可临时提升配额（控制台申请，北京/新加坡即时生效，最长30天）
- PTU（专属吞吐单元）：大客户预留容量，绕过共享限流
- Batch API：非实时异步批处理，50%折扣，更高吞吐允许
- **SLA**：可用性 ≥ 99.9%（赔偿阶梯：99.0-99.9%→10%代金券；95-99%→25%；<95%→50%）
- **无延迟 SLA**：无官方 TTFT/P99 延迟承诺

**百炼 vs PAI-EAS 成本交叉点 [推断]**：
- 月调用量低（<300万token/天）→ 百炼 MaaS（Token 计费更划算）
- 月调用量高（>3000万token/天）或 QPS 长期 >100 → PAI-EAS 私有部署（GPU 时长摊薄）
- 数据不出域 → 只能 PAI-EAS 或自建

### 2. PAI 全栈 [官方]
- DSW（开发） / DLC（训练） / EAS（推理） / Designer（低代码）
- GPU 资源池化、PD 分离（Prefill-Decode 分离部署，训练推理混合调度）、推理加速（vLLM / TensorRT / ACC）
- 大规模训练实践：千卡级 DLC 训练集群 + CPFS 共享数据 + OSS Checkpoint
- EAS 核心能力：自动扩缩 / A/B Test / 蓝绿发布 / 模型版本管理 / 流量镜像
- 选型边界：需要全流程 ML 工程能力时用 PAI；只需 API 调模型时用百炼

**PAI-EAS GPU 定价表 [官方]**：

| GPU 型号 | 实例规格 | 显存 | vCPU | 内存 | 按量(CNY/时) | 月估(CNY) |
| --- | --- | --- | --- | --- | --- | --- |
| NVIDIA T4 | gn6i | 16GB | 16 | 62GB | ~14.82 | ~10,670 |
| NVIDIA A10 (GU30) | gn7i / ml.gu7i | 24GB | 16 | 60GB | ~12.71 | ~9,151 |
| NVIDIA V100 | gn6v | 32GB | 8 | 32GB | ~26.46 | ~19,051 |
| NVIDIA A100 | gn7e / ml.gu8xf | 80GB | 16 | 125GB | ~34.74 | ~25,013 |
| NVIDIA H800/H100 | 灵骏智算 | 80GB | - | - | 未公开（需销售，仅包年） | - |

- 资源抵扣包：59 CNY = 200 CNY 额度（约 3 折）
- A10 是最优性价比选择（24GB 够跑 7B/14B 模型，价格最低）
- H800/H100 仅灵骏智算程序提供，包年包月，需销售介入

**GPU 推理 Capacity Planning 参考 [实战+推断]**：

| 模型 | GPU 配置 | 输出吞吐(tokens/s) | 并发支撑 | 场景 |
| --- | --- | --- | --- | --- |
| Qwen2.5-7B | 1×A10 (24GB) | ~30-45 (短上下文) | ~50-80 并发 | 轻量客服/摘要 |
| Qwen2.5-7B | 1×A100 (80GB) | ~40 (短) / ~19 (长) | ~100 并发 | 标准推理 |
| Qwen2.5-14B | 2×A10 (48GB) | ~20-30 | ~30-50 并发 | 复杂对话/Agent |
| Qwen2.5-72B | 4×A100 (320GB) | ~155 总 / ~39/卡 | ~20-40 并发 | 企业级推理 |
| DeepSeek-R1-671B | 8×H800 (FP8) | ~620 总 | ~100 并发 | 超大模型 |

> **Capacity Planning 公式（估算）**：所需 GPU 卡数 ≈ (目标并发 × 平均output长度) / (单卡tokens/s × 可接受延迟秒数)
> 例：2000并发 × 200 token/响应 / (30 tokens/s/卡 × 5s 可接受延迟) = ~2667/150 ≈ **27 张 A10**（跑14B模型）

### 3. 通义系列 [官方+推断]
- **千问 Qwen**：语言模型，开源生态（Qwen2.5 系列 72B/14B/7B/1.5B 全开源）；长文本（128K+）；多模态（VL/Audio）
- **万相**：图像/视频生成（文生图/图生图/视频生成），对标 Midjourney/DALL-E
- **Qoder CN 系列**（原灵码 Lingma，2026.5.20 更名）：从 AI 代码助手升级为 Agentic Coding Platform。产品线含 Qoder CN（IDE 插件/Quest 自主开发）、QoderWork CN（桌面办公助手）、CLI、Cloud Agents。对标 GitHub Copilot/Cursor/Windsurf。全球版 qoder.com / 中国版 qoder.com.cn
- **听悟**：语音识别/会议纪要/音频理解
- 与百炼的关系：通义模型 = 底层能力；百炼 = 调用/编排/管理平台；DashScope = API 网关通道

### 4. AI Infra（推理服务架构）[实战-ATA]

**PD 分离（Prefill-Decode 分离部署）**— 来源：ATA SA动手系列2 (11020600554)
- 核心思想：Prefill = 计算密集(TP8)，Decode = 显存密集(DP8+EP16)，资源配比完全不同
- 实测数据(DeepSeek-R1-671B FP8)：
  - Prefill GPU 利用率 95%+；Decode 受限于 KV Cache 显存
  - 1K 上下文：204 tokens/s 解码，441 并发
  - 64K 上下文：34 tokens/s 单卡，TTFT 2569ms
  - MTP 推测解码：TPOT 58ms→52ms (-10.3%)，接受率 ~68%
- KV Cache 传输：GPUDirect RDMA（200Gbps，微秒延迟，零 CPU 开销）
- Chunked Prefill：64K 序列不 Chunk 会 OOM，Chunk 后延迟仅 +7.8%
- P/D 比例公式：Ratio = (InputLen × PrefillTime/Token) / (OutputLen × DecodeTime/Token)
  - 1P3D（平衡）、2P1D（RAG/长输入）、1P8D（代码生成/长输出）
- 监控关键指标：prefill_queue_length / decode_queue_length / ttft_p99 / tpot_p99 / GPU 利用率
- 客户实践：中国移动智算推理池 PD 分离测试
- 产品组合：PAI + ACS/ACK GPU 集群 + IB 网络(RDMA) + SGLang 推理引擎

**高可用 AI 系统**— 来源：ATA "构建高可用AI系统" (11020604053)
- 核心模式：AI Gateway(Higress) 多模型容灾 → 0.5 秒自动切换
- Prompt 热更新：MSE 配置中心管理 Prompt 版本 + 灰度发布，无需重启
- Agent 沙箱：AgentRun 隔离执行，动态加载+权限控制+行为审计
- 弹性实例：ACK + ECI，10 → 200 实例自动扩缩
- 向量库优化：跨地域向量库部署，延迟降低 70%
- 设计原则：HA 架构必须匹配业务重要性，避免过度设计
- 核心公式：生产级 = 高可用模型 × 高可用应用 × 全景可观测

**通用推理架构选型**：
- ACS / ECI / GPU ECS 选型 → 参见 cloud-solutions.md 升级阈值表
- PD 分离适用场景：模型 >100B 参数 + 长上下文 + 高并发 + 多卡集群

### 5. AI Agent 与智能体 [官方+实战]

**百炼 Agent 2.0 架构**— 来源：help.aliyun.com + ATA 多篇
- Zero-code 可视化编排 + Full-code ADK 开发套件 + ADP 生产部署
- MCP 协议（Model Context Protocol）：标准化工具互操作
- Multi-Agent 编排：专家 Agent 协作处理子任务
- Workflow：DAG 编排，条件分支/循环/人工审批/异步等待

**Agent 落地三层次**— 来源：ATA "Java程序员转型AI" (12020603721)
- L1 Prompt User（高可替代性）→ L2 AI 应用集成者（中等）→ L3 AI-Native 架构师（极低可替代性）
- Java 开发者转型优势：OOP→nn.Module 映射、分布式经验、CI/CD/监控工程力
- Spring AI 框架：Java 原生 AI 集成路径（ChatClient Builder / 流式响应）
- RAG 四阶项目训练：API 调用(1-2周) → RAG系统(3-6周) → Multi-Agent(7-10周) → 企业AI应用(11-12周)

**企业级 Agent 案例**：
- 一汽红旗"云妹"：BI 报告 7天→5秒，人员减半 [官方]
- 哈啰出行：运维成本-30%，GMV+5%，纠纷准确率87% [官方]
- 51Talk：响应-30%，成本-20~30%，事件驱动+主动外呼 [官方]
- 网商银行：决策树+FaaS+Agent 诊断系统，故障恢复3分钟内，准确率93.5% [实战-ATA]

### 6. RAG 与向量检索 [官方+实战]

**百炼知识库规格**：
- 标准版：100GB 存储，1 QPS 固定（开发测试）
- 旗舰版：9999GB，1-200 RCU（1 RCU ≈ 50 QPS），0.2 CNY/RCU/hr
- 单次检索最多 20 分片，每分片 ≤ 6000 token

**向量库选型矩阵**：
- **百炼内置知识库**：零代码 RAG，全托管，最快上手
- **Hologres 向量**：OLAP + 向量统一，SQL 友好，AI Function 自动生成 embedding
- **AnalyticDB 向量**：AI + 数据分析融合，内置向量搜索引擎
- **Lindorm 向量**：十亿级向量，宽表+向量一站式，最低成本高写入
- **OpenSearch 向量版**：托管搜索+向量，电商/内容搜索型 RAG

**RAG 工程化**：
- 文档解析 / 切片策略（chunk_size ≤ 6000 token + overlap + 语义切片）
- 召回策略：混合检索 BM25+向量 / Rerank 重排
- 知识库工程化：文档更新频率 / 索引刷新 / 评测指标（Recall@K, MRR）
- 混合检索（关键词+向量+Re-ranking）为生产推荐模式 [实战-ATA]

### 7. AIGC 内容生产 [推断]
- 文生图 / 文生视频 / 数字人
- 创意生产工业化（HappyHorse 等内部能力）
- 行业落地（广告、娱乐、教育、零售）

### 8. 模型即服务 (MaaS) [官方+推断]
- 收入 / 调用量 / GPU 台数趋势（参考 fbi-maas-daily-data-analysis 数据）
- 模型 API 定价模式
- 客户画像

---

## 待蒸馏种子

- [x] ATA：高可用 AI 系统 (11020604053) — 已蒸馏到上方第4节
- [x] ATA：SA 动手系列 PD 分离部署 (11020600554) — 已蒸馏到上方第4节
- [x] ATA：Java 程序员转型 AI (12020603721) — 已蒸馏到上方第5节
- [x] ATA：网商银行智能化定位 (12020553615) — 已蒸馏到上方第5节
- [ ] ATA：Aegis AI 数字分身 — 未找到原文，待确认标题
- [ ] ATA：AIGC 证券投教视频 — 视频 AIGC
- [ ] help.aliyun.com 百炼最新文档 — 部分已融入第5/6节
- [ ] help.aliyun.com PAI EAS 推理服务最佳实践
