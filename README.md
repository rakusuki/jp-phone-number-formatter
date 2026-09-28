# jp-phone-number-formatter

クリップボード上の日本の電話番号を判定し、適切な位置にハイフンを挿入してクリップボードへ戻す Chrome Bookmarklet です。

A Chrome bookmarklet that formats supported Japanese phone numbers with the appropriate hyphen placement and writes the result back to the clipboard.

## Supported formats / 対応形式

| Input | Output | Type |
|---|---|---|
| `09012345678` | `090-1234-5678` | Mobile |
| `08012345678` | `080-1234-5678` | Mobile |
| `07012345678` | `070-1234-5678` | Mobile |
| `05012345678` | `050-1234-5678` | IP phone |
| `0120123456` | `0120-123-456` | Toll-free |
| `0312345678` | `03-1234-5678` | Geographic fixed line |
| `0421234567` | `042-123-4567` | Geographic fixed line |
| `0467123456` | `0467-12-3456` | Geographic fixed line |
| `0597921234` | `05979-2-1234` | Geographic fixed line |
| `9012345678` | `090-1234-5678` | Mobile (leading 0 omitted) |
| `5012345678` | `050-1234-5678` | IP phone (leading 0 omitted) |
| `120123456` | `0120-123-456` | Toll-free (leading 0 omitted) |
| `312345678` | `03-1234-5678` | Geographic fixed line (leading 0 omitted) |

`0570` and `0800` are intentionally rejected by the current project specification. Other special-purpose/service-number ranges are not accepted unless explicitly supported above.

## Usage / 使い方

1. `bookmarklet/bookmarklet.js` の1行全体をコピーします。
2. Chromeで新しいブックマークを作成し、URL欄へ貼り付けます。
3. ハイフンなしの電話番号をクリップボードへコピーします。
4. 作成したブックマークレットを実行します。
5. 成功すると、ハイフン付き電話番号でクリップボードが上書きされ、結果がダイアログ表示されます。
6. 非対応形式の場合はエラーダイアログを表示し、変換しません。

> Clipboard access is subject to the browser's security model. Chrome may refuse clipboard access on pages or contexts where the Clipboard API is unavailable.

## Leading-zero completion / 先頭0の自動補完

先頭の `0` が省略された半角数字のみの入力は、`0` を一時的に補完して既存の電話番号判定へ渡します。補完後の番号が対応形式として成立する場合のみ変換します。

```text
9012345678 → 09012345678 → 090-1234-5678
312345678  → 0312345678  → 03-1234-5678
```

補完後にも既存の拒否ルールを適用します。そのため、`570123456` は `0570...`、`8001234567` は `0800...` として拒否されます。判定不能な番号を、0補完だけを理由に推測して整形することはありません。

## Fixed-line handling / 固定電話の判定

日本の固定電話は市外局番の桁数が一定ではありません。この実装では既知の地理的市外局番を長いものから照合し、市外局番・市内局番・加入者番号（4桁）へ分割します。単純な `3-3-4` 固定分割は行いません。

市外局番体系の確認には、総務省「電気通信番号制度 市外局番の一覧」を基準資料として使用してください。番号計画は変更される可能性があるため、データ更新時には公式資料との再照合を推奨します。

## Development / 開発

Requires Node.js 18 or later.

```bash
npm test
```

No runtime package dependency is required.

## Project structure

```text
jp-phone-number-formatter/
├── bookmarklet/
│   └── bookmarklet.js   # Chromeに登録する1行版
├── src/
│   └── formatter.js     # 判定・フォーマット本体
├── test/
│   └── formatter.test.js
├── .gitignore
├── LICENSE
├── package.json
└── README.md
```

## Design policy

- 入力はハイフンなしの半角数字のみ。
- 先頭 `0` が省略されている場合は `0` を仮補完してから既存の判定を行う。
- `070` / `080` / `090` は携帯電話として `3-4-4` に整形。
- `050` は `050-XXXX-XXXX` として許可。
- `0120` は `0120-XXX-XXX` として許可。
- `0570` / `0800` は現仕様では明示的に拒否。
- 固定電話は既知の市外局番に一致した場合のみ許可。
- 判定不能な番号を推測して整形しない。

## License

MIT License. See `LICENSE`.
