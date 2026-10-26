---
title: "暗号論的に安全な乱数生成（secretsモジュール: PEP 506）"
aliases: [secretsモジュール, 安全な乱数, PEP 506]
category: library
python_version: "3.6"
peps: [506]
created: 2026-10-26
updated: 2026-10-26
tags: [py36, library, security, cryptography]
---

# 暗号論的に安全な乱数生成（secretsモジュール: PEP 506）

## 1. 概要と動機

多くの開発者が、パスワードリセットトークン、セッションID、暗号キーの生成に誤って標準の `random` モジュールを使用していました。しかし、`random` モジュールはメルセンヌ・ツイスタ（Mersenne Twister）アルゴリズムを採用しており、統計的シミュレーション用であって暗号論的に安全ではありません（過去の出力を観測することで内部状態を完全に予測可能）。

PEP 506 は、OSが提供する最も信頼できる乱数源（`/dev/urandom` や `CryptGenRandom`）に直接アクセスし、セキュリティ上センシティブな用途に特化した `secrets` モジュールを導入しました。

---

## 2. 仕様と構文

- **主な関数**:
  - `secrets.token_bytes(nbytes=None)`: ランダムなバイト列を生成。
  - `secrets.token_hex(nbytes=None)`: 16進数文字列を生成。
  - `secrets.token_urlsafe(nbytes=None)`: URLセーフなBase64エンコード文字列を生成。
  - `secrets.choice(sequence)`: シーケンスから暗号論的に安全に要素を1つ選択。
  - `secrets.compare_digest(a, b)`: タイミング攻撃を防ぐ固定時間比較（Constant-time comparison）。

```python
# 安全なトークン生成と検証の基本形
token = secrets.token_urlsafe(32)   # 32バイトエントロピーのURL安全文字列
is_valid = secrets.compare_digest(token, input_token)  # タイミング攻撃防御
```

---

## 3. 最小コード例

```python
import secrets

# パスワードリセット用トークン（URLに安全に埋め込み可能）
reset_token = secrets.token_urlsafe(32)
print("URL-safe token:", reset_token)

# 16進数APIキー
api_key = secrets.token_hex(16)
print("API Key:", api_key)

# タイミング攻撃に強いトークン照合
is_valid = secrets.compare_digest(reset_token, reset_token)
print("Token matches securely:", is_valid)
```

### 実行出力（トークンはランダム）
```text
URL-safe token: 5sK1N0p... (43文字程度のBase64文字列)
API Key: a1b2c3d4e5f6... (32文字の16進数文字列)
Token matches securely: True
```

---

## 4. 実務上の指針と落とし穴

- **文字列比較に `==` を使わない**:
  認証トークンや署名の検証に通常の `token == input_token` を使用すると、文字ごとの不一致時に早期リターンする挙動を悪用した「タイミング攻撃（Timing Attack）」に対して脆弱になります。必ず `secrets.compare_digest()` を使用してください。
  ```python
  # 危険: 文字列の長さや先頭一致によるタイミング攻撃のリスク
  # is_ok = (user_token == secret_token)

  # 安全: 固定時間比較を行う
  is_ok = secrets.compare_digest(user_token, secret_token)
  ```
- **ゲームやシミュレーションでの過剰使用**:
  `secrets` はOSのシステムコールを伴うため、大量の乱数を要求するモンテカルロ法やゲームループでは `random` や NumPy の方が高速で適しています。

---

## 5. 関連リンク

- [[versions/py36|Python 3.6 概要]]
- [[ROADMAP|全体ロードマップ]]

---

## 6. 参考資料

- [PEP 506 – Adding A Secrets Module To The Standard Library](https://peps.python.org/pep-0506/)
- [Python 公式ドキュメント: secrets](https://docs.python.org/3/library/secrets.html)
