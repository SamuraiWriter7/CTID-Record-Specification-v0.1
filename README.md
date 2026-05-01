# CTID Record Specification v0.1  
**Communication Trace ID (CTID) for Communication Royalty OS**

CTID は、意味生成を伴う通信行為を一意に識別し、  
その起源・因果・文脈・寄与接続性を追跡可能にするための通信痕跡識別仕様です。

本リポジトリは、通信印税OSにおける痕跡層の最小カーネルとして、  
`ctid-record-v0.1` の仕様・Schema・サンプル・検証フローを提供します。

---

## Overview

通信印税OSでは、通信を単なる情報伝達としてではなく、  
**参照・要約・応答・再利用・影響** を含む **意味生成行為** として扱います。

CTID（Communication Trace ID）は、そのような意味生成通信に対して発行される  
**構造的な痕跡識別子** です。

CTIDの目的は、通信によって生じた意味の流れを、

- 記録し
- 追跡し
- 可視化し
- 後続の寄与解析や分配へ接続する

ことにあります。

これは通信量課金のためのIDではありません。  
CTIDは、**意味痕跡ID** です。

---

## One-line Definition

> **CTID is a trace identifier for meaning-generating communication events, designed to preserve origin, causality, context, and contribution linkability.**

---

## Why CTID?

通常のメッセージIDは、「送受信があった」という事実は示せても、  
その通信が

- 何を参照したのか
- どの応答から生まれたのか
- どのような文脈で意味を持ったのか
- 後続の価値循環にどう接続されるのか

までは保持しません。

CTID は、この欠落を埋めるための仕様です。

---

## Core Principles

CTID は、通信印税OSの3原則に従います。

### 1. Semantic Generation Principle
通信は、単なる情報伝達ではなく意味生成行為である。

### 2. Trace Visibility Principle
意味生成を生んだ通信痕跡は、透明かつ構造的に記録・可視化されなければならない。

### 3. Contribution Circulation Principle
通信痕跡に基づく寄与は、価値として循環されなければならない。

---

## Scope

CTID は、以下のような **意味生成通信** に適用されます。

- 人間 ↔ AI の対話
- AI ↔ AI のエージェント通信
- 要約・応答・参照・再利用
- ワークフロー内の因果的メッセージ連鎖
- 派生・変形・影響の記録

### Out of Scope

CTID は、以下を直接の対象とはしません。

- 通信量課金
- 低レイヤ通信制御
- 単なる接続維持パケット
- 意味差分のない再送
- 暗号通貨そのものの実装
- 全文保存の強制

---

## Repository Structure

