---
title: "標準IANAタイムゾーン（zoneinfoモジュール: PEP 615）"
aliases: [zoneinfo, タイムゾーン, PEP 615, IANAタイムゾーン]
category: library
python_version: "3.9"
peps: [615]
created: 2026-10-26
updated: 2026-10-26
tags: [py39, library, datetime, timezone]
---

# 標準IANAタイムゾーン（zoneinfoモジュール: PEP 615）

## 1. 概要と動機

Python標準ライブラリの `datetime` は長年、固定オフセット（`timezone.utc` 等）しか持たず、夏時間（DST）や各国の歴史的タイムゾーン規則を扱うにはサードパーティの `pytz` や `dateutil` に依存していました。特に `pytz` は `tz.localize(dt)` という非標準のAPIを要求し、通常の `datetime(..., tzinfo=tz)` と併用するとオフセット計算が壊れるという悪名高い落とし穴がありました。

PEP 615 は、システムまたは公式のIANAタイムゾーンデータベースを標準で参照できる `zoneinfo` モジュールを導入しました。

---

## 2. 仕様と構文

- **`ZoneInfo(key)`**: `"Asia/Tokyo"`, `"America/New_York"` などのIANAキー文字列からタイムゾーンオブジェクトを生成。
- **標準 `datetime` との完全統合**:
  コンストラクタの `tzinfo` 引数や `astimezone()` メソッドにそのまま渡すだけで、夏時間の切り替えを含めて正確に計算可能。

```python
# zoneinfo の基本生成とタイムゾーン変換
jst = ZoneInfo("Asia/Tokyo")
utc_now = datetime.now(timezone.utc)
jst_now = utc_now.astimezone(jst)  # 日本標準時に変換
```

---

## 3. 最小コード例

```python
from datetime import datetime
from zoneinfo import ZoneInfo

# 東京の現在時刻
tokyo_tz = ZoneInfo("Asia/Tokyo")
dt_tokyo = datetime(2026, 10, 26, 11, 0, tzinfo=tokyo_tz)

# ニューヨーク時間へ安全に変換
ny_tz = ZoneInfo("America/New_York")
dt_ny = dt_tokyo.astimezone(ny_tz)

print("Tokyo:", dt_tokyo)
print("New York:", dt_ny)
```

### 実行出力
```text
Tokyo: 2026-10-26 11:00:00+09:00
New York: 2026-10-25 22:00:00-04:00
```

---

## 4. 実務上の指針と落とし穴

- **Windows環境でのデータ配布**:
  POSIX環境（Linux/macOS）ではOS側のタイムゾーンデータ（`/usr/share/zoneinfo`）が自動使用されますが、WindowsではOSにIANAデータベースがないため、PyPIの `tzdata` パッケージを併せてインストールすることが推奨されます。
- **`pytz` からの完全脱却**:
  新規プロジェクトでは `pytz` の使用は非推奨であり、標準の `zoneinfo` への一本化が推奨されます。
  ```python
  # pytz のような localize() は不要
  # 常にコンストラクタか astimezone で直接指定する
  dt = datetime(2026, 1, 1, 12, 0, tzinfo=ZoneInfo("Asia/Tokyo"))
  ```

---

## 5. 関連リンク

- [[versions/py39|Python 3.9 概要]]
- [[ROADMAP|全体ロードマップ]]

---

## 6. 参考資料

- [PEP 615 – Support for the IANA Time Zone Database in the Standard Library](https://peps.python.org/pep-0615/)
- [Python 公式ドキュメント: zoneinfo](https://docs.python.org/3/library/zoneinfo.html)
