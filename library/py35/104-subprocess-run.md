---
title: "サブプロセス実行の高水準API（subprocess.run）"
aliases: [subprocess.run, サブプロセス実行, subprocess]
category: library
python_version: "3.5"
peps: []
created: 2026-10-26
updated: 2026-10-26
tags: [py35, library, os, process]
---

# サブプロセス実行の高水準API（subprocess.run）

## 1. 概要と動機

Python 3.4 以前の `subprocess` モジュールには、`subprocess.call()`, `subprocess.check_call()`, `subprocess.check_output()`、および低水準な `subprocess.Popen` など複数のAPIが乱立していました。外部コマンドを実行して終了を待ち、標準出力・標準エラー出力をキャプチャし、エラーハンドリングを行うという日常的なタスクに対して、どの関数を使うべきか混乱が生じていました。

Python 3.5 では、これらすべてのユースケースを統合・洗練した標準の推奨エントリーポイントとして `subprocess.run()` が導入されました。

---

## 2. 仕様と構文

- **シグネチャ**: `subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, capture_output=False, shell=False, cwd=None, timeout=None, check=False, encoding=None, errors=None, text=None, env=None)`
- **戻り値**: `CompletedProcess` インスタンス
  - `args`: 実行されたコマンド引数
  - `returncode`: 終了ステータスコード
  - `stdout`, `stderr`: 出力結果
  - `check_returncode()`: 終了コードが0以外のときに `CalledProcessError` を発生
- **`check=True`**: コマンドが失敗した場合に即座に例外を発生させる。

```python
# subprocess.run の典型的な呼び出しパターン
proc = subprocess.run(
    ["git", "status", "--short"],  # コマンドと引数のリスト
    capture_output=True,           # stdout と stderr を両方キャプチャ
    text=True,                     # 出力を bytes ではなく str で取得
    check=True                     # エラー時に CalledProcessError を送出
)
```

---

## 3. 従来APIとの比較

Python 3.4 以前で出力をキャプチャしエラー検証を行う手順と、Python 3.5 の `subprocess.run()` の比較です。

```python
import subprocess
import sys

cmd = [sys.executable, "-c", "print('Command executed')"]

# --- 従来（Python 3.4以前）: check_output や Popen の混在 ---
# 出力取得には check_output()、終了コードのみには check_call() と使い分けが必要で、
# 標準エラー出力の結合やタイムアウト制御には Popen の明示的ハンドリングが求められた。
try:
    legacy_output = subprocess.check_output(cmd, stderr=subprocess.STDOUT)
    print("Legacy stdout (bytes):", legacy_output)
except subprocess.CalledProcessError as e:
    print("Failed with code:", e.returncode)

# --- Python 3.5+: subprocess.run() ---
# capture_output, text, check などのフラグを統合した単一の推奨インターフェース
result = subprocess.run(cmd, capture_output=True, text=True, check=True)
print("Modern stdout (str):", result.stdout.strip())
print("Modern returncode:", result.returncode)
```

---

## 4. 最小コード例

```python
import subprocess
import sys

# Pythonコマンドを実行して出力をキャプチャする例
result = subprocess.run(
    [sys.executable, "-c", "print('Hello from subprocess!'); import sys; sys.exit(0)"],
    capture_output=True,
    text=True,
    check=True
)

print("Return code:", result.returncode)
print("Stdout:", result.stdout.strip())
```

### 実行出力
```text
Return code: 0
Stdout: Hello from subprocess!
```

---

## 5. 実務上の指針と落とし穴

- **`shell=True` のセキュリティリスク**:
  信頼できないユーザー入力をコマンド文字列に含めて `shell=True` を指定すると、重大なコマンドインジェクション脆弱性が発生します。原則として引数はリスト形式 `['cmd', 'arg1', ...]` で渡し、`shell=False`（デフォルト）で使用してください。
  ```python
  # 危険: シェルインジェクションの脆弱性
  # subprocess.run(f"cat {user_filename}", shell=True)

  # 安全: 引数リスト形式で渡す
  subprocess.run(["cat", user_filename], check=True)
  ```
- **パイプライン処理の注意**:
  `cmd1 | cmd2` のように複数のプロセスをパイプで繋ぐ高度な並行処理を行う場合は、依然として低水準な `subprocess.Popen` を直接利用するか、シェル経由ではなく個別のパイプ設定が必要です。

---

## 6. 関連リンク

- [[versions/py35|Python 3.5 概要]]
- [[ROADMAP|全体ロードマップ]]

---

## 7. 参考資料

- [Python 3.5 What's New: subprocess.run()](https://docs.python.org/3/whatsnew/3.5.html#subprocess)
- [Python 公式ドキュメント: subprocess](https://docs.python.org/3/library/subprocess.html)
