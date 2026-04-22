# ContextSystem

コンテキスト（文脈・エージェントやツールに渡す情報の設計など）に関する**知見**と**実験**を置くリポジトリです。

## ドキュメントと AI エージェントの入口

- **仕様本文（正）**: **`CONTEXTSYSTEM_SPEC.md`** — 可否・手順・結果・ディレクトリ単位の原則・更新ルールはすべてここ。
- **AI エージェント用の入口**: 各エージェントが自動で読むファイルを公式仕様に合わせて配置し、**すべて `CONTEXTSYSTEM_SPEC.md` へ誘導**する（本文の二重記述なし）。
  - `AGENTS.md` — Codex CLI / Cursor / GitHub Copilot 用
  - `CLAUDE.md` — Claude Code 用（`@AGENTS.md` を import）
  - `.github/copilot-instructions.md` — GitHub Copilot 用の別経路
- **人間向けの概要**: 本 README。

導線の根拠と公式仕様の引用は `CONTEXTSYSTEM_SPEC.md` §1 を参照。

トップレベルに試みやノートを追加するときは、`CONTEXTSYSTEM_SPEC.md` の「ルート直下スコープ一覧」と、各スコープ内の `AGENTS.md` を更新してください。