```text
.
├─ ctid-record-v0.1.yaml
├─ schemas/
│  └─ ctid-record-v0.1.schema.json
├─ examples/
│  └─ ctid-record.sample.json
└─ .github/
   └─ workflows/
      └─ validate-specs.yml
Start Here

最初に読む順番は、以下を推奨します。

ctid-record-v0.1.yaml
人間可読な仕様本文
schemas/ctid-record-v0.1.schema.json
機械可読な検証用Schema
examples/ctid-record.sample.json
実際にSchemaへ通る最小サンプル
.github/workflows/validate-specs.yml
GitHub Actions による継続的検証フロー
CTID Format

CTID は、以下の URI風フォーマットを推奨します。

ctid:v0.1:<issuer_namespace>:<unique_id>
Example
ctid:v0.1:kazene:01966d7e-8c5c-7b41-a9d0-4b8d5f3b11aa
Components
ctid
固定プレフィックス
v0.1
仕様バージョン
issuer_namespace
発行主体またはドメイン
unique_id
一意識別子（UUIDv7推奨）
CTID Record Model

ctid-record-v0.1 は、単一の意味生成通信イベントを記述するレコードです。

Required Fields
schema_version
ctid
root_ctid
parent_ctids
event_type
timestamp
actor
target
semantic_digest
permissions
status
Optional Fields
thread_id
source_refs
context_envelope
integrity
contribution_hints
Event Types

v0.1 では、以下の event_type を標準値として定義します。

reference
summarize
respond
reuse
transform
relay
invoke
evaluate
Minimal Example
{
  "schema_version": "ctid-record-v0.1",
  "ctid": "ctid:v0.1:kazene:01966d7e-8c5c-7b41-a9d0-4b8d5f3b11aa",
  "root_ctid": "ctid:v0.1:kazene:01966d7e-8c5c-7b41-a9d0-4b8d5f3b11aa",
  "parent_ctids": [],
  "event_type": "respond",
  "timestamp": "2026-05-01T06:30:00Z",
  "actor": {
    "actor_id": "agent:masamune",
    "actor_type": "ai_agent"
  },
  "target": {
    "actor_id": "human:kazene",
    "actor_type": "human"
  },
  "semantic_digest": {
    "summary": "通信印税OSの定義文に応答し、CTID仕様化への橋渡しを行った",
    "content_hash": "sha256:abc123def456abc123def456abc123def456abc123def456abc123def456abcd"
  },
  "permissions": {
    "trace_allowed": true,
    "summarize_allowed": true,
    "reuse_allowed": false,
    "attribution_mode": "required"
  },
  "status": "observed"
}

詳細な完全例は examples/ctid-record.sample.json を参照してください。

Schema Usage
Local Validation

ローカルで Schema 検証を行うには、まず依存関係をインストールします。

python -m pip install --upgrade pip
pip install jsonschema PyYAML

次に、以下のコマンドで examples/ctid-record.sample.json を検証できます。

python - <<'PY'
import json
from pathlib import Path
from jsonschema import Draft202012Validator

schema = json.loads(Path("schemas/ctid-record-v0.1.schema.json").read_text(encoding="utf-8"))
sample = json.loads(Path("examples/ctid-record.sample.json").read_text(encoding="utf-8"))

Draft202012Validator.check_schema(schema)
Draft202012Validator(schema).validate(sample)

print("OK: CTID Record v0.1 sample is valid.")
PY
GitHub Actions Validation

CI では .github/workflows/validate-specs.yml により、
Schema と sample の整合性を自動検証します。

これにより、仕様変更時にサンプルやSchemaの破綻を早期に検出できます。

Design Notes
1. CTID is not a billing ID

CTID は通信量課金のためのIDではありません。
意味生成の痕跡を扱うためのIDです。

2. CTID is not a full transcript mandate

CTID は全文保存を必須としません。
要約＋ハッシュのみでも成立可能です。

3. CTID is a traceability kernel

CTID 自体は報酬額を決めません。
CTID は、寄与解析・可視化・分配へ接続するための基礎層です。

Relationship to Communication Royalty OS

CTID は、通信印税OSにおける痕跡層の最小核です。
通信印税OS全体では、以下のような流れを想定しています。

通信 → 痕跡 → 寄与解析 → 可視化 → 分配 → 還流 → 新通信

このうち CTID が担うのは、主に 通信 → 痕跡 の変換部分です。

Versioning

本リポジトリは現在、v0.1 の初期仕様を扱っています。
v0.1 は最小実装を重視したカーネル版です。

将来バージョンでは、以下のような拡張が想定されます。

issued_by
observed_by
privacy_level
confidence_score
settlement_ref
reward_ref
dispute_ref
license_ref
Status

Draft / Experimental

この仕様は、通信印税OSにおける痕跡識別の最小核を定義するためのドラフトです。
制度設計・可視化・寄与計算・分配モデルへの接続を前提としていますが、
現時点ではまだ拡張中の実験仕様です。

License

ライセンス方針は、この仕様の今後の公開方針に応じて別途定義してください。
現段階では、リポジトリの公開目的に応じて以下のいずれかが候補になります。

オープンな仕様公開を重視する場合: CC BY 4.0
コア思想の保護を重視する場合: 独自ライセンス
コード断片中心で運用する場合: MIT
Future Work

次のステップとしては、以下が自然です。

contribution-model-v0.1 の定義
ctid-graph-v0.1 の定義
通信寄与計算式（深度×拡散×純度）の仕様化
ARA-01通信版への接続
Qコイン／信用／感謝の分配設計
Short Definition for Citation

CTID is a structural trace identifier for meaning-generating communication.
