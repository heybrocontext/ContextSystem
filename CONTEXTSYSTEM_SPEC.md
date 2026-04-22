# ContextSystem — AI エージェント向け利用仕様（正）

このファイルが**仕様本文の唯一の正本**です。可否・手順・結果・スコープ一覧・更新規則は**すべてここ**にあります。他のファイル（`AGENTS.md` / `CLAUDE.md` / `.github/copilot-instructions.md`）は、この本文に到達させるためのツール別エントリポイントで、**本文は持ちません**。

## 1. エージェント別エントリポイント（いつ・どの確度で読まれるか）

各 AI エージェントの自動読込仕様は公式ドキュメントで異なる。下表は**本リポジトリに置いた3つの入口ファイル**（`AGENTS.md` / `CLAUDE.md` / `.github/copilot-instructions.md`）が**どのエージェントで・いつ・どれくらい確実に読まれるか**を、公式ドキュメントに基づいて整理したもの。どの経路から来ても**最終的に本ファイル（`CONTEXTSYSTEM_SPEC.md`）を必ず開く**ように、各入口ファイルはその指示だけを含む。

| エージェント / 実行形態 | 読まれる入口ファイル | いつ読まれるか | 確度（公式仕様） | 出典 |
|--------------------------|----------------------|----------------|------------------|------|
| **OpenAI Codex**（CLI / TUI / `codex exec`） | `./AGENTS.md`（`AGENTS.override.md` があればそちらが優先） | セッション開始時（1 run につき 1 回、TUI は起動ごと）。プロジェクトルート→CWD を走査して連結。合計 32 KiB を超えた分は打ち切り | **ルート `AGENTS.md` の非空ファイルは必ず読込**（空・巨大すぎて打ち切り時を除く） | [^codex] |
| **Cursor**（Agent / Chat、ルートの `AGENTS.md`） | `./AGENTS.md` | Agent / Chat のセッション開始時にモデルコンテキスト先頭へ挿入。サブディレクトリ内で作業するとネスト `AGENTS.md` が優先 | **`.cursor/rules` の代替として読込**。Cursor Tab とインライン編集（Cmd/Ctrl+K）には適用されない | [^cursor] |
| **GitHub Copilot コーディングエージェント**（GitHub.com / VS Code / JetBrains / Xcode / Eclipse） | `.github/copilot-instructions.md`＋`./AGENTS.md`（＋`CLAUDE.md` も可） | エージェント実行ごとに対象リポジトリの指示を読込。ディレクトリツリーで最も近い `AGENTS.md` が優先 | **リポジトリ全体指示と Agent 指示の両方に対応** | [^copilot-support][^copilot-repo] |
| **GitHub Copilot コードレビュー** | `.github/copilot-instructions.md` のみ | PR レビュー時。ベースブランチ側の指示を使用 | `AGENTS.md` / `CLAUDE.md` は**使わない**（`.github/copilot-instructions.md` と `.github/instructions/**` のみ） | [^copilot-support] |
| **GitHub Copilot Chat（IDE）** | `.github/copilot-instructions.md`（全 IDE）／`AGENTS.md`（VS Code のみ） | Chat 送信時に自動付与 | VS Code では Agent 指示も読込。Visual Studio / JetBrains / Xcode / Eclipse の Chat は `AGENTS.md` 非対応 | [^copilot-support] |
| **GitHub Copilot CLI** | `.github/copilot-instructions.md`＋`./AGENTS.md` | CLI 実行ごと | 両方対応 | [^copilot-support] |
| **Claude Code**（CLI / IDE 拡張） | `./CLAUDE.md`（または `./.claude/CLAUDE.md`）。本リポジトリでは `CLAUDE.md` 内で `@AGENTS.md` を import | セッション開始時に作業ディレクトリから上方向へ走査して全 `CLAUDE.md` を**フル読込**（`CLAUDE.local.md` も追記） | **`AGENTS.md` は自動では読まない**。`@AGENTS.md` import 経由でのみ到達 | [^claude] |

[^codex]: OpenAI Codex「Custom instructions with AGENTS.md」https://developers.openai.com/codex/guides/agents-md/
[^cursor]: Cursor Docs「Rules — AGENTS.md / FAQ」https://cursor.com/docs/context/rules
[^copilot-support]: GitHub Docs「Support for different types of custom instructions」https://docs.github.com/en/copilot/reference/custom-instructions-support
[^copilot-repo]: GitHub Docs「Adding repository custom instructions for GitHub Copilot」https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions
[^claude]: Anthropic「How Claude remembers your project」https://docs.anthropic.com/en/docs/claude-code/memory

