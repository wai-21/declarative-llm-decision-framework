---
name: "skillweaver-routing"
description: "工具过多时的精准路由：分解→SAD对齐→向量检索→DAG组合，替代暴力塞入，Token降99%、准确率大涨。"
---

# SkillWeaver 精准技能路由（通用版）

> 当 Agent 可用工具/技能过多时，"该用哪个工具"比"能不能用"更重要。
> 本技能提炼自阿里研究院 SkillWeaver 框架（arXiv:2606.18051）：
> 用「分解 → 检索 → 组合」三阶段管线替代"把所有工具描述暴力塞入上下文"，
> 实测 Token 消耗降 99.9%（884,000 → ~1,160），工具选择准确率从 21.1% 大幅提升。

## 何时使用本技能

- 可用工具/技能数量多（几十到上千），无法/不宜全部塞入上下文
- 用户请求需要**组合多个工具**才能完成（如"下载数据→转换格式→生成报告"）
- 工具描述总 Token 占用过高、命中率低、出现上下文溢出/截断报错
- 任何需要"从工具库中选对工具"的决策点

**不要用**：工具数量少（<20）且一次性任务时，直接调用即可，不必走完整管线。

## 核心理念

把"让 LLM 读完整本百科全书再回答"改成"先用向量检索翻到正确页码，再让 LLM 只读那一页"。

```
用户查询 → Decompose(子任务序列) → Retrieve(候选技能集) → Compose(DAG 执行图)
                    ↑____________ SAD 反馈环路贯穿 ____________|
```

## 三阶段管线 + SAD

### 阶段 0（一次性，离线）：建技能索引

1. 为每个可用技能/工具准备 `{name, description}`
   - **描述必须用工具自己的技术词汇**（如 `api-client`、`http-fetch`），不要意译
2. Embedding 模型编码描述：
   - 英文：`all-MiniLM-L6-v2`（轻量）/ `BGE-base-en-v1.5`（更准）
   - 中文：`BAAI/bge-base-zh-v1.5`
3. 建 FAISS 索引（`IndexFlatIP` 即可；2209 个技能约 15 秒建库，单次查询 <15ms）

### 阶段 1：Decompose 任务分解

- 将用户查询拆为**有序原子子任务**，每个子任务应对应一个可执行工具
- ⚠️ **防过度分解**：模型越大越倾向拆得过细（大厨把红烧肉拆成 8 步，货架上只有"猪肉-五花"），微观步骤在技能库中找不到匹配 → 检索全灭。**分解粒度是检索准确率的最大瓶颈**

### 阶段 2：SAD 反馈环（核心创新，必须执行）

对齐 LLM 词汇与工具库词汇：

```
① 子任务(LLM 的话) → ② 语义检索 Top-K → ③ 回注工具名+描述 → ④ LLM 用工具词汇重写
→ 有变化则回到①再检索，直至无变化或达到 max_iter(3)
```

- 效果：7B 模型分解准确率 51.0% → 67.7%（+33%）；Qwen-Max 达 92%
- 困难任务（4-5 技能协作）准确率提升 50%
- 无 SAD 时 14B 模型反而 < 7B（过度分解陷阱）——对齐 > 算力

### 阶段 3：Retrieve 候选召回（LLM 不参与，0 Token）

- Bi-Encoder 向量检索召回 Top-K（k=5~10）
- ⚠️ Bi-Encoder 召回 Top-10 ≈ 70%，但 Top-1 仅 ≈ 37% → **生产环境必须加重排**：
  Cross-Encoder（如 `cross-encoder/ms-marco-MiniLM-L-6-v2`）或 LLM Reranker 对候选精排

### 阶段 4：Compose 组合 DAG

- 评估候选工具间兼容性（前序输出 = 后续输入），为每个子任务选定最终工具
- 输出 DAG：节点=工具，边=依赖；无依赖的步骤并行、有依赖的按序执行
- 不要用 ReAct 式"走一步看一步"（多工具编排场景实测 0% 准确率），必须先规划完整路线再执行

## 生产必加：错误恢复层（论文未覆盖）

- 每节点：超时阈值 + 重试策略
- 首选工具失败 → 自动降级备选工具
- 断点续传：已成功节点不重跑
- 输出校验：每步输出格式校验，防畸形数据向下游传播

## 关键认知（决策参考）

1. **对齐 > 算力**：7B 小模型 + 好的路由/对齐策略 > 裸奔的 14B+ 大模型。与其升级模型，不如先做工具词汇对齐
2. **规划式 > 反应式**：复杂工具编排必须"先想清楚再做"（DAG 预规划），ReAct 只适合单步简单场景
3. **分解粒度是瓶颈**：检索算法再强，分解错了也白搭；用 SAD 把粒度拉回现实

## 参考代码骨架（Python）

### 索引构建

```python
from sentence_transformers import SentenceTransformer
import faiss, numpy as np, json

skills = [...]  # [{"name": "api-client", "desc": "通过HTTP API获取远程数据"}, ...]
encoder = SentenceTransformer("all-MiniLM-L6-v2")  # 中文用 bge-base-zh-v1.5
embeddings = encoder.encode([s["desc"] for s in skills])
index = faiss.IndexFlatIP(embeddings.shape[1])
index.add(embeddings.astype(np.float32))
```

### 分解 + SAD + 组合

```python
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="...")  # 任意 LLM

def decompose(query):
    prompt = f"将用户查询拆解为有序原子子任务(JSON): {query}"
    return json.loads(llm.invoke(prompt).content)["subtasks"]

def retrieve_topk(subtask, k=5):
    qv = encoder.encode([subtask])
    scores, idxs = index.search(qv, k)
    return [skills[i] for i in idxs[0]]

def sad_loop(subtasks, max_iter=3):
    for _ in range(max_iter):
        changed = False
        for i, t in enumerate(subtasks):
            cands = retrieve_topk(t)
            ctx = "、".join(f"{c['name']}: {c['desc']}" for c in cands)
            rewritten = llm.invoke(
                f"参考可用工具[{ctx}]，用工具的真实词汇重写子任务'{t}'"
            ).content.strip()
            if rewritten != t:
                subtasks[i] = rewritten
                changed = True
        if not changed:
            break
    return subtasks

def compose_dag(subtasks):
    plan = [retrieve_topk(t, k=1)[0]["name"] for t in subtasks]
    # LLM 评估兼容性 → 输出 nodes/edges
    return {"nodes": [{"tool": t} for t in plan], "edges": [...]}
```

## 参考来源

- arXiv:2606.18051 SkillWeaver: Compositional Skill Routing for LLM Agents
- VentureBeat 报道（Token 降 99%、884K→1.16K、21.1%→大幅领先）
- 复现组件：all-MiniLM-L6-v2 / BGE-base-en-v1.5 + FAISS + LangChain
