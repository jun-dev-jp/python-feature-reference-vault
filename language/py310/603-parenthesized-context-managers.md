---
title: "括弧付きコンテキストマネージャ（複数行構文）"
aliases: [括弧付きwith, 複数行with構文, Parenthesized Context Managers]
category: language
python_version: "3.10"
peps: []
created: 2026-10-26
updated: 2026-10-26
tags: [py310, language, syntax, contextmanager]
---

# 括弧付きコンテキストマネージャ（複数行構文）

## 1. 概要と動機

Python で複数のコンテキストマネージャを同時に開く際（例: 2つのファイルを同時に開いて読み書きする）、行が長くなった場合に改行しようとすると、文法上 `with` の後ろに丸括弧を置くことができず、行末にバックスラッシュ `\` を置いて継続行にする必要がありました。バックスラッシュによる継続行はミスが起きやすく（末尾にスペースが入ると構文エラー）、Pythonの「括弧による自然な改行」という理念に反していました。

Python 3.9 で導入された新PEGパーサ（PEP 617）の表現力向上を活かし、Python 3.10 では `with` 文の後に丸括弧を配置して複数行にわたるコンテキストマネージャを自然に記述できるようになりました。

---

## 2. 仕様と構文

- **構文**:
  ```python
  with (
      open("input.txt", "r") as f_in,
      open("output.txt", "w") as f_out,
  ):
      f_out.write(f_in.read())
  ```
- 末尾のカンマ（Trailing Comma）も許可されます。

---

## 3. 従来記法との比較

Python 3.9 以前の複数コンテキストマネージャ改行記法と、Python 3.10 の丸括弧付き `with` 構文の比較です。

```python
import io

buf1 = io.StringIO("Data 1")
buf2 = io.StringIO("Data 2")

# --- 従来（Python 3.9以前）: バックスラッシュ行継続または横に長い1行 ---
# 行末にスペースが混入すると SyntaxError になる脆弱性やスタイル崩れの温床だった
with buf1 as f1, \
     buf2 as f2:
    legacy_res = f1.read() + f2.read()

buf1.seek(0)
buf2.seek(0)

# --- Python 3.10+: 丸括弧による自然な複数行コンテキストマネージャ ---
# 末尾カンマを許容し、PEP 8 の「丸括弧内での自然な改行」スタイルに適合
with (
    buf1 as f1,
    buf2 as f2,
):
    modern_res = f1.read() + f2.read()

print("Modern res:", modern_res)
```

---

## 4. 最小コード例

```python
import io

buffer_a = io.StringIO("Data from A")
buffer_b = io.StringIO("Data from B")

# 丸括弧を用いて複数行に整形されたクリーンなwith文
with (
    buffer_a as in_a,
    buffer_b as in_b,
):
    merged = f"{in_a.read()} & {in_b.read()}"

print("Merged result:", merged)
```

### 実行出力
```text
Merged result: Data from A & Data from B
```

---

## 5. 実務上の指針と落とし穴

- **Python 3.9以前との互換性**:
  Python 3.9 ではPEGパーサが採用されていましたが、この構文自体は公式には未解禁または不完全でした。Python 3.9以前をサポートするコードベースでは、`contextlib.ExitStack` を用いるか従来の1行記述が必要です。
- **バックスラッシュ継続行の排除**:
  本構文の導入により、可読性を損ねていたバックスラッシュ `\` による継続行を完全に排除できます。
  ```python
  # レガシーなアンチパターン（バックスラッシュ行継続）
  # with open("a.txt") as f1, \
  #      open("b.txt") as f2: ...

  # 推奨パターン（丸括弧で囲み末尾カンマを許容）
  with (
      open("a.txt") as f1,
      open("b.txt") as f2,
  ):
      pass
  ```

---

## 6. 関連リンク

- [[versions/py310|Python 3.10 概要]]
- [[language/py310/601-pattern-matching|601-pattern-matching]]

---

## 7. 参考資料

- [Python 3.10 What's New: Parenthesized context managers](https://docs.python.org/3/whatsnew/3.10.html#parenthesized-context-managers-are-now-supported)
