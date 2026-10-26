---
title: "標準TOMLパーサ（tomllib）"
aliases: [tomllib, TOML標準対応, PEP 680]
category: library
python_version: "3.11"
peps: [680]
created: 2026-10-26
updated: 2026-10-26
tags: [py311, library, toml, config]
---

# 標準TOMLパーサ（tomllib）

## 1. 概要と動機

`pyproject.toml`（PEP 518, PEP 621）がPythonエコシステムの標準設定フォーマットとして定着する中、標準ライブラリにはJSON（`json`）やINI（`configparser`）はあってもTOMLを解釈するモジュールがありませんでした。ビルドツールや設定読み込みのために外部依存（`tomli` 等）が必要でした。

PEP 680 は、高速で安定した `tomli` をベースに、読み込み専用の `tomllib` モジュールを標準ライブラリに追加しました。

---

## 2. 仕様と構文

- **読み込み専用**: `tomllib.loads(s)`（文字列から）および `tomllib.load(f)`（バイナリファイルから）。
- **バイナリモード必須**: `tomllib.load()` は `rb` モードで開かれたバイナリファイルオブジェクトを要求します（UTF-8デコードを高速にCレベルで処理するため）。
- **シリアライズ（dump/dumps）は非提供**: 設定ファイルの書き込みはコメントやフォーマット保持が難しいため標準では提供されず、書き出しが必要な場合は外部の `tomli-w` や `tomlkit` を使用します。

```python
# ファイルからの TOML 読み込み（バイナリモード "rb" が必須）
with open("pyproject.toml", "rb") as f:
    config = tomllib.load(f)
```

---

## 3. 最小コード例

```python
import tomllib

toml_content = """
[project]
name = "demo-vault"
version = "0.1.0"
authors = ["Jun <jun@example.com>"]

[tool.pytest.ini_options]
minversion = "7.0"
testpaths = ["tests"]
"""

data = tomllib.loads(toml_content)
print("Project name:", data["project"]["name"])
print("Version:", data["project"]["version"])
print("Pytest testpaths:", data["tool"]["pytest"]["ini_options"]["testpaths"])
```

### 実行出力
```text
Project name: demo-vault
Version: 0.1.0
Pytest testpaths: ['tests']
```

---

## 4. 実務上の指針と落とし穴

- **Python 3.10以前との互換性**:
  Python 3.10以下をサポートする場合は、以下のようなフォールバックインポートを記述します。
  ```python
  try:
      import tomllib
  except ModuleNotFoundError:
      import tomli as tomllib
  ```

---

## 5. 関連リンク

- [[versions/py311|Python 3.11 概要]]
- [[language/py311/701-exception-groups|701-exception-groups]]

---

## 6. 参考資料

- [PEP 680 – tomllib: Support for Parsing TOML in the Standard Library](https://peps.python.org/pep-0680/)
