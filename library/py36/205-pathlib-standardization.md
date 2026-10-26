---
title: "pathlib の実用標準化とファイルシステムパスプロトコル (PEP 519)"
aliases: [pathlib標準化, PEP 519, os.PathLike, pathlib]
category: library
python_version: "3.6"
peps: [519]
created: 2026-10-26
updated: 2026-10-26
tags: [py36, library, filesystem, pathlib, pep519]
---

# pathlib の実用標準化とファイルシステムパスプロトコル (PEP 519)

---

## 1. 概要と動機

Python 3.4 で導入された `pathlib` モジュールは、ファイルパスを文字列ではなくオブジェクトとして直感的に扱う強力な仕組みでしたが、Python 3.5 までは標準ライブラリの多くの組み込み関数（`open()`、`os.path.join()`、`os.remove()` など）が `str` や `bytes` しか受け付けず、毎回 `str(path)` で文字列化する必要がありました。

Python 3.6 の **PEP 519** では、オブジェクトがパス表現であることを示すプロトコル **`os.PathLike`** と特殊メソッド **`__fspath__()`** が制定されました。これにより、組み込み `open()` を含む標準ライブラリ全体が `Path` オブジェクトを直接受け取れるようになり、モダンPythonにおける標準パス操作手法として完全に定着しました。

---

## 2. 仕様と構文

- **ファイルシステムパスプロトコル**:
  クラスに `__fspath__()` を実装すると `os.PathLike` プロトコルに準拠し、ファイルシステムパスとして扱えます。
- **標準関数の直接サポート**:
  `open()`, `os.listdir()`, `shutil.copy()` などが `os.PathLike` を直接引数として受理。
- **パス連結演算子**:
  `/` 演算子によるプラットフォーム非依存の直感的なパス結合。

```python
from pathlib import Path

# / 演算子でパスを連結（OS差異を自動吸収）
base_dir = Path("/var/log")
log_file = base_dir / "app" / "debug.log"

# open() に Path オブジェクトを直接渡して読み書き可能（Python 3.6+）
with open(log_file, "w", encoding="utf-8") as f:
    f.write("System initialized\n")
```

---

## 3. 最小コード例

```python
from pathlib import Path

def demonstrate_pathlib():
    # カレントディレクトリ配下に作業用パスを定義
    data_dir = Path("./sample_workspace")
    target_file = data_dir / "config.json"

    # ディレクトリ作成（親ディレクトリも自動作成、既存でもOK）
    data_dir.mkdir(parents=True, exist_ok=True)

    # 便利なテキスト書き込み・読み込みヘルパー
    target_file.write_text('{"status": "ok"}', encoding="utf-8")
    content = target_file.read_text(encoding="utf-8")
    print("File Content:", content)

    # パス情報の解析
    print("Filename:", target_file.name)
    print("Suffix:", target_file.suffix)
    print("Parent:", target_file.parent)

    # クリーンアップ
    target_file.unlink()
    data_dir.rmdir()

if __name__ == "__main__":
    demonstrate_pathlib()
```

### 実行出力
```text
File Content: {"status": "ok"}
Filename: config.json
Suffix: .json
Parent: sample_workspace
```

---

## 4. 実務上の指針と落とし穴

- **文字列連結によるパス操作の廃止**:
  文字列の `+` や `f"{dir}/{file}"` でパスを構築すると、Windows（`\`）とPOSIX（`/`）の差異で不具合の温床になります。常に `Path` と `/` 演算子を使用します。
- **`.read_text()` / `.write_text()` の encoding 指定**:
  `Path.read_text()` や `Path.write_text()` はデフォルトでプラットフォーム依存のエンコーディングを使用するため、必ず `encoding="utf-8"` を明記してください。

```python
from pathlib import Path

# ✕ アンチパターン: 文字列連結や区切り文字のハードコード
log_path = "/var/log" + "/" + "app.log"  # Windowsで動作が不安定になる

# ◯ ベストプラクティス: Path オブジェクトの利用と encoding の明示
log_path = Path("/var/log") / "app.log"
log_path.write_text("Hello", encoding="utf-8")  # 文字化け防止
```

---

## 5. 関連リンク

- [[versions/py36|Python 3.6 概要]]
- [[library/py35/104-subprocess-run|104-subprocess-run]]
- [[ROADMAP|全体ロードマップ]]

---

## 6. 参考資料

- [PEP 519 – Adding a file system path protocol](https://peps.python.org/pep-0519/)
- [Python 公式ドキュメント: pathlib — オブジェクト指向のファイルシステムパス](https://docs.python.org/ja/3/library/pathlib.html)
