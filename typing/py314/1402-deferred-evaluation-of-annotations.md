---
title: "型注釈のオンデマンド遅延評価（PEP 649 / PEP 749）"
aliases: [遅延評価アノテーション, PEP 649, PEP 749, annotation scopes]
category: typing
python_version: "3.14"
peps: [649, 749]
created: 2026-10-26
updated: 2026-10-26
tags: [py314, typing, runtime, annotations, metaprogramming]
---

# 型注釈のオンデマンド遅延評価（PEP 649 / PEP 749）

## 1. 概要と動機

Python 3.7の PEP 563（`from __future__ import annotations`）は、型注釈を文字列として保持することでインポート時のパフォーマンス向上と前方参照の解決を図りました。しかし、この方式には以下の深刻な副作用がありました。
- **実行時型検査ライブラリの破壊**: Pydantic、FastAPI、Cattrs などの実行時に `get_type_hints()` を呼ぶライブラリにおいて、文字列化された型を `eval()` で再評価する必要があり、名前空間の不整合や実行時オーバーヘッドが生じていた。

PEP 649 および PEP 749 は、この長年の論争に終止符を打つ解決策として採択されました。型注釈を「単なる文字列」にするのではなく、「**型注釈を評価するコードを関数/記述子としてコンパイルし、必要になった初回アクセス時にのみオンデマンド評価（Lazy evaluation）する**」アーキテクチャを導入しました。

---

## 2. 仕様と構文

> [!NOTE] バージョンステータス: 採択・実装中（Accepted / Implementation for 3.14）

- **`__annotate__` 関数の自動生成**:
  関数やクラスが定義された際、型注釈を生成する特殊関数 `__annotate__(format)` が内部的に作成される。
- **アクセス時評価**:
  `obj.__annotations__` にアクセスされた時点で `__annotate__` が呼び出され、キャッシュされる。
- **フォーマット指定（PEP 749）**:
  - `FORMAT_VALUE (1)`: 通常の評価値（型オブジェクトそのもの）
  - `FORMAT_FORWARDREF (2)`: 未定義名があってもエラーにせず `ForwardRef` として返す
  - `FORMAT_SOURCE (3)`: ソースコード上の文字列表現として取得

```python
# 型注釈のオンデマンド評価モデル
def fetch_item(id: int) -> ResultItem: ...
# 定義時点では ResultItem は評価されず、型検査時に初めて解決される
```

---

## 3. 最小コード例

Python 3.14 での遅延評価の挙動モデルと、未定義の前方参照がエラーにならず定義後に自動解決される様子の概念例です。

```python
# Python 3.14 アノテーション遅延評価モデル
class Container:
    # 定義時点で未定義の 'Item' をクォートなし・__future__なしで記述可能
    def add(self, item: "Item") -> None:
        pass

# 定義後にクラスが出現
class Item:
    pass

if __name__ == "__main__":
    import typing
    # アノテーションにアクセスした瞬間に初めて評価される
    hints = typing.get_type_hints(Container.add)
    print("Resolved hints:", hints)
    print("Is Item class:", hints['item'] is Item)
```

### 実行出力
```text
Resolved hints: {'item': <class '__main__.Item'>, 'return': <class 'NoneType'>}
Is Item class: True
```

---

## 4. 実務上の指針と落とし穴

- **`from __future__ import annotations` からの移行**:
  PEP 563 は将来のバージョンでデフォルト化される予定でしたが、PEP 649 の採択により撤回されました。既存の `from __future__ import annotations` を記述したコードも後方互換性のためにサポートされますが、新規開発では PEP 649 / 749 のネイティブ遅延評価に委ねることが推奨されます。
- **インポート速度の向上**:
  クラスや関数を多数ロードする大規模Webフレームワーク（FastAPIやDjangoなど）において、起動時に使われない型注釈の評価コストがゼロになり、起動時間が短縮されます。
- **実行時のアクセスAPI**:
  `__annotations__` 属性を直接参照するのではなく、常に `annotationlib.get_annotations()` または `typing.get_type_hints()` を経由して取得してください。
  ```python
  import annotationlib
  # 推奨される型注釈取得方法
  hints = annotationlib.get_annotations(func, format=annotationlib.Format.VALUE)
  ```

---

## 5. 関連リンク

- [[versions/py314|Python 3.14 概要]]
- [[typing/py37/302-postponed-annotations|302-postponed-annotations]]
- [[language/py312/801-type-parameter-syntax|801-type-parameter-syntax]]

---

## 6. 参考資料

- [PEP 649 – Deferred Evaluation Of Annotations Using Descriptors](https://peps.python.org/pep-0649/)
- [PEP 749 – Implementing Deferred Evaluation of Annotations](https://peps.python.org/pep-0749/)
- [Python 3.14 What's New: Deferred Evaluation of Annotations](https://docs.python.org/3.14/whatsnew/3.14.html)
