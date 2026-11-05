**English** | [正體中文](README.zh-TW.md)

# ASD-STE100 Skill — 為 agent 輸出而設的簡化技術英語

一個 Claude Code skill，把冗長、有歧義的英文改寫為 [ASD-STE100 簡化技術英語](https://www.asd-ste100.org/)（STE）——這是航太與國防產業為了讓飛機維修指令不可能被誤讀而建立的受控語言標準。

這個 skill 把同一套紀律轉用到另一種讀者身上：**AI agent**。它要解析另一個 agent 的輸出、工具描述、錯誤訊息或 agent 之間的指令，而流程中沒有人可以釐清歧義。

## 為何用 STE，為何為 agent 而用

STE 之所以存在，是因為飛機上一句被誤讀的指令可能致人於死，而預設讀者往往不是英語母語者，也沒有作者可以打電話問清楚。這套標準的解法是：一個詞只有一個意思、主動語態、簡單時態、一句一個指令、句子要短、不省略詞。

LLM agent 解析另一個 agent 的輸出時，處境驚人地相似——沒有回傳通道，沒辦法問「你是指 X 還是 Y？」。讓技師不會誤讀扭力規格的同一套規則，也讓下游 agent 不會誤讀工具描述或 agent 之間的訊息。

## 改寫前／改寫後

| 改寫前 | 改寫後 |
|---|---|
| "This tool will attempt to synchronize state across the various backends that have been configured, and if a conflict is detected it may resolve it automatically depending on the strategy that has been set, or otherwise it will surface the conflict for manual review." | "The tool tries to synchronize state across the configured backends. If it finds a conflict, it reads the configured strategy. If the strategy allows automatic resolution, the tool may resolve the conflict without a user. If the tool does not resolve the conflict, it reports the conflict for manual review." |
| "An error may have occurred while processing your request due to a possible mismatch in the expected data format, which could be caused by an outdated client version." | "Your request may have failed. The cause may be a data format that does not match what the server expects. An outdated client can cause this mismatch. Check your client version." |

更多範例（包含 STE 官方規則本身的示範）見 [`examples/before-after.md`](examples/before-after.md)。

## 這個 skill 做什麼

1. **選擇模式。** **嚴格模式（Strict）** 涵蓋程序、錯誤訊息與工具描述。**STE 風格模式（STE-flavored）** 涵蓋 README、PR 描述與說明性散文。STE 風格模式保留句子紀律，但不套用固定詞彙的封鎖。
2. 讀懂輸入的英文文本。
3. 逐句標出每一處違規：詞義不明、現在完成式／複合時態、行為者不清的被動語態、一句多指令、過長的名詞堆疊、省略詞、句子過長、片語動詞、名詞化的動作、分號、模糊限定堆疊、行銷形容詞。
4. 改寫每個被標出的句子——不丟掉原文任何事實、條件或範圍限定詞。若較短的說法會失去必要的精確度，它保留較長的說法並標出這項取捨，而不是默默簡化。
5. 單獨輸出改寫後的文本——沒有前言、不宣告模式、不做變更摘要——若刻意保留某處不簡化，再加一行 `Kept as-is:` 註記。

若要求說明理由（「show the diff」、「which rules did it break」），則改為輸出逐條列出規則的改寫前／改寫後表格。

它檢查的結構規則都是機械性的——你可以指出是哪個詞或哪個標點違反了規則。依賴 ASD 字典的規則只標為建議、不強制執行；需要品味判斷的規則則留給你自己。

linter 只檢查結構模式。它不會比對原文與改寫、不會驗證要求強度是否維持不變，也不會證明改寫保留了原意。零違規的結果只代表已設定的結構檢查沒有發現問題。

這個確定性 linter 檢查分號、一份簡短的軟性片語動詞清單（spin up、reach out、dive into、kick off、circle back、touch base）、名詞化、行銷形容詞、被動語態、現在完成式（含 "has run" 這類不規則過去分詞）、過長句子、同義詞輪替，以及支援的清單項目中的懸空連接詞。它不檢查名詞堆疊長度（那需要詞性標註），也不認得清單以外的片語動詞，所以 "take off the panel" 會通過。它從不標出模糊限定或語氣。

在句長檢查中，一個「詞」是以空白分隔、且至少含一個字母或數字的 token。單一反引號的行內程式碼、清單標記、引用區塊標記，以及獨立標點（例如前後有空白的破折號）都不算詞。

這個 repo 自己的文字也無法通過 lint：規則表格引用了它們所禁止的模式，有些句子也偏長。用 `python scripts/ste-lint.py --baseline 36 SKILL.md` 檢查，並把結果當成範例而非缺陷來讀。

懸空連接詞規則檢查行首的清單標記：前置空白為零到三個、標記之後是 ASCII 空白。它支援無序標記 `-`、`*`、`+`，以及以 `.` 或 `)` 結尾的有序數字標記，例如 `1.` 或 `1)`。它會檢查縮排的續行，直到最後一個有意義的行為止。它不解析引用區塊內部的清單語法、惰性續行（lazy continuation），也不處理完整的巢狀清單語意。單獨一行若前置空白達四個以上，不視為清單標記。在作用中的清單項目內，縮排到推算出的內容欄位者視為續行文字。圍籬（fence）偵測在引用區塊與圍籬本身都遵循 CommonMark。圍籬以三個以上的反引號或波浪號開啟，在引用區塊內亦然。它以一段同字元、長度不短於開啟者、且其後只有空白或 tab 的序列關閉。引用區塊內的圍籬在引用結束時關閉。在 HTML 區塊中，`>` 開頭的行是文字，不會開啟圍籬。HTML 區塊以 `<div>` 這類區塊標籤起始，或以單獨佔一行的完整標籤起始（且該行不延續段落）。行內 HTML（例如文字行中的 `<span>`）不會開啟 HTML 區塊。在主掃描中，任何引用深度下的 HTML 區塊內都不會開啟圍籬。HTML 區塊也可以位於引用區塊內，並在引用結束時結束。懸空連接詞掃描不追蹤 HTML 區塊，所以在那裡仍會開啟圍籬。linter 不追蹤清單項目，因此接受任何縮排位置的圍籬。它也接受比開啟圍籬深最多三欄的關閉圍籬，在頂層亦然。清單項目內的圍籬不會在項目結束時關閉。從清單標記同一行開始的圍籬不會被偵測到；它的關閉行反而會開啟一個圍籬，linter 便跳過其後的文字。

刻意無效的 examples/linter-edge-cases.md 檔示範了不完整的 Markdown 清單項目。執行 python scripts/ste-lint.py examples/linter-edge-cases.md，確認 linter 回報預期的兩項 finding。此檔是測試 fixture，刻意保留英文，不應當成合規的 STE 文字使用。

它**不會**重製 ASD 官方約 900 字的核准詞字典。這套標準可免費取得，但不可自由再散布：第 9 版僅允許在取得 ASD 書面授權時重製，或由八類列名組織重製，而本專案不屬於其中任何一類。這個 skill 套用的是背後的*原則*（選用最平實的可用詞，且每次用法一致），而不是比對固定的詞表。若需要經認證、符合 STE 的文件，請使用真正的標準。

完整規則摘要與引用來源：[`references/writing-rules.md`](references/writing-rules.md)。

## 安裝

### 快速安裝（npx skills）

安裝這個 skill 最快的方式是 [skills CLI](https://skills.sh/)——不必 clone、不必設定路徑。在專案根目錄執行：

```bash
npx skills add danyuchn/asd-ste100-skill
```

這會從 GitHub repo 取得 skill 並安裝到目前專案。CLI 會送出匿名安裝遙測（skill 名稱與時間戳，不含個人或裝置資訊），用於 skills.sh 排行榜排名。設定 `DISABLE_TELEMETRY=1` 可退出。

日後用 `npx skills update` 更新。

### Clone

```bash
git clone https://github.com/danyuchn/asd-ste100-skill ~/.claude/skills/asd-ste100
```

這會把 repo clone 到 `~/.claude/skills/`，讓 skill 在每個 Claude Code 專案都能使用。適合貢獻者，以及想要一份可用 `git pull` 更新的即時 checkout 的人。

## 使用方式

用「簡化或釐清英文文本」的請求觸發：

```
Disambiguate this tool description
Rewrite this error message so an agent can't misparse it
Apply ASD-STE100 to this instruction
```

或貼上文字，請 Claude「disambiguate this」／「apply STE100 to this」／「reduce ambiguity in this output」。

你只會拿回改寫後的文字，沒有其他東西。若想知道套用了哪些規則，在請求中加上「show the diff」或「explain the changes」。

## 適用範圍

適用於：agent 對 agent 的訊息、工具／函式描述、錯誤訊息、系統提示、agent 之間的指令——任何機器或非母語讀者必須在無人可問的情況下解析的英文文本。

不適用於：創意寫作、行銷文案，或任何以語氣與細微差異為重點的內容——STE 在設計上刻意平板、字面。

有一個限制必須先講明：它修的是文本的形式，不是內容。一段沒東西可說的文字，改寫後會變短、變乾淨，但仍然空洞。

## 其他語言

- **正體中文（台灣）:** [tboydar/asd-ste100-skill-zh-tw](https://github.com/tboydar/asd-ste100-skill-zh-tw) 是本 skill 的正體中文譯本，含 `README.zh-TW.md`、`SKILL.zh-TW.md`、`references/writing-rules.zh-TW.md` 與 `examples/before-after.zh-TW.md`。ASD-STE100 只定義英文，中文譯本為說明文件，不是官方標準。
- **希伯來文:** [g4lb/asd-ste100-hebrew-skill](https://github.com/g4lb/asd-ste100-hebrew-skill) 是本 skill 的希伯來文移植版，內含希伯來文 linter。ASD-STE100 只定義英文，因此希伯來文規則是改編，不是官方標準。
- **印尼文:** [jshsinaga/asd-ste100-indonesian-skill](https://github.com/jshsinaga/asd-ste100-indonesian-skill) 把本 skill 改編給印尼文技術文本使用。其印尼文規則是改編，不是官方 ASD-STE100 標準。

## 相關工具

- [meiluosi/rewrite-fidelity-skill](https://github.com/meiluosi/rewrite-fidelity-skill) 比對原文與改寫，標出語意與語氣上的變動。它使用樣式比對，所以每一項標記都要自己再確認。

## 資料來源

- [ASD-STE100 官方網站](https://www.asd-ste100.org/)
- [ASD-STE100 — 關於 STE](https://www.asd-ste100.org/about_STE.html)
- [ASD Europe — 簡化技術英語](https://www.asd-europe.org/standards-specifications/simplified-technical-english/)
- [簡化技術英語 — 維基百科](https://en.wikipedia.org/wiki/Simplified_Technical_English)
- [TechScribe — ASD-STE100 簡化技術英語](https://www.techscribe.co.uk/techw/asd-simplified-technical-english.htm)

## 授權

MIT——見 [LICENSE](LICENSE)。
