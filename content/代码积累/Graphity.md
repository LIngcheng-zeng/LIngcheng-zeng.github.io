 /graphify . --neo4j-push bolt://localhost:7687

  执行时它会调用 graphify.export.push_to_neo4j()，用 MERGE 写入，幂等安全。

---
  当前状态

  我们刚才用手动脚本已经导入了一次（438 节点 + 675 边），数据已在 Neo4j 里。

  如果后续重新运行 /graphify（比如代码更新后），推荐用内置标志替代手动脚本：

  /graphify . --neo4j-push bolt://localhost:7687 --neo4j-user neo4j --neo4j-password xxx

  或者如果只想生成 Cypher 文件而不直推：

  /graphify . --neo4j



 后续每次代码更新后，只需运行：

  /graphify . --neo4j-push bolt://localhost:7687





❯ 源代码的git 提交记录 以及claude的对话历史怎么提交到graphify

● 两个来源需要分别处理：

---
  1. Git 提交记录

  导出为 Markdown，放进 graphify 能扫描到的目录：

  ! git log --pretty=format:"## %h %s%n**Author:** %an  **Date:** %ad%n%n%b%n---" \
    --date=short > docs/git-history.md

  然后增量更新图：

  /graphify . --update --neo4j-push bolt://localhost:7687

  graphify 会把每条 commit 提取为节点，%b（commit body）里的关键词会形成关系边，与代码节点产生 references 连接。

---
  2. Claude 对话历史

  Claude Code 没有自动导出接口，需要手动保存。两种方式：

  方式 A：直接粘贴到 Markdown

  新建 docs/claude-sessions/2026-04-11-neo4j-import.md，将对话内容粘贴进去，然后跑 --update。

  方式 B：用 /graphify add 逐条添加关键决策

  /graphify add "决策：graphify 导入 Neo4j 时节点标签用 :GraphifyNode 而非 :Entity，避免与 Graphiti 冲突"

  这会作为一个独立节点写入图，并与相关代码节点产生 rationale_for 边——比粘贴全文更有价值，因为它直接记录了决策理由。







 根据 graphify skill 的文档，完整命令列表：

  构建 / 更新

  ┌────────────────────────────┬──────────────────────────────┐
  │            命令            │             说明             │
  ├────────────────────────────┼──────────────────────────────┤
  │ /graphify .                │ 全量构建当前目录             │
  ├────────────────────────────┼──────────────────────────────┤
  │ /graphify . --update       │ 增量更新（只处理变更文件）   │
  ├────────────────────────────┼──────────────────────────────┤
  │ /graphify . --mode deep    │ 深度提取，更多 INFERRED 边   │
  ├────────────────────────────┼──────────────────────────────┤
  │ /graphify . --cluster-only │ 仅重跑社区聚类，不重提取     │
  ├────────────────────────────┼──────────────────────────────┤
  │ /graphify . --no-viz       │ 跳过 HTML，只生成报告 + JSON │
  └────────────────────────────┴──────────────────────────────┘

  导出

  ┌────────────────────────────────────────────────┬────────────────────────────────┐
  │                      命令                      │              说明              │
  ├────────────────────────────────────────────────┼────────────────────────────────┤
  │ /graphify . --neo4j                            │ 生成 cypher.txt 供手动导入     │
  ├────────────────────────────────────────────────┼────────────────────────────────┤
  │ /graphify . --neo4j-push bolt://localhost:7687 │ 直推 Neo4j                     │
  ├────────────────────────────────────────────────┼────────────────────────────────┤
  │ /graphify . --svg                              │ 导出 SVG（嵌入 Notion/GitHub） │
  ├────────────────────────────────────────────────┼────────────────────────────────┤
  │ /graphify . --graphml                          │ 导出 GraphML（Gephi/yEd）      │
  ├────────────────────────────────────────────────┼────────────────────────────────┤
  │ /graphify . --obsidian                         │ 生成 Obsidian vault            │
  ├────────────────────────────────────────────────┼────────────────────────────────┤
  │ /graphify . --wiki                             │ 生成 agent 可爬取的 wiki       │
  └────────────────────────────────────────────────┴────────────────────────────────┘

  查询

  ┌──────────────────────────────────────┬────────────────────────────┐
  │                 命令                 │            说明            │
  ├──────────────────────────────────────┼────────────────────────────┤
  │ /graphify query "问题"               │ BFS 广度遍历（宽泛上下文） │
  ├──────────────────────────────────────┼────────────────────────────┤
  │ /graphify query "问题" --dfs         │ DFS 深度追踪（路径链路）   │
  ├──────────────────────────────────────┼────────────────────────────┤
  │ /graphify query "问题" --budget 1500 │ 限制输出 token 数          │
  ├──────────────────────────────────────┼────────────────────────────┤
  │ /graphify path "A" "B"               │ 两个概念之间的最短路径     │
  ├──────────────────────────────────────┼────────────────────────────┤
  │ /graphify explain "类名"             │ 解释一个节点的所有关联     │
  └──────────────────────────────────────┴────────────────────────────┘

  增量摄入

  ┌─────────────────────────────────────┬────────────────────────────────┐
  │                命令                 │              说明              │
  ├─────────────────────────────────────┼────────────────────────────────┤
  │ /graphify add <url>                 │ 抓取 URL 加入语料库并更新图    │
  ├─────────────────────────────────────┼────────────────────────────────┤
  │ /graphify add <url> --author "Name" │ 标注作者                       │
  ├─────────────────────────────────────┼────────────────────────────────┤
  │ /graphify . --watch                 │ 监听文件变更，代码改动自动重建 │
  └─────────────────────────────────────┴────────────────────────────────┘

  自动化

  ┌─────────────────────────┬─────────────────────────────────────────────┐
  │          命令           │                    说明                     │
  ├─────────────────────────┼─────────────────────────────────────────────┤
  │ graphify hook install   │ 安装 git post-commit 钩子，每次提交自动重建 │
  ├─────────────────────────┼─────────────────────────────────────────────┤
  │ graphify claude install │ 写入 CLAUDE.md，让 Claude Code 自动使用图   │
  ├─────────────────────────┼─────────────────────────────────────────────┤
  │ /graphify . --mcp       │ 启动 MCP stdio server，供其他 agent 查询    │
  └─────────────────────────┴─────────────────────────────────────────────┘

