日本語 | [English](README_EN.md) | [中文](README.md)

> 日本語版はコミュニティによりメンテナンスされています。内容が最新でない場合は、中文版・英語版を正としてください。

> 🍴 **これはフォークです**：[xbtlin/ai-berkshire](https://github.com/xbtlin/ai-berkshire) からフォークしました。このフォークでの変更点：`skills/*.md` をすべて agent 中立な動作記述に書き換え（Claude Code 専用の `TeamCreate`/`SendMessage`/`TeamDelete` などのツール呼び出しを廃止）、**GitHub Copilot CLI** でも正しく実行できるようにしました（Claude Code / Codex との互換性はそのまま維持）。Copilot CLI 向けのツール対応表をまとめた `COPILOT.md` を追加しています。詳細は [`docs/superpowers/specs/`](docs/superpowers/specs/) の設計ドキュメントと [`docs/superpowers/plans/`](docs/superpowers/plans/) の実装計画をご覧ください。

# AI Berkshire — AI時代のバリュー投資リサーチフレームワーク

> 「価格はあなたが払うもの、価値はあなたが得るもの」 — ウォーレン・バフェット

**AI Berkshire** は、Claude Code、Codex、GitHub Copilot CLIに対応した投資リサーチSkillのコレクションです。バフェット・マンガー・段永平（ダン・ヨンピン）・李録（リ・ルー）という4人のバリュー投資の巨人の方法論を体系化し、AIエージェントによりプロフェッショナル水準のリサーチを提供します。

[設計思想](#設計思想) · [Skills一覧](#skills一覧19個) · [Setup Guide](#setup-guide) · [使い方](#使い方) · [実践レポート](#実践リサーチレポート)

---

## 設計思想

AIに直接「この株は買うべきか」と聞くと、たいてい「一方では…、他方では…」というバランス型の分析が返ってきて、意思決定には使えません。AI Berkshireが解決するのは分析の質と意思決定の規律の問題です：

- **4人の巨人による対抗、単一分析ではない**：段永平（ビジネス本質）、バフェット（護城河と評価）、マンガー（逆思考）、李録（長期的確実性）の4つの視点がそれぞれ独立してリサーチし、互いに挑戦し合う——無難にまとめられた1本のレポートにはしません
- **バイアス対策メカニズム**：情報充実度の等級付け（A/B/C）、失敗シナリオの逆算、レッドラインの拒否リスト、反コンセンサスチェックにより「情報量が多い＝確実性が高い」という幻想を防ぐ
- **金融データの精度**：すべての計算は [`tools/financial_rigor.py`](tools/financial_rigor.py)（Python `decimal.Decimal` による正確な十進計算、`float` は使わない）を経由し、LLMの暗算に頼らない。重要データは最低2つの独立ソースでクロス検証
- **再現性**：同じ入力から構造・深さが一貫したレポートを生成し、企業間・時系列での比較がしやすい
- **マルチAgent並列**：`/investment-team` などのチーム型skillは複数の独立したサブタスクを並列起動する——1つのプロンプトを分割するのではなく、各役割が独立してリサーチ・判断し、最後に統合する

### アーキテクチャ

3層構造：
- **Skill層**：19の明確なエントリーポイントを、シーン（深掘りリサーチ／決算分析／業界スクリーニング／ポートフォリオ管理／思考ツール）に応じて選択
- **Agent層**：チーム型skill（`/investment-team`、`/earnings-team` など）が複数の独立視点のサブタスクを並列起動——それぞれが調査・判断・相互挑戦した後、最終統合する。軽量skillはこの層を経由せず直接ツールへ
- **Tool層**：`tools/financial_rigor.py` などが正確な計算・クロス検証・レポート監査を担当し、各レポートのデータの厳密性を検証可能にする

---

## Skills一覧（19個）

### 🔬 深掘りリサーチ系

| Skill | 用途 | 適したシーン |
|-------|------|---------|
| [`/investment-research`](skills/investment-research.md) | 4人の巨人による総合分析 | 上場企業の全方位投資リサーチ |
| [`/investment-team`](skills/investment-team.md) | マルチAgent並列リサーチチーム | 4つの役割が並列リサーチ——最速かつ最も包括的 |
| [`/management-deep-dive`](skills/management-deep-dive.md) | 経営陣の深掘りリサーチ | 「株を買うことは人を買うこと」——経営陣が核心変数の時に深掘り |
| [`/private-company-research`](skills/private-company-research.md) | 未上場企業の深掘りリサーチ | Ant Group、SpaceXなど情報が少ない未上場企業を調査 |
| [`/deep-company-series`](skills/deep-company-series.md) | 1社を8本の長文シリーズで解剖 | 認知のリセットから意思決定までの長文深掘りシリーズ |

### 📊 決算分析系

| Skill | 用途 | 適したシーン |
|-------|------|---------|
| [`/earnings-review`](skills/earnings-review.md) | 決算の精読（一次情報） | 二次情報に頼らず原文決算を読む——バフェットのように年次報告書を読む |
| [`/earnings-team`](skills/earnings-team.md) | 決算精読チーム＋記事公開 | 4人の巨人が並列で決算を解読 → 編集で推敲 → 読者レビュー → 公開可能な記事に |

### 🏭 業界スクリーニング系

| Skill | 用途 | 適したシーン |
|-------|------|---------|
| [`/industry-research`](skills/industry-research.md) | 産業チェーン全景スキャン | 業界全体の投資機会をチェーンの各段階ごとに調査 |
| [`/industry-funnel`](skills/industry-funnel.md) | 業界ファネルスクリーニング | 全市場 → ≤10社に絞り込み → 最終3社を深掘り分析 |
| [`/quality-screen`](skills/quality-screen.md) | 除外スクリーニング（7つの硬い指標） | 一流でない企業を素早く除外、個別株/業界/指数/テーマの一括スクリーニングに対応 |
| [`/bottleneck-hunter`](skills/bottleneck-hunter.md) | サプライチェーン・ボトルネックハンター | メガトレンドを起点に、産業チェーン上の物理的ボトルネックと裁定機会を探す |
| [`/investment-checklist`](skills/investment-checklist.md) | バフェット式購入前チェックリスト | 6つの関門で、10分で深掘りする価値があるか判断 |

### 📈 ポートフォリオ管理系

| Skill | 用途 | 適したシーン |
|-------|------|---------|
| [`/portfolio-review`](skills/portfolio-review.md) | ポートフォリオ管理と最適化 | 「企業をリサーチする」から「ポートフォリオを管理する」へ——ポジション、集中度、リバランス |
| [`/thesis-tracker`](skills/thesis-tracker.md) | 投資論文トラッキング | 購入後の規律システム：論文が反証されていないか継続的に追跡 |
| [`/thesis-drift`](skills/thesis-drift.md) | 投資論文ドリフト検出 | 2つの論文/レポートを比較し、事実の変化・評価の変化・表現の変化を区別 |
| [`/news-pulse`](skills/news-pulse.md) | 株価急変動の迅速な原因分析 | 株価が急騰/急落した時、10分で「何が起きたか」を把握 |

### 🧠 思考ツール系

| Skill | 用途 | 適したシーン |
|-------|------|---------|
| [`/dyp-ask`](skills/dyp-ask.md) | 段永平に問う | 段永平の思考法であらゆる問題（ビジネス、投資、人生）を考える |
| [`/financial-data`](skills/financial-data.md) | 財務データ取得とクロス検証の規範 | 重要データが2つの独立ソースから得られていることを保証し、誤差>1%を警告 |
| [`/wechat-article`](skills/wechat-article.md) | 長文レポートを記事に変換 | 著者・編集者・読者の3つの役割が協働し、リサーチレポートを公開可能な記事に書き直す |

---

## Setup Guide

### 1. AIクライアントのインストール

このリポジトリは1つの agent 中立な標準ワークフローを維持し、Claude Codeコマンド、Codex skill、そして skills.sh エコシステム経由でGitHub Copilot CLIなど対応する汎用agent向けにインストール可能なパッケージを提供します。使用するクライアントをインストールしてください。

**Claude Code：**

```bash
npm install -g @anthropic-ai/claude-code
```

**Codex：**

```bash
# macOS / Linux
curl -fsSL https://chatgpt.com/codex/install.sh | sh

# または npm
npm install -g @openai/codex

# または Homebrew
brew install --cask codex

# 確認
codex --version
```

Windowsユーザーは公式のPowerShellインストールコマンドを使用できます：`powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"`。

**GitHub Copilot CLI：** 個別のクライアントインストールは不要です。下記の「Skillsのインストール」を参照してください。

### 2. Skillsのインストール

**Claude Code**（macOS / Linux）：

```bash
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
./scripts/install-claude-commands.sh
```

Windows（PowerShell / Command Prompt）：

```bat
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
.\scripts\install-claude-commands.bat
```

これらのskillは頻繁にツールを呼び出すため、Claude Codeはデフォルトで呼び出しごとに承認確認を求めます（クライアントの権限機構によるもので、本リポジトリでは変更できません）。ワークフローを信頼し、信頼できる環境で実行している場合は、承認確認をスキップできます：

```bash
claude --dangerously-skip-permissions
```

注意：このモードはClaude Codeのツール承認保護を無効化します。リポジトリ・コマンド・作業ディレクトリを信頼できる場合のみ使用してください。

**Codex**（macOS / Linux）：

```bash
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
./scripts/install-codex-skills.sh

# オプション：Claude Codeに近い /investment-research 体験を得るためCodexスラッシュプロンプトをインストール
./scripts/install-codex-prompts.sh
```

Windows（PowerShell / Command Prompt）：

```bat
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
.\scripts\install-codex-skills.bat

REM オプション：Codexスラッシュプロンプトをインストール
.\scripts\install-codex-prompts.bat
```

**GitHub Copilot CLI：**

`codex-skills/*/SKILL.md` はディレクトリ名に反して、実際は [skills.sh](https://skills.sh) 標準の汎用skillパッケージ形式です。`npx skills` はCopilot CLIを含む対応agentにこれをインストールします：

```bash
npx skills add sinmentis/ai-berkshire-generic@investment-team -g -y
# 他のskill名に置き換え可（上記Skills一覧参照）、または npx skills find で探索
```

既知の制限：セッション途中でインストールしたskillは、そのCopilot CLIセッション内では即座に認識されません。新しいセッションを開始してください。本リポジトリを直接 `git clone` して、Copilot CLIに `skills/*.md` を読み込ませ、リポジトリ付属の `tools/*.py` を呼び出させる方法もあります——単体のskillパッケージより完全です。詳細は [`COPILOT.md`](COPILOT.md) を参照してください。

**3つのエントリーポイントの関係**：`skills/*.md` はagent中立な標準ワークフローのソースで、Claude Code / Codex / Copilot CLIから直接利用可能；`codex-skills/*/SKILL.md` は `scripts/sync-codex-skills.py` が `skills/*.md` から生成する汎用skillパッケージ；`codex-prompts/*.md` はオプションのCodexスラッシュプロンプト互換レイヤーです。

### 3. コストとモデル選択

深掘りリサーチ系のSkillはデフォルトで複数ラウンドのリサーチ、クロス検証、複数役割による総合判断を行うため、トークン消費は多くなります——これはより完全なビジネス・財務・業界・リスク分析と引き換えです。

護城河・評価・経営陣・リスクのクロス判断が関わる場合、分析の質はモデルの能力により大きく依存します。コスト削減のために重要な判断の質を犠牲にすることは推奨しません。コストを抑えたい場合は、まずワークフローを調整してください：企業の素早い除外には [`/quality-screen`](skills/quality-screen.md)、株価急変動の原因分析には [`/news-pulse`](skills/news-pulse.md) を使い、深掘りする価値があると判断してから [`/investment-research`](skills/investment-research.md) や [`/investment-team`](skills/investment-team.md) を実行してください。

---

## 使い方

Claude Codeで直接呼び出す：

```bash
# 深掘りリサーチ
/investment-research テンセント
/investment-team Meituan
/management-deep-dive ワン・シン Meituan
/private-company-research SpaceX
/deep-company-series PDD

# 決算分析
/earnings-review テンセント 2025Q4
/earnings-team PDD 2025年次報告書

# 業界スクリーニング
/industry-research 原子力発電
/industry-funnel AIコンピューティング
/quality-screen ハンセン指数構成銘柄
/bottleneck-hunter AIインフラ
/investment-checklist Moutai, NVIDIA, Apple

# ポートフォリオ管理
/portfolio-review テンセント30%, Meituan20%, Moutai20%, 現金30%
/thesis-tracker PDD
/thesis-drift PDD reports/pdd-thesis-2025Q4.md reports/pdd-thesis-2026Q1.md
/news-pulse テンセント

# 思考ツール
/dyp-ask PDDの護城河は結局どこにあるのか？
/wechat-article Meituan
```

Codexではインストール後Codexを再起動し、skill名でタスクを説明する：

```text
investment-researchを使ってテンセントをリサーチ
earnings-reviewを使ってPDDの2025年次報告書を分析
industry-funnelを使ってAIコンピューティングをスクリーニング
bottleneck-hunterを使ってAIインフラのボトルネックをスキャン
thesis-driftを使ってPDDの2つの投資論文を比較
wechat-articleを使ってMeituanの投資記事を書く
```

Codexスラッシュプロンプトをインストールした場合、Codexを再起動して`/`メニューから検索します。Codexの公式カスタムプロンプトエントリーポイントは通常 `prompts:<name>` として表示されます：

```text
/prompts:investment-research テンセント
```

GitHub Copilot CLIでは、対応するskillをインストール後、自然言語でタスクを説明するか、skill名で呼び出してください。具体的な規約は [`COPILOT.md`](COPILOT.md) を参照してください。

---

## 実践リサーチレポート

本フレームワークで生成したレポート例：

| 企業 | 使用Skill | 核心的結論 | レポート |
|------|-----------|---------|---------|
| PDD (Pinduoduo) | `/investment-team` | 総合3.4/5、極めて割安だが10年の確実性が不足、中程度のポジションが適切 | [レポートを見る](reports/拼多多/) |
| テンセント (0700.HK) | `/investment-research` | ソーシャル独占+優れた資本配分、14倍の予想PERは合理的にやや低い | [レポートを見る](reports/腾讯/) |
| 7社比較 | `/investment-checklist` | Moutai・テンセントは通過；NVIDIA・Meituan・快手は条件付き通過；PDD・Pop Martはグレーゾーン | [レポートを見る](reports/多公司对比-checklist-20260408.md) |
| 巨人たちの保有銘柄トラッキング | カスタムリサーチ | バフェット/李録/段永平の最新13F保有銘柄＋PDDのコストベース分析 | [レポートを見る](reports/大师持仓追踪-research-20260408.md) |

本フレームワークで生成したレポートのPRを歓迎します。

---

## 今後の方向性

- [ ] ヒストリカルバックテスト：AIリサーチ vs 実際の株価パフォーマンス
- [ ] マクロ経済サイクル分析フレームワーク
- [ ] MCPベースのリアルタイムデータ接続（Wind/Bloomberg/Yahoo Finance）

---

## 免責事項

本プロジェクトは学習・研究目的のみであり、いかなる投資助言も構成しません。投資にはリスクが伴います。必ず自己責任でデューデリジェンス（DYOR）を行ってください。

## License

MIT License
