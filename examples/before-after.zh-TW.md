# 改寫前／改寫後範例

## 第一部分——STE 官方範例

這些範例示範真正的 ASD-STE100 規則，取自公開的二手來源（見 `references/writing-rules.md`）。它們是規則的改寫示範，不是標準本身的引文。

| 規則 | 改寫前 | 改寫後 | 原因 |
|---|---|---|---|
| 一詞一義 | "Verify the system." / "Check the connections." / "Confirm receipt." | "Make sure the system is correct."（固定使用一個核准詞） | 三個近義詞迫使讀者猜測它們是否指同一個動作。 |
| 一詞一詞性 | "Oil the valve." | "Apply oil to the valve." | 若 "oil" 只核准為名詞，把它當動詞用就破壞了一詞一角色的保證。 |
| 動詞意思精確 | "Follow the safety instructions." | "Obey the safety instructions." | "Follow" 可以指「跟在後面」或「遵守」——STE 選用沒有歧義的那個。 |
| 只用簡單時態 | "We have received the technical reports from HQ." | "We received the technical reports from HQ." | 現在完成式多出第二種解析（「收到了，而且現在仍然相關？」），簡單過去式避開了它。 |
| 用動詞，不用名詞 | "Perform an inspection of the filter." | "Inspect the filter." | 名詞形隱藏了動作，還加了一個不帶意義的填充動詞。 |
| 不用片語動詞 | "Take off the access panel." | "Remove the access panel." | "Take off" 也可以指「起飛」和「扣除」——兩個字合起來無法預測意思。 |

## 第二部分——應用於 agent 輸出

以下是為這個 skill 實際用途建立的原創範例：改寫 AI agent 輸出，讓另一個 agent、翻譯層或非母語讀者能無歧義地解析。它們是示範，不是任何真實系統的引文。

以下詞數以空白分隔的 token 計算（`text.split()`），標點不另外計數。換用不同的分詞器會得到不同數字。

### 範例 A——工具描述

**Before:**
> This tool will attempt to synchronize state across the various backends that have been configured, and if a conflict is detected it may resolve it automatically depending on the strategy that has been set, or otherwise it will surface the conflict for manual review.

**標出的違規：**
- 一句兩個指令（同步 + 解決／回報）。
- 關係子句中的現在完成式（"have been configured"、"has been set"）。
- 44 詞，遠超描述性文本 25 詞上限。

注意*沒有*被標出的部分："will attempt to" 與 "may resolve"。那些是模糊限定，不是違規。這個工具並未被承諾會成功，改寫也不能承諾。

**After:**
> The tool tries to synchronize state across the configured backends. If it finds a conflict, it reads the configured strategy. If the strategy allows automatic resolution, the tool may resolve the conflict without a user. If the tool does not resolve the conflict, it reports the conflict for manual review.

最後一句的分支依據是衝突是否已被解決，而不是策略允許什麼。這就是原文 "or otherwise" 的意思：備援涵蓋的是「被允許、但終究沒有發生」的解決。

### 範例 B——錯誤訊息

**Before:**
> An error may have occurred while processing your request due to a possible mismatch in the expected data format, which could be caused by an outdated client version.

**標出的違規：**
- 一句承載三項獨立主張（發生錯誤、格式不符、客戶端版本）。
- 28 詞，超過描述性上限。

未標出："may have occurred" 與 "could be caused by"。這則訊息由一個不知道哪裡出錯的系統所寫。兩個模糊限定都是對這份無知的準確陳述。

**After:**
> Your request may have failed. The cause may be a data format that does not match what the server expects. An outdated client can cause this mismatch. Check your client version.

**這個範例正是語氣規則存在的理由。**本檔較早的版本把第一句改寫成 "The request failed"，把第三句改寫成 "an outdated client **is the most common cause**"。兩者讀來都更順，兩者都錯：前者斷言了系統只是懷疑的失敗，後者憑空加入原文完全沒有的頻率主張。補上原因、頻率或機制的改寫，已經不再是改寫。

另請注意，"may have failed" 保留了簡單時態規則原本會移除的複合動詞形式。**當時態規則與語氣規則衝突時，語氣優先**——在這裡刪掉助動詞，會連同時態一起刪掉那份不確定性。

### 範例 C——agent 之間的指令

**Before:**
> Once the upstream job has completed and assuming no errors were raised, the downstream agent should proceed to consume the output artifact, though it is worth noting that partial artifacts are sometimes produced under timeout conditions.

**標出的違規：**
- 現在完成式（"has completed"）與從屬子句堆疊（"assuming..."、"though it is worth noting..."）。
- 一句三個獨立事實（完成條件、下一步動作、邊界情況警告）。
- 36 詞，超過指令 20 詞上限。

**After:**
> Wait for the upstream job to finish with no errors. Then read the output artifact. Warning: a timeout can produce a partial artifact. Check that the artifact is complete before you use it.

兩項刻意決定，值得說明而不是藏起來：
- "should proceed to consume" 變成祈使句 "read"。STE 允許在指令中這樣做——對執行 agent 提出的建議就是命令。在描述性文本中不要做同樣的轉換。
- 最後一句是**新增的**。原文警告了部分產物，卻沒說該怎麼辦。加上檢查讓警告可執行，但那是新增內容，所以在這裡標明，而不是當成改寫蒙混過去。若原文的沉默是刻意的，就刪掉這一句。

### 範例 D——README 散文（STE 風格模式）

**Before:**
> Our caching layer is designed to slot seamlessly into your existing stack with minimal friction and no vendor lock-in; it leverages semantic similarity to dramatically reduce the cache misses that traditionally plague LLM workloads.

**標出的違規：**
- 沒有量測支撐的行銷形容詞與宣稱（"seamlessly"、"minimal friction"、"dramatically"）。
- 用分號連接兩個獨立想法。
- 名詞化與軟性措辭（"is designed to slot into"、"leverages"）。
- 34 詞，超過描述性 25 詞上限。

**After:**
> We designed our cache layer to fit into your current stack without vendor lock-in. It uses semantic similarity to reduce cache misses, a long-standing problem for LLM workloads.

風格模式沒有強制每個概念只用一個固定詞。它仍然砍掉了行銷形容詞、分號與長度。它保留了 "designed to"，因為原文陳述的是設計目標，不是結果。

本檔較早的版本加了兩項事實：說一般快取以完全相同的文字比對請求，又說這個快取不會在你的技術棧之外儲存資料。原文兩者都沒說。加入事實的改寫，已經不再是改寫。

## 如何閱讀這些範例

第一部分展示這個 skill 所依據的實際規則。第二部分展示轉用：同一套紀律——一詞一義、主動語態、簡單時態、一句一個指令、明確寫出條件而非埋在從屬子句裡——讓機器對機器與跨語言的文本更安全可解析，不只是飛機手冊。