---
  当前项目最常用的组合：

  /graphify . --update --neo4j-push bolt://localhost:7687





---
  1. 查所有决策节点

  // 全部 Decision 节点
  MATCH (n:GraphifyNode)
  WHERE n.label STARTS WITH 'Decision:'
  RETURN n.label, n.source_file
  ORDER BY n.source_file

  // 全部 Rationale（设计理由）节点
  MATCH (n:GraphifyNode)
  WHERE n.label STARTS WITH 'Rationale:'
  RETURN n.label, n.source_file

---
  2. 决策 → 影响的代码节点

  // 某个决策影响了哪些模块？
  MATCH (d:GraphifyNode)-[r]->(target:GraphifyNode)
  WHERE d.label STARTS WITH 'Decision:'
  RETURN d.label AS decision, type(r) AS relation, target.label AS affects
  ORDER BY d.label

  // Rationale 指向它解释的概念（RATIONALE_FOR 关系）
  MATCH (r:GraphifyNode)-[:RATIONALE_FOR]->(concept:GraphifyNode)
  WHERE r.label STARTS WITH 'Rationale:'
  RETURN r.label AS rationale, concept.label AS explains

---
  3. 按会话查决策时间线

  // 每个 Session 包含哪些决策
  MATCH (s:GraphifyNode)-[r]->(d:GraphifyNode)
  WHERE s.label STARTS WITH 'Session '
    AND d.label STARTS WITH 'Decision:'
  RETURN s.label AS session, collect(d.label) AS decisions
  ORDER BY s.source_file

  // 所有 Session 的时间线（按文件名排序即时间顺序）
  MATCH (n:GraphifyNode)
  WHERE n.label STARTS WITH 'Session '
  RETURN n.label, n.source_file
  ORDER BY n.source_file

