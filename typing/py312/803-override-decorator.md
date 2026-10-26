---
title: "@override デコレータによる安全な派生実装（PEP 698）"
aliases: [@override, オーバーライドデコレータ, PEP 698]
category: typing
python_version: "3.12"
peps: [698]
created: 2026-10-26
updated: 2026-10-26
tags: [py312, typing, oop, override, static-analysis]
---

# @override デコレータによる安全な派生実装（PEP 698）

## 1. 概要と動機

オブジェクト指向プログラミングにおいて、親クラスのメソッドを子クラスでオーバーライドする際、親クラス側のメソッド名がリファクタリングで変更されたり、子クラス側のスペルミスがあったりすると、オーバーライド関係がサイレントに壊れてしまう問題がありました。Java や TypeScript には `@Override` や `override` キーワードがありコンパイル時に検知されますが、Pythonにはこれがありませんでした。

PEP 698 は、型チェッカにオーバーライドの意図を明示的に伝える `@typing.override` デコレータを導入しました。

---

## 2. 仕様と構文

- **構文**: `@typing.override` をメソッドに付与。
- **静的解析の検証**:
  基底クラスのいずれかに同名のメソッドや属性が存在しない場合、型チェッカ（mypy / pyright）がエラーを報告します。
- **実行時の挙動**: 実行時には単に元の関数をそのまま返すだけの無害なデコレータです。

```python
# @override による派生メソッド宣言
class Derived(Base):
    @override
    def execute(self) -> None:
        super().execute()
```

---

## 3. 最小コード例

```python
from typing import override

class BaseService:
    def handle_request(self) -> str:
        return "base response"

class CustomService(BaseService):
    @override
    def handle_request(self) -> str:
        return "custom response"

    # もしスペルミスで @override def handl_request と書くと、
    # 型チェッカが「基底クラスに handl_request は存在しない」と警告する

if __name__ == "__main__":
    s = CustomService()
    print("Response:", s.handle_request())
```

### 実行出力
```text
Response: custom response
```

---

## 4. 実務上の指針と落とし穴

- **Python 3.11以前との互換性**:
  `from typing_extensions import override` を使用することで、古いPython環境でも安全に利用できます。
- **実行時の静的検証制限**:
  `@override` は実行時には検証を行わないため、実行時エラーにはなりません。mypy や pyright などの静的型チェッカの実行が必須です。
  ```python
  # 型チェッカ実行時のみエラー検出
  class Child(Base):
      @override
      def misspelt_method(self): ...  # Error: Method does not override base class
  ```

---

## 5. 関連リンク

- [[versions/py312|Python 3.12 概要]]
- [[typing/py38/403-typing-extensions-standardized|403-typing-extensions-standardized]]

---

## 6. 参考資料

- [PEP 698 – Override Decorator for Static Typing](https://peps.python.org/pep-0698/)
- [Python 3.12 What's New: PEP 698](https://docs.python.org/3/whatsnew/3.12.html#pep-698-override-decorator-for-static-typing)
