```py
"""
Claude Code session exporter.

Scans ~/.claude/projects/<cwd>/ for new JSONL sessions,
extracts key decisions via Claude API, writes structured Markdown,
then triggers graphify --update --neo4j-push.

Usage:
    python3 docs/export_sessions.py              # incremental + neo4j push
    python3 docs/export_sessions.py --no-push   # export only, skip push
    python3 docs/export_sessions.py --all       # re-export all sessions
"""

import argparse
import json
import os
import re
import subprocess
import sys
from dataclasses import dataclass, field, asdict
from datetime import datetime, timezone
from pathlib import Path

import anthropic

# ---------- Config ----------

CWD           = Path("/home/linux_zeng/projects/classRelation")
SESSION_DIR   = Path.home() / ".claude/projects/-home-linux-zeng-projects-classRelation"
OUTPUT_DIR    = CWD / "docs/claude-sessions"
MANIFEST_FILE = OUTPUT_DIR / ".exported.json"
GRAPHIFY_BIN  = CWD / "graphify-out/.graphify_python"
NEO4J_URI     = "bolt://localhost:7687"
NEO4J_USER    = "neo4j"
NEO4J_PASS    = "neo4j"
CLAUDE_MODEL  = "claude-sonnet-4-6"

# ---------- Data models ----------

@dataclass
class Decision:
    topic: str
    decision: str
    rationale: str
    alternatives: list[str] = field(default_factory=list)


@dataclass
class SessionSummary:
    session_id: str
    date: str
    title: str
    decisions: list[Decision] = field(default_factory=list)


# ---------- ExportManifest ----------

class ExportManifest:
    def __init__(self):
        self._data: dict = self._load()

    def _load(self) -> dict:
        if MANIFEST_FILE.exists():
            return json.loads(MANIFEST_FILE.read_text(encoding="utf-8"))
        return {"exported": {}}

    def save(self):
        MANIFEST_FILE.write_text(
            json.dumps(self._data, ensure_ascii=False, indent=2),
            encoding="utf-8",
        )

    def is_exported(self, session_id: str) -> bool:
        return session_id in self._data["exported"]

    def mark_exported(self, session_id: str, md_path: str):
        self._data["exported"][session_id] = {
            "exported_at": datetime.now(timezone.utc).isoformat(),
            "md_file": md_path,
        }
        self.save()


# ---------- SessionScanner ----------

class SessionScanner:
    def scan(self, force_all: bool = False) -> list[Path]:
        manifest = ExportManifest()
        files = sorted(SESSION_DIR.glob("*.jsonl"), key=lambda p: p.stat().st_mtime)
        if force_all:
            return files
        return [f for f in files if not manifest.is_exported(f.stem)]


# ---------- SessionParser ----------

class SessionParser:
    """Parse a JSONL session file into (role, text) pairs."""

    def parse(self, path: Path) -> list[tuple[str, str]]:
        messages: list[tuple[str, str]] = []
        try:
            for raw in path.read_text(encoding="utf-8").splitlines():
                raw = raw.strip()
                if not raw:
                    continue
                record = json.loads(raw)
                role = record.get("type", "")
                if role not in ("user", "assistant"):
                    continue
                text = self._extract_text(record)
                if text and len(text) > 20:
                    messages.append((role, text))
        except Exception as e:
            print(f"  [warn] failed to parse {path.name}: {e}")
        return messages

    def _extract_text(self, record: dict) -> str:
        content = record.get("message", {}).get("content", "")
        if isinstance(content, str):
            return content.strip()
        if isinstance(content, list):
            parts = []
            for block in content:
                if isinstance(block, dict) and block.get("type") == "text":
                    parts.append(block.get("text", "").strip())
            return "\n".join(parts).strip()
        return ""

    def session_date(self, path: Path) -> str:
        mtime = path.stat().st_mtime
        return datetime.fromtimestamp(mtime).strftime("%Y-%m-%d")


# ---------- DecisionExtractor ----------

EXTRACT_PROMPT = """\
You are analyzing a Claude Code development session transcript.
Extract the key technical decisions made during this session.

For each decision, identify:
- topic: what problem or question was being addressed
- decision: the specific choice that was made
- rationale: the technical reasoning behind the choice
- alternatives: other options that were considered but rejected

Focus only on concrete technical decisions (architecture, implementation approach, tool choice, design pattern, etc.).
Skip procedural steps, error fixes, and clarifying questions that did not result in a decision.

Output JSON only, no explanation:
{
  "title": "<2-6 word session title>",
  "decisions": [
    {
      "topic": "...",
      "decision": "...",
      "rationale": "...",
      "alternatives": ["...", "..."]
    }
  ]
}

Session transcript:
"""


class DecisionExtractor:
    def __init__(self):
        self._client = anthropic.Anthropic()

    def extract(self, messages: list[tuple[str, str]], session_id: str) -> SessionSummary:
        transcript = self._build_transcript(messages)
        if not transcript:
            return SessionSummary(session_id=session_id, date="", title="Empty session")

        try:
            response = self._client.messages.create(
                model=CLAUDE_MODEL,
                max_tokens=2048,
                messages=[{"role": "user", "content": EXTRACT_PROMPT + transcript}],
            )
            raw = response.content[0].text.strip()
            raw = re.sub(r"^```json\s*", "", raw)
            raw = re.sub(r"\s*```$", "", raw)
            data = json.loads(raw)
            decisions = [Decision(**d) for d in data.get("decisions", [])]
            return SessionSummary(
                session_id=session_id,
                date="",
                title=data.get("title", session_id[:8]),
                decisions=decisions,
            )
        except anthropic.APIStatusError as e:
            if e.status_code == 400 and "credit" in str(e).lower():
                print(f"  [fallback] API credits exhausted, writing raw transcript", end="")
            else:
                print(f"  [fallback] API error ({e.status_code}), writing raw transcript", end="")
            return self._fallback_summary(messages, session_id)
        except Exception as e:
            print(f"  [fallback] {type(e).__name__}, writing raw transcript", end="")
            return self._fallback_summary(messages, session_id)

    def _fallback_summary(self, messages: list[tuple[str, str]], session_id: str) -> SessionSummary:
        """When API is unavailable, embed raw transcript so graphify can extract decisions."""
        return SessionSummary(
            session_id=session_id,
            date="",
            title=f"Session {session_id[:8]}",
            decisions=[
                Decision(
                    topic="_raw_transcript",
                    decision=self._build_transcript(messages),
                    rationale="Extracted by graphify semantic pass",
                    alternatives=[],
                )
            ],
        )

    def _build_transcript(self, messages: list[tuple[str, str]]) -> str:
        lines = []
        for role, text in messages:
            label = "User" if role == "user" else "Claude"
            # Truncate very long blocks to keep prompt manageable
            snippet = text[:1500] + "..." if len(text) > 1500 else text
            lines.append(f"[{label}]\n{snippet}\n")
        return "\n".join(lines)[:40000]  # hard cap ~10k tokens