---
  4. 查某个决策的完整上下文

  // 以 "MAP_JOIN" 为关键词追溯决策
  MATCH (n:GraphifyNode)
  WHERE n.label CONTAINS 'MAP_JOIN'
     OR n.label CONTAINS 'Implicit Equality'
     OR n.label CONTAINS 'ProvenanceContext'
  RETURN n.label, n.source_file, n.source_location

  // 两跳内：MAP_JOIN 决策 → 相关节点 → 再延伸一层
  MATCH path = (d:GraphifyNode)-[*1..2]-(neighbor:GraphifyNode)
  WHERE d.label CONTAINS 'MAP_JOIN'
  RETURN path
  LIMIT 50

---
  5. 决策之间的关联（跨会话影响链）

  // 所有 INFERRED 边连接的决策节点（推断出的潜在关联）
  MATCH (a:GraphifyNode)-[r]->(b:GraphifyNode)
  WHERE (a.label STARTS WITH 'Decision:' OR a.label STARTS WITH 'Rationale:')
    AND r.confidence = 'INFERRED'
  RETURN a.label AS from, type(r), b.label AS to, r.confidence_score
  ORDER BY r.confidence_score DESC

---
  6. 决策按主题分组（超边查询）

  // 哪些超边包含决策节点
  MATCH (h:HyperEdge)
  RETURN h.label, h.confidence_score
  ORDER BY h.confidence_score DESC

  // 展开超边成员
  MATCH (h:HyperEdge)-[:MEMBER]->(n:GraphifyNode)
  WHERE h.label CONTAINS 'Decision' OR h.label CONTAINS 'Protocol'
  RETURN h.label AS group, collect(n.label) AS members

---
  当前图中的决策快照

  已入图的决策节点（来自历史会话）：

  ┌───────────────────────────────────────────────┬──────────────────┐
  │                     决策                      │     来源会话     │
  ├───────────────────────────────────────────────┼──────────────────┤
  │ Implement Write-Side Assignment (MappingMode) │ session-639bf785 │
  ├───────────────────────────────────────────────┼──────────────────┤
  │ Introduce SourceAnalyzer SPI                  │ session-a53e458b │
  ├───────────────────────────────────────────────┼──────────────────┤
  │ Render Separate Mermaid Blocks per Component  │ session-a53e458b │
  ├───────────────────────────────────────────────┼──────────────────┤
  │ Unify Internal Class Names to FQN             │ session-63a8345c │
  ├───────────────────────────────────────────────┼──────────────────┤
  │ Export Claude Sessions as Markdown            │ session-1c549f2c │
  ├───────────────────────────────────────────────┼──────────────────┤
  │ Use :GraphifyNode Label (Neo4j collision fix) │ session-1c549f2c │
  ├───────────────────────────────────────────────┼──────────────────┤
  │ Rationale: ProvenanceContext uses Composition │ 本次会话         │
  ├───────────────────────────────────────────────┼──────────────────┤
  │ Rationale: Two-Phase Protocol design          │ 本次会话         │
  └───────────────────────────────────────────────┴──────────────────┘

  注：本次会话（MAP_JOIN 全套实现）尚未有对应的 session-xxxx.md 文件（export_sessions.py 显示"No new
  sessions"），等会话落盘后执行一次 /graphify --update 即可将本次决策完整入图。

  按内容查询决策（现在可展开正文）

  // 查所有 Decision 节点，展示完整正文
  MATCH (n:GraphifyNode)
  WHERE n.label STARTS WITH 'Decision:'
  RETURN n.label AS 决策,
         n.source_file AS 来源文件,
         n.description AS 正文
  ORDER BY n.source_file

  // 按关键词全文搜索决策正文
  MATCH (n:GraphifyNode)
  WHERE (n.label STARTS WITH 'Decision:' OR n.label STARTS WITH 'Rationale:')
    AND n.description CONTAINS 'SPI'   -- 替换为你要搜索的词
  RETURN n.label, n.description

  // 查 Session 摘要
  MATCH (n:GraphifyNode)
  WHERE n.label STARTS WITH 'Session '
  RETURN n.label AS 会话, n.description AS 摘要
  ORDER BY n.source_file

  // Rationale → 它解释的概念 + 正文
  MATCH (r:GraphifyNode)-[:RATIONALE_FOR]->(concept:GraphifyNode)
  WHERE r.label STARTS WITH 'Rationale:'
  RETURN r.label AS 设计理由,
         r.description AS 正文,
         concept.label AS 解释的概念

