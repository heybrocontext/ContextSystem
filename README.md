# ContextSystem

コンテキスト（文脈・エージェントやツールに渡す情報の設計など）に関する**知見**と**実験**を置くリポジトリです。

## ドキュメントと AI エージェントの入口

- **仕様本文（正本）**: **`CONTEXTSYSTEM_SPEC.md`** — 可否・手順・結果・ディレクトリ単位の原則・更新ルールはすべてここに書く。
- **AI エージェント用の入口**: 各エージェントが自動で読むファイルを公式仕様に合わせて置き、**いずれも `CONTEXTSYSTEM_SPEC.md` に誘導する**（本文の二重記述はしない）。
  - `AGENTS.md` — OpenAI Codex CLI / Cursor / GitHub Copilot 系で参照される想定
  - `CLAUDE.md` — Claude Code 用（先頭の `@AGENTS.md` は Claude Code の取り込み構文）
  - `GEMINI.md` — GitHub がエージェント指示の代替として列挙するルート `GEMINI.md` 用（`AGENTS.md` と同文の薄い入口）
  - `.github/copilot-instructions.md` — GitHub Copilot のリポジトリ全体指示としての別経路
- **この README**: 人間向けの概要。

導線の根拠と公式ドキュメントへの参照は `CONTEXTSYSTEM_SPEC.md` の第1節を参照すること。

トップレベルに試みやノートを追加するときは、`CONTEXTSYSTEM_SPEC.md` の「ルート直下スコープ一覧」と、各スコープ内の `AGENTS.md` を更新すること。
