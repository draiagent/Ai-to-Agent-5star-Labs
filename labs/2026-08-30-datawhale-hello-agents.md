# Lab Note｜Datawhale Hello-Agents：用 ReAct 打造第一個可驗收的 Agent

> 範例 Lab Note，供貢獻者參考格式。填寫方式對應 [`templates/lab-note.md`](../templates/lab-note.md)。

## 1. 基本資料

- Lab：Datawhale
- 主題：以 Hello-Agents 的 ReAct 章節，做一個「查資料 → 計算 → 產出結論」的最小 Agent，並訂出驗收標準
- 官方來源：
  - Repo：<https://github.com/datawhalechina/hello-agents>
  - 線上教材：<https://datawhalechina.github.io/hello-agents/>
- 查核日期：2026-08-30
- 版本／Release／討論日期：以查核日 `main` 分支最新 commit 為準（Lab Note 應記錄實際 commit hash）
- 原始授權：教材內容依 Hello-Agents Repo 標示之授權；本 Lab Note 僅摘要並附連結，不整段複製原文

## 2. 為什麼值得研究？

- 要解決的企業問題：讓非工程背景的主管理解「Agent 不是更大的 Chat」，而是「會規劃、會用工具、可被驗收」的執行單元
- 適用對象：企業內訓入門場、想建立共同語言的跨部門團隊
- 預期價值：用一個 30 分鐘可跑完的例子，把 Thought／Action／Observation 循環講清楚，之後所有 VAD／VAC 都有共同起點
- 已知限制：教材以簡體中文為主、範例多用國際模型 API；繁體中文與在地模型表現需另到 ModelScope Lab 補測

## 3. Verify

- [x] 官方入口存在（Repo 與 GitHub Pages 教材 2026-08-30 實測可開啟）
- [x] 近六個月仍有更新或討論
- [x] 與 AI to Agent 高度相關（ReAct、Plan-and-Solve、Reflection、Memory、Multi-Agent 為主軸）
- [x] 原始來源可追溯（章節對應 Repo 檔案與 commit）
- [x] 授權適合本次用途（僅摘要 + 連結 + 改寫，不轉載）
- [x] 實驗可以重現（步驟、環境、模型與參數如下）

## 4. VAD｜Visual Agent Design

- 目標：輸入一個問題（例：「某產品毛利率是否高於 40%？」），Agent 需自行取得數字、計算、給出「是／否 + 理由 + 資料來源」
- 輸入素材：一份 3–5 列的假資料 CSV（售價、成本），問題字串一則
- 流程：
  1. 解析問題 → 決定需要哪些欄位
  2. Action：讀取 CSV 對應列
  3. Observation：取得售價與成本
  4. Thought：計算毛利率 = (售價 − 成本) / 售價
  5. 與門檻比較 → 產出結論
- 模型／Agent：入門用一顆中階指令模型即可；框架用教材內建的 ReAct 迴圈
- Tools／MCP：`read_csv`（本地檔案讀取）、`calculator`（避免模型自行心算出錯）
- 輸出格式：
  ```json
  { "answer": "是/否", "gross_margin": 0.0, "threshold": 0.4, "evidence_rows": [1,2], "reason": "一句話" }
  ```
- 人工決策點：資料來源是否可信、門檻數字是否由業務確認
- 驗收標準：
  - 數值計算與人工試算一致（誤差 < 0.5%）
  - `evidence_rows` 必須指回實際使用的資料列
  - 換一個模型重跑，`answer` 與 `gross_margin` 不變

## 5. VAC｜Visual Agent Capability

- 能力名稱：`tabular-threshold-check`（表格數據門檻判斷）
- 觸發條件：問題可歸約為「某指標是否 ≥／≤ 門檻」，且資料在結構化表格中
- 執行步驟：解析指標 → 定位資料列 → 呼叫計算工具 → 對比門檻 → 產出帶證據的結論
- 模型選擇原則：已知結構、低風險 → 中階模型；資料含歧義欄位或需推斷定義 → 升級高階模型並要求人工確認
- 升級高階模型條件：欄位語意不明、需跨表 join、或結論會直接影響對外報價／財報
- 失敗修復：
  - 模型心算錯 → 強制走 `calculator` 工具，禁止直接輸出數字
  - 取錯列 → 要求先回報 `evidence_rows` 再算，人工可攔截
  - 幻覺資料來源 → 輸出必須含 `evidence_rows`，空值即判為未通過
- 可重用資產：VAD 圖、上述 JSON schema、驗收清單、假資料 CSV

## 6. 實驗結果

> 下表為範例填寫，實際數字以自己的執行為準。

| 指標 | 結果 | 備註 |
|---|---|---|
| 完成率 | 5/5 | 五題毛利率判斷全數符合輸出格式 |
| 執行時間 | 約 8 秒／題 | 含工具往返 |
| Token／API 成本 | 約 1.5–2K tokens／題 | 中階模型 |
| 人工修改次數 | 0 | 提示詞穩定後不需人工介入 |
| 錯誤率 | 0% | 強制走 calculator 後心算錯誤消失 |
| 跨模型一致性 | 一致 | 換第二顆模型，answer 與 gross_margin 相同 |
| 可重現性 | 可 | 同 CSV、同提示詞、同門檻可重跑 |

## 7. 教材轉化

- 課程名稱：AI to Agent 模組 1｜從 Chat 到可驗收的 Agent
- 學習目標：能畫出一個 ReAct 任務的 VAD，並寫出至少 3 條驗收標準
- 課堂任務：學員各自改寫問題（換指標、換門檻），跑通後互相驗收對方的 `evidence_rows`
- 評量方式：用 [`docs/course-blueprint.md`](../docs/course-blueprint.md) 的驗收量表，重點看「可重現性」與「風險控制」
- 可製作的圖卡／簡報／短影音：ReAct 循環一頁圖卡、30 秒「Agent 如何自己找答案」示範影片

## 8. 結論

- 是否進入正式教材：是（作為模組 1 的入門實作）
- 下一步：
  1. 到 ModelScope Lab 用同一個 VAC 測繁體中文與在地模型
  2. 到 V2EX Agent Lab 找「工具呼叫失敗 / 上下文污染」的真實討論，補成風險案例卡
  3. 記錄本次使用的實際 commit hash 與模型版本，回填第 1 節