```py
"""
Enrich Neo4j decision/rationale nodes with description text extracted from source files.

Strategy per node type:
  Decision / Rationale nodes  — keyword search in source markdown, extract surrounding paragraph
  Session nodes               — extract the summary section (## 摘要 or first [Claude] paragraph)

Run after neo4j_import.py whenever new session files are added.
"""

import json
import re
from pathlib import Path
from neo4j import GraphDatabase

NEO4J_URI  = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASS = "neo4j_zlc"
GRAPH_FILE = Path(__file__).parent / "graph.json"
PROJECT_ROOT = Path(__file__).parent.parent   # classRelation/

WINDOW_CHARS = 600   # chars to capture around the matched keyword


# ── text extraction helpers ────────────────────────────────────────────────────

def _read_file(source_file: str) -> str:
    """Read source file relative to project root, return empty string on failure."""
    path = PROJECT_ROOT / source_file
    if not path.exists():
        return ""
    try:
        return path.read_text(encoding="utf-8")
    except Exception:
        return ""


def _extract_around_keyword(text: str, keywords: list[str], window: int = WINDOW_CHARS) -> str:
    """
    Find the first keyword match in text and return `window` chars of surrounding context.
    Falls back to the first non-empty paragraph if no keyword matches.
    """
    text_lower = text.lower()
    best_pos = len(text)

    for kw in keywords:
        pos = text_lower.find(kw.lower())
        if 0 <= pos < best_pos:
            best_pos = pos

    if best_pos < len(text):
        start = max(0, best_pos - 80)
        end   = min(len(text), best_pos + window)
        snippet = text[start:end].strip()
        # trim to clean paragraph boundaries
        snippet = re.sub(r'\n{3,}', '\n\n', snippet)
        return snippet

    # fallback: first non-trivial paragraph
    for para in text.split('\n\n'):
        para = para.strip()
        if len(para) > 80 and not para.startswith('#'):
            return para[:window]
    return text[:window].strip()


def _extract_session_summary(text: str) -> str:
    """
    Extract the summary/key-points section from a Claude session markdown.
    Looks for ## 摘要, ## Summary, ## 关键决策, or falls back to the first Claude turn.
    """
    # Try explicit summary sections
    for header in ['## 摘要', '## Summary', '## 关键决策', '## Key Decisions', '## 决策']:
        idx = text.find(header)
        if idx >= 0:
            section = text[idx: idx + 1200].strip()
            return re.sub(r'\n{3,}', '\n\n', section)

    # Fall back: first [Claude] block
    match = re.search(r'\[Claude\]\s*\n+(.*?)(?=\n\[|\Z)', text, re.DOTALL)
    if match:
        return match.group(1).strip()[:WINDOW_CHARS]

    return text[:WINDOW_CHARS].strip()


def build_description(node: dict) -> str:
    label       = node.get("label", "")
    source_file = node.get("source_file", "")
    text        = _read_file(source_file)

    if not text:
        return ""

    if label.startswith("Session "):
        return _extract_session_summary(text)

    # For Decision / Rationale nodes: use significant words from label as keywords
    stopwords = {"decision","rationale","session","the","and","for","of","to","in",
                 "a","an","is","are","was","with","from","by","or","that","this",
                 "use","uses","into","via","as","be","on","at","its","their"}
    keywords = [
        w for w in re.split(r'[\s:()→\-/]+', label)
        if len(w) > 3 and w.lower() not in stopwords
    ]
    return _extract_around_keyword(text, keywords)


# ── Neo4j update ──────────────────────────────────────────────────────────────

UPDATE_CQL = """
UNWIND $batch AS row
MATCH (n:GraphifyNode {id: row.id})
SET n.description = row.description
"""


def enrich(driver, nodes: list[dict]):
    target_prefixes = ("Decision:", "Rationale:", "Session ")
    candidates = [
        n for n in nodes
        if any(n.get("label", "").startswith(p) for p in target_prefixes)
    ]
    print(f"Candidate nodes to enrich: {len(candidates)}")

    batch = []
    for n in candidates:
        desc = build_description(n)
        if desc:
            batch.append({"id": n["id"], "description": desc})

    print(f"Nodes with extracted description: {len(batch)}")

    if not batch:
        print("Nothing to update.")
        return

    with driver.session() as s:
        s.run(UPDATE_CQL, batch=batch)
    print(f"Updated {len(batch)} node(s) in Neo4j.")

    # Preview
    print("\n── Preview (first 3) ──────────────────────────────")
    for item in batch[:3]:
        preview = item['description'].replace('\n', ' ')[:120]
        print(f"  [{item['id'][:40]}]\n  → {preview}…\n")


# ── main ──────────────────────────────────────────────────────────────────────

if __name__ == "__main__":
    graph = json.loads(GRAPH_FILE.read_text(encoding="utf-8"))
    # networkx node_link format: nodes are under 'nodes' key
    nodes = graph.get("nodes", [])

    driver = GraphDatabase.driver(NEO4J_URI, auth=(NEO4J_USER, NEO4J_PASS))
    try:
        enrich(driver, nodes)
    finally:
        driver.close()

```



