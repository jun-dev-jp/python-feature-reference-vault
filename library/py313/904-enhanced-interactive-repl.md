---
title: "対話型REPL環境の刷新（PyRepl統合・カラー表示）"
aliases: [新REPL, PyRepl, カラーREPL, 3.13 REPL]
category: library
python_version: "3.13"
peps: []
created: 2026-10-26
updated: 2026-10-26
tags: [py313, library, repl, cli, tooling]
---

# 対話型REPL環境の刷新（PyRepl統合・カラー表示）

## 1. 概要と動機

CPython標準の対話型シェル（REPL）は、数十年にわたり基本的な行単位のreadlineベースで動作しており、複数行ブロックの編集（上矢印キーで複数行関数に戻る等）ができない、構文ハイライトがない、履歴検索が貧弱であるなど、IPython等のサードパーティツールに比べて開発体験で見劣りしていました。

Python 3.13 では、PyPyプロジェクトから移植された純Python実装の高度端末エディタ **PyRepl** が標準統合され、追加の依存関係なしに高機能な対話環境がデフォルト化されました。

---

## 2. 仕様と構文

- **複数行編集**: カーソルを上下キーで複数行の関数やブロック内を自由に行き来して編集可能。
- **カラー表示**: トレースバック、プロンプト、構文エラー箇所が標準でカラー表示される。
- **F1〜F3 ヘルプ・コマンド**:
  - `F1`: コマンド履歴閲覧
  - `F2`: 貼り付けモード（インデント崩れ防止）
  - `F3`: コマンド説明
- **無効化**: 環境変数 `PYTHON_BASIC_REPL=1` を設定することで従来のクラシックREPLに戻すことも可能。

```bash
# 対話シェルの起動とクラシックモードの切り替え
python               # 新しいカラー・複数行 PyRepl で起動
PYTHON_BASIC_REPL=1 python  # 従来の readline 版で起動
```

---

## 3. 最小コード例

REPL起動時のカラー対応と対話機能の確認例です。

```python
import sys

def repl_info():
    print("Python Interactive Shell Version:", sys.version.split()[0])
    # 対話環境の起動はターミナルから `python` を実行
    print("Supports multiline editing, syntax colors, and interactive paste mode!")

if __name__ == "__main__":
    repl_info()
```

### 実行出力
```text
Python Interactive Shell Version: 3.13.0
Supports multiline editing, syntax colors, and interactive paste mode!
```

---

## 4. 実務上の指針と落とし穴

- **スクリプト実行時への影響**:
  この変更はターミナルで対話型シェルを起動した場合にのみ有効であり、ファイル実行（`python script.py`）や標準入出力のリダイレクト時には影響しません。
- **ペースト時のインデント崩れ対策**:
  複数行のコードをターミナルに貼り付ける際は、F2キーで「ペーストモード」に切り替えることで自動インデントによるコード破損を防ぐことができます。
  ```text
  # REPL操作
  (paste mode) >>> def example():
  ...                  return 1
  ```

---

## 5. 関連リンク

- [[versions/py313|Python 3.13 概要]]
- [[language/py313/901-free-threaded-cpython|901-free-threaded-cpython]]

---

## 6. 参考資料

- [Python 3.13 What's New: Improved Interactive Help and REPL](https://docs.python.org/3/whatsnew/3.13.html#a-better-interactive-interpreter)
