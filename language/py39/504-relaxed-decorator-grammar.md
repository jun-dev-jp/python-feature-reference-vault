---
title: "緩和されたデコレータ文法（PEP 614）"
aliases: [デコレータ文法緩和, PEP 614, relaxed decorator syntax]
category: language
python_version: "3.9"
peps: [614]
created: 2026-10-26
updated: 2026-10-26
tags: [py39, language, syntax, decorators]
---

# 緩和されたデコレータ文法（PEP 614）

## 1. 概要と動機

Python 2.4 で導入されたデコレータ文法は、`@dotted_name` または `@dotted_name(args)` という限定的な構文規則（文法上の制限）に従っていました。そのため、辞書に登録されたデコレータ関数（`@buttons["click"]`）や、複雑な属性チェーン、関数呼び出しの結果から属性を取り出すような有効なPython式を `@` の直後に記述できず、一時変数に一度代入する必要がありました。

PEP 614 はこのパーサの制限を撤廃し、任意の有効な代入式相当の式（`test` 文法規則）をデコレータ式として記述できるようにしました。

---

## 2. 仕様と構文

- **構文**: `@expr`（任意の式を指定可能）
- **利用可能になった例**:
  - 辞書参照: `@registry[action_type]`
  - 条件式: `@(decorator_a if condition else decorator_b)`
  - メソッドチェーン: `@builder.add_hook().register`

```python
# 任意の式を直接デコレータとして記述
@decorators_dict["auth_required"]
def secured_endpoint():
    pass
```

---

## 3. 最小コード例

```python
registry = {}

def register(name: str):
    def decorator(func):
        registry[name] = func
        return func
    return decorator

handlers = {
    "login": register("login_handler"),
    "logout": register("logout_handler"),
}

# 辞書アクセス式をデコレータとして直接利用（PEP 614）
@handlers["login"]
def handle_user_login():
    return "User authenticated"

if __name__ == "__main__":
    print("Registered handlers:", registry)
    print("Result:", registry["login_handler"]())
```

### 実行出力
```text
Registered handlers: {'login_handler': <function handle_user_login at 0x...>}
Result: User authenticated
```

---

## 4. 実務上の指針と落とし穴

- **過度な複雑化の抑制**:
  構文上は任意の式が書けるようになりましたが、三項演算子や複雑なインライン計算を `@` 直下に詰め込みすぎると可読性が著しく低下します。GUIツールキットのボタンハンドラ登録や、辞書ディスパッチャへの登録などの直感的なユースケースに留めるのが好ましいです。
- **丸括弧の活用**:
  条件式など演算子の優先順位が複雑な式をデコレータにする場合は、`@(dec_a if flag else dec_b)` のように明示的に括弧で囲むことで文法解釈エラーを防ぎます。
  ```python
  # 三項演算子をデコレータに適用する場合は括弧で囲む
  @(audit_log if IS_PRODUCTION else dry_run)
  def sensitive_operation():
      pass
  ```

---

## 5. 関連リンク

- [[versions/py39|Python 3.9 概要]]
- [[language/py310/601-pattern-matching|601-pattern-matching]]

---

## 6. 参考資料

- [PEP 614 – Relaxing Grammar Restrictions On Decorators](https://peps.python.org/pep-0614/)
- [Python 3.9 What's New: PEP 614](https://docs.python.org/3/whatsnew/3.9.html#pep-614-relaxed-grammar-restrictions-on-decorators)