**共通前提（すべてのエージェント）**

- 自動読込は「**入口ファイルの内容**」しか保証しない。本ファイル（`CONTEXTSYSTEM_SPEC.md`）は自動では読まれないため、入口ファイルに「本ファイルを必ず開く」指示を書くことで**強制的に**本ファイルへ合流させる構造にしている。
- 入口ファイルに本文・別ルールを書き足さない（合流させるための指示だけに保つ）。
- 公式仕様の変更に追随する場合は、**入口ファイルを差し替える**だけでよい（本文はこの1ファイルのまま）。

## 2. エージェントが最初に行う手順

クローン直後のエージェントは、上記の入口経由で本ファイルに到達したら、次を順に行う。

1. 本ファイルの **§3・§4・§5** を読む（可否・手順・結果・ディレクトリの原則）。
2. **§6 ルート直下スコープ一覧**を見る。作業対象のディレクトリがあれば、その中の `AGENTS.md` も読む（サブスコープは単体で閉じた文書になっている）。
3. 文書に書かれている範囲だけを前提に作業する。書かれていないことは推測で横断結合しない。

## 3. このリポジトリの役割

コンテキスト（文脈・エージェントに渡す情報の設計など）に関する**知見**と**実験**を置く。  
ルート直下の**各ディレクトリは情報スコープが閉じた単位**であり、原則として**そのディレクトリ単体で利用可能**である。

クローン後にエージェントが把握すべき解像度は次の軸：**いま何が使えるか／使えないか／使える範囲で何ができるか／どうやるか／何が結果か**。下表と各スコープの `AGENTS.md` は同じ軸で埋める。

## 4. 現状：何が使えるか / 使えないか

### 使えること

| 対象 | できること | やり方 | 結果 |
|------|------------|--------|------|
| リポジトリ全体 | 設計原則・エージェント向け仕様の確認 | 本ファイルを読む（必要なら `README.md` も）。入口ファイル経由で来た場合も最終的に本ファイルに到達している前提 | スコープ境界・可否・手順・更新ルールが分かる |
| ルート直下の各ディレクトリ（追加後） | そのスコープ内の手順・実験の実行 | 当該ディレクトリの `AGENTS.md`（または `README.md`）を読み、記載の手順に従う | 各スコープが約束する成果物・ログ・検証結果が得られる |

### 使えないこと（現状）

| 対象 | 理由 |
|------|------|
| ルート直下以外からの横断自動化 | 設計上、ルートの各ディレクトリは相互に前提を持たない |
| 未記載の「暗黙の手順」 | 手順は必ず文書化する。文書にないことはサポート対象外として扱う |

## 5. 設計原則（ディレクトリ単位）

- **ルート直下の各ディレクトリ**がひとつの単位である。
- 各ディレクトリは**情報スコープがそこで閉じる**（他ディレクトリの前提を読まなくてよい）。
- 各ディレクトリは**単体で利用できる**よう、ドキュメント・コード・設定を揃える。

## 6. ルート直下スコープ一覧（エージェント用インデックス）

| ディレクトリ | 単体利用の入口 | いまできること（要約） |
|--------------|----------------|-------------------------|
| （未追加） | — | 実験ディレクトリを追加したら本表と当該 `AGENTS.md` を更新すること |

新規ディレクトリ追加時は、最低限次を満たす。

1. 上表に1行追加する。
2. そのディレクトリ内に **`AGENTS.md`** を置き、「使える／使えない／できること／どうやるか／何が結果か」の5軸を埋める（そのスコープ内だけで完結する内容に限る）。

## 7. メンテナンス契約（システムとしてのルール）

- **仕様本文の正本**はこの `CONTEXTSYSTEM_SPEC.md` のみ。各スコープ内の仕様は当該ディレクトリの `AGENTS.md`。ルートの入口ファイル（`AGENTS.md` / `CLAUDE.md` / `.github/copilot-instructions.md`）は**本文を持たず**、本ファイルへ誘導するだけ。
- 利用可否・手順・期待結果が変わったら、**同じコミットで**本ファイル（と該当スコープの `AGENTS.md`）を更新する。
- 本ファイルをリネームした場合は、入口3ファイルと `README.md` のリンク先を同じコミットで揃える。
- **新しい AI エージェントを追加でサポートしたい場合**は、その公式ドキュメントで自動読込対象のファイル名を確認し、**本ファイルを指すだけの薄い入口ファイル**を追加、§1 の表に1行追記する（本文の複製はしない）。
- エージェントは「文書に書かれたことのみ」を実行可能な能力として扱い、推測で横断結合しない。