```python
"""
Import graphify graph.json into Neo4j.

Node labels:  :GraphifyNode + :Code or :Document (based on file_type)
Relationships: dynamic type from 'relation' field (spaces → _)
Hyperedges:   represented as :HyperEdge node + :MEMBER_OF relationships
NOTE: Avoiding :Entity label — already used by Graphiti (Zep)
"""

import json
import re
from pathlib import Path
from neo4j import GraphDatabase

NEO4J_URI  = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASS = "neo4j_zlc"
GRAPH_FILE = Path(__file__).parent / "graph.json"

BATCH_SIZE = 100


def sanitize_rel_type(rel: str) -> str:
    """Convert relation string to valid Neo4j relationship type."""
    return re.sub(r"[^a-zA-Z0-9_]", "_", rel).upper()


def load_graph() -> dict:
    with open(GRAPH_FILE, encoding="utf-8") as f:
        return json.load(f)


# ---------- Node import ----------

NODE_UPSERT = """
UNWIND $batch AS row
MERGE (n:GraphifyNode {id: row.id})
SET n.label        = row.label,
    n.file_type    = row.file_type,
    n.source_file  = row.source_file,
    n.source_loc   = row.source_location,
    n.community    = row.community,
    n.description  = row.description
WITH n, row
CALL apoc.create.addLabels(n, [row.extra_label]) YIELD node
RETURN count(node)
"""

NODE_UPSERT_NO_APOC = """
UNWIND $batch AS row
MERGE (n:GraphifyNode {id: row.id})
SET n.label        = row.label,
    n.file_type    = row.file_type,
    n.source_file  = row.source_file,
    n.source_loc   = row.source_location,
    n.community    = row.community,
    n.description  = row.description
RETURN count(n)
"""


def import_nodes(session, nodes: list, use_apoc: bool):
    batch = []
    for n in nodes:
        ft = n.get("file_type", "")
        batch.append({
            "id":            n["id"],
            "label":         n.get("label", ""),
            "file_type":     ft,
            "source_file":   n.get("source_file", ""),
            "source_location": n.get("source_location", ""),
            "community":     n.get("community", -1),
            "description":   n.get("description", ""),
            "extra_label":   "Code" if ft == "code" else "Document",
        })
        if len(batch) == BATCH_SIZE:
            _flush_nodes(session, batch, use_apoc)
            batch = []
    if batch:
        _flush_nodes(session, batch, use_apoc)


def _flush_nodes(session, batch, use_apoc):
    if use_apoc:
        try:
            session.run(NODE_UPSERT, batch=batch)
            return
        except Exception:
            pass
    session.run(NODE_UPSERT_NO_APOC, batch=batch)


# ---------- Relationship import ----------
# Neo4j doesn't support dynamic rel types in a single Cypher statement,
# so we group links by relation type and issue one query per type.

REL_UPSERT_TMPL = (
    "UNWIND $batch AS row "
    "MATCH (a:GraphifyNode {{id: row.src}}) "
    "MATCH (b:GraphifyNode {{id: row.tgt}}) "
    "MERGE (a)-[r:{rel_type}]->(b) "
    "SET r.weight           = row.weight, "
    "    r.confidence       = row.confidence, "
    "    r.confidence_score = row.confidence_score, "
    "    r.source_file      = row.source_file, "
    "    r.source_loc       = row.source_location "
    "RETURN count(r)"
)


def import_links(session, links: list):
    # Group by sanitized relation type
    by_type: dict[str, list] = {}
    for lk in links:
        rt = sanitize_rel_type(lk.get("relation", "RELATED"))
        by_type.setdefault(rt, []).append({
            "src":            lk["source"],
            "tgt":            lk["target"],
            "weight":         lk.get("weight", 1.0),
            "confidence":     lk.get("confidence", ""),
            "confidence_score": lk.get("confidence_score", 1.0),
            "source_file":    lk.get("source_file", ""),
            "source_location": lk.get("source_location", ""),
        })

    for rel_type, rows in by_type.items():
        cypher = REL_UPSERT_TMPL.format(rel_type=rel_type)
        for i in range(0, len(rows), BATCH_SIZE):
            session.run(cypher, batch=rows[i:i + BATCH_SIZE])
        print(f"  [{rel_type}] {len(rows)} relationships imported")


# ---------- Hyperedge import ----------
# Strategy: create a :HyperEdge node, then connect each member via :MEMBER_OF

HYPER_NODE = """
MERGE (h:HyperEdge {id: $id})
SET h.label            = $label,
    h.relation         = $relation,
    h.confidence       = $confidence,
    h.confidence_score = $confidence_score,
    h.source_file      = $source_file
"""

HYPER_MEMBER = """
UNWIND $members AS mid
MATCH (n:GraphifyNode {id: mid})
MATCH (h:HyperEdge {id: $hid})
MERGE (n)-[:MEMBER_OF]->(h)
"""


def import_hyperedges(session, hyperedges: list):
    for he in hyperedges:
        session.run(HYPER_NODE,
            id=he["id"],
            label=he.get("label", ""),
            relation=he.get("relation", ""),
            confidence=he.get("confidence", ""),
            confidence_score=he.get("confidence_score", 1.0),
            source_file=he.get("source_file", ""),
        )
        session.run(HYPER_MEMBER, hid=he["id"], members=he.get("nodes", []))
    print(f"  {len(hyperedges)} hyperedges imported")


# ---------- Index / Constraint ----------

CONSTRAINTS = [
    "CREATE CONSTRAINT graphifynode_id IF NOT EXISTS FOR (n:GraphifyNode) REQUIRE n.id IS UNIQUE",
    "CREATE CONSTRAINT hyperedge_id IF NOT EXISTS FOR (h:HyperEdge) REQUIRE h.id IS UNIQUE",
]


def create_constraints(session):
    for c in CONSTRAINTS:
        try:
            session.run(c)
        except Exception as e:
            print(f"  [warn] constraint skipped: {e}")


# ---------- Entry point ----------

def main():
    graph = load_graph()
    nodes      = graph.get("nodes", [])
    links      = graph.get("links", [])
    hyperedges = graph.get("hyperedges", [])

    print(f"Loaded: {len(nodes)} nodes, {len(links)} links, {len(hyperedges)} hyperedges")

    driver = GraphDatabase.driver(NEO4J_URI, auth=(NEO4J_USER, NEO4J_PASS))
    try:
        with driver.session() as session:
            print("Creating constraints...")
            create_constraints(session)

            print("Importing nodes...")
            # Detect APOC availability
            use_apoc = False
            try:
                session.run("RETURN apoc.version()").single()
                use_apoc = True
                print("  APOC detected — using dynamic labels")
            except Exception:
                print("  APOC not found — skipping extra labels (Code/Document)")
            import_nodes(session, nodes, use_apoc)
            print(f"  {len(nodes)} nodes imported")

            print("Importing relationships...")
            import_links(session, links)

            print("Importing hyperedges...")
            import_hyperedges(session, hyperedges)

    finally:
        driver.close()

    print("\nDone. Verify in Neo4j Browser:")
    print("  MATCH (n:GraphifyNode) RETURN count(n)")
    print("  MATCH ()-[r]->() RETURN type(r), count(r) ORDER BY count(r) DESC")
    print("  MATCH (h:HyperEdge) RETURN h.label")


if __name__ == "__main__":
    main()

```

