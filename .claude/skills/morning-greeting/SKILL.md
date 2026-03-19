---
name: morning-greeting
description: ユーザーが「おはよう」と言ったときに使うスキル。朝の挨拶をして、今日の調子を聞き、やることがあれば一緒に整理する。
---

# 朝の挨拶スキル

## このスキルを使うタイミング
ユーザーが「おはよう」「おはようございます」と言ったとき。

## やること
1. 今日の日付を確認して元気よく挨拶する
2. Google Calendarから今日の予定を取得して表示する（下記参照）
3. 「今日の調子はどう？」と聞く
4. 「他にやりたいことある？一緒に整理しようか？」と聞く

## Google Calendarの予定取得

環境変数 `GOOGLE_CALENDAR_ICS_URLS` にスペース区切りで複数のICS URLが設定されている。
以下のコマンドで全カレンダーの今日の予定をまとめて取得する:

```bash
python3 -c "
import subprocess, sys
from datetime import date

today = date.today().strftime('%Y%m%d')
urls = '$GOOGLE_CALENDAR_ICS_URLS'.split()
results = []

for url in urls:
    try:
        r = subprocess.run(['curl', '-s', '--max-time', '5', url], capture_output=True, text=True)
        content = r.stdout
        for event in content.split('BEGIN:VEVENT')[1:]:
            summary = dtstart = ''
            for line in event.replace('\r','').split('\n'):
                if line.startswith('SUMMARY:'):
                    summary = line[8:].strip()
                elif line.startswith('DTSTART'):
                    dtstart = line.split(':')[-1].strip()[:8]
            if dtstart == today and summary and summary not in results:
                results.append(summary)
    except Exception:
        pass

for r in sorted(results):
    print(f'- {r}')
"
```

- 予定がある場合: 挨拶の中に「今日の予定」としてリスト表示する
- 予定がゼロ件の場合: 「今日は予定なし！」と伝える
- `GOOGLE_CALENDAR_ICS_URLS` が未設定の場合: カレンダーなしで挨拶を続ける

## トーン
- 友達に話しかけるような明るい口調
- 敬語は使わない
- 短めにテンポよく
