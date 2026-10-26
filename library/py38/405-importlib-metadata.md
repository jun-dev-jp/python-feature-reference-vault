---
title: "importlib.metadata によるパッケージメタデータとリソースの標準アクセス (PEP 566)"
aliases: [importlib.metadata, パッケージメタデータ取得, PEP 566, importlib.resources]
category: library
python_version: "3.8"
peps: [566]
created: 2026-10-26
updated: 2026-10-26
tags: [py38, library, packaging, importlib, pep566]
---

# importlib.metadata によるパッケージメタデータとリソースの標準アクセス (PEP 566)

---

## 1. 概要と動機

かつてPythonにおいて、インストール済みパッケージのバージョン取得、エントリーポイント（CLIコマンド定義）の探索、あるいはパッケージ内の静的アセットファイルの読み込みには、サードパーティの `pkg_resources`（setuptools付属）が広く使われていました。

しかし `pkg_resources` はインポートが極めて低速であり、実行時オーバーヘッドが大きいという深刻な課題を抱えていました。Python 3.8 で標準ライブラリに導入された **`importlib.metadata`**（および `importlib.resources`）により、軽量かつ標準的なAPIでパッケージ情報へアクセスできるようになり、`pkg_resources` からの完全移行が実現しました。

---

## 2. 仕様と構文

- **バージョン取得**: `importlib.metadata.version("package-name")`
- **メタデータ詳細**: `importlib.metadata.metadata("package-name")` で `pyproject.toml` や `setup.cfg` で定義された概要辞書を取得。
- **エントリーポイント検索**: `importlib.metadata.entry_points()` でプラグインやCLI拡張を動的にディスカバリ。
- **ファイル一覧**: `importlib.metadata.files("package-name")` で配布物に含まれるファイルパス一覧を取得。

```python
from importlib import metadata

# インストール済みパッケージのバージョン取得
try:
    ver = metadata.version("pip")
    print(f"pip version: {ver}")
except metadata.PackageNotFoundError:
    print("Package not installed")
```

---

## 3. 最小コード例

```python
from importlib import metadata

def inspect_package_info():
    # 標準組み込みや仮想環境内のパッケージ（例: pip）を検査
    pkg_name = "pip"
    try:
        ver = metadata.version(pkg_name)
        meta = metadata.metadata(pkg_name)
        print("Package:", pkg_name)
        print("Version:", ver)
        print("Summary:", meta.get("Summary", "N/A"))
        print("Author:", meta.get("Author", "N/A"))
    except metadata.PackageNotFoundError:
        print(f"Package '{pkg_name}' not found.")

if __name__ == "__main__":
    inspect_package_info()
```

### 実行出力
```text
Package: pip
Version: 24.2
Summary: The PyPA recommended tool for installing Python packages.
Author: The pip developers
```

---

## 4. 実務上の指針と落とし穴

- **`pkg_resources` の使用完全停止**:
  `import pkg_resources` は現在非推奨（Deprecated）であり、スクリプトの起動時間を大きく悪化させます。必ず `importlib.metadata` や `importlib.resources` を使用してください。
- **Python 3.8〜3.9 と 3.10+ での `entry_points` API差異**:
  Python 3.10 以降では `entry_points(group="...")` と直接グループ指定可能ですが、Python 3.8/3.9 との互換性を維持する場合は辞書インターフェースまたは `importlib_metadata` バックポートを併用するのが一般的です。

```python
# ✕ 非推奨: 起動が遅く非推奨となった pkg_resources
import pkg_resources
v = pkg_resources.get_distribution("pip").version

# ◯ 推奨: 高速・標準の importlib.metadata
from importlib import metadata
v = metadata.version("pip")
```

---

## 5. 関連リンク

- [[versions/py38|Python 3.8 概要]]
- [[library/py36/205-pathlib-standardization|205-pathlib-standardization]]
- [[ROADMAP|全体ロードマップ]]

---

## 6. 参考資料

- [PEP 566 – Data Structure for the Metadata of Python Software Packages](https://peps.python.org/pep-0566/)
- [Python 公式ドキュメント: importlib.metadata — パッケージメタデータへのアクセス](https://docs.python.org/ja/3/library/importlib.metadata.html)