# ---------- MarkdownWriter ----------

class MarkdownWriter:
    def write(self, summary: SessionSummary, output_dir: Path) -> Path:
        slug = re.sub(r"[^a-z0-9]+", "-", summary.title.lower()).strip("-")
        filename = f"{summary.date}-{slug}.md"
        path = output_dir / filename

        lines = [
            f"# Claude Session: {summary.title}",
            f"**Date:** {summary.date}  ",
            f"**Session ID:** `{summary.session_id}`  ",
            f"**Project:** classRelation",
            "",
        ]

        if not summary.decisions:
            lines += ["*No key decisions extracted from this session.*", ""]
        elif len(summary.decisions) == 1 and summary.decisions[0].topic == "_raw_transcript":
            # Fallback mode: write raw conversation for graphify semantic extraction
            lines += ["## 对话记录\n", summary.decisions[0].decision, ""]
        else:
            for i, d in enumerate(summary.decisions, 1):
                lines += [
                    f"## 决策 {i}：{d.topic}",
                    "",
                    f"**选择：** {d.decision}",
                    "",
                    f"**理由：** {d.rationale}",
                    "",
                ]
                if d.alternatives:
                    lines.append("**排除方案：**")
                    for alt in d.alternatives:
                        lines.append(f"- {alt}")
                    lines.append("")

        path.write_text("\n".join(lines), encoding="utf-8")
        return path


# ---------- GraphifyRunner ----------

class GraphifyRunner:
    def run_update_push(self):
        python = str(GRAPHIFY_BIN) if GRAPHIFY_BIN.exists() else "python3"
        cmd = [
            python, "-c",
            f"""
import json
from graphify.build import build_from_json
from graphify.cluster import cluster
from graphify.export import push_to_neo4j
from graphify.detect import detect, save_manifest
from graphify.extract import collect_files, extract
from graphify.cache import check_semantic_cache
from pathlib import Path

# Detect new/changed files
detection = detect(Path('.'))
save_manifest(detection['files'])

# Reload existing graph and push
import networkx as nx
from networkx.readwrite import json_graph
graph_path = Path('graphify-out/graph.json')
if not graph_path.exists():
    print('No graph.json found - run /graphify first')
    raise SystemExit(1)
G = json_graph.node_link_graph(json.loads(graph_path.read_text()), edges='links')
communities = {{int(n): d.get('community', 0) for n, d in G.nodes(data=True)}}
result = push_to_neo4j(G, uri='{NEO4J_URI}', user='{NEO4J_USER}', password='{NEO4J_PASS}', communities=communities)
print(f'Neo4j push: {{result[\"nodes\"]}} nodes, {{result[\"edges\"]}} edges')
""",
        ]
        print("Running graphify update + neo4j push...")
        try:
            subprocess.run(cmd, cwd=str(CWD), check=True)
        except subprocess.CalledProcessError as e:
            print(f"  [warn] graphify push failed (exit {e.returncode})")


# ---------- Main ----------

def main():
    parser = argparse.ArgumentParser(description="Export Claude Code sessions to Markdown")
    parser.add_argument("--no-push", action="store_true", help="Skip neo4j push")
    parser.add_argument("--all", action="store_true", dest="force_all", help="Re-export all sessions")
    args = parser.parse_args()

    OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

    scanner   = SessionScanner()
    parser_   = SessionParser()
    extractor = DecisionExtractor()
    writer    = MarkdownWriter()
    manifest  = ExportManifest()

    new_files = scanner.scan(force_all=args.force_all)
    if not new_files:
        print("No new sessions to export.")
        return

    print(f"Found {len(new_files)} new session(s) to export.")
    exported = []

    for path in new_files:
        session_id = path.stem
        date = parser_.session_date(path)
        print(f"\n[{date}] {session_id[:8]}... ", end="", flush=True)

        messages = parser_.parse(path)
        if not messages:
            print("empty, skipping.")
            continue

        print(f"{len(messages)} messages → extracting decisions...", end="", flush=True)
        summary = extractor.extract(messages, session_id)
        summary.date = date

        md_path = writer.write(summary, OUTPUT_DIR)
        manifest.mark_exported(session_id, str(md_path.relative_to(CWD)))

        decision_count = len(summary.decisions)
        print(f" {decision_count} decisions → {md_path.name}")
        exported.append(md_path)

    print(f"\nExported {len(exported)} session(s) to {OUTPUT_DIR}/")

    if exported and not args.no_push:
        GraphifyRunner().run_update_push()


if __name__ == "__main__":
    main()

```

