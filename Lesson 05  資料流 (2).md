<h2 align="center">Lesson 05: 資料流(2)</h2>

### 5-1 資料寫入 (produce)
1. 參數提供
- 需言明 topic 及至少一台 broker (以連線至叢集) 方可匯入。
- key 可不提供但 value 不得為空值，依 key 決定資料寫入 partition 之模式：輪放 (round robin) 或類叢。<br>
例：topic, broker 及 value 為必要提供，key 及 partition 則為非必要。

2. 寫入模式
- fire-and-forget: producer.poll()<br>
資料發送後便繼續運作。效率最高，但有遺失之慮。
- synchronous send: `producer.flush()`<br>
同步化追蹤資料之寫入。確保資料完整，但效率低。
- asynchronous send: `producer.poll()`+callback(觸發非同步狀態回報)<br>
非同步化追蹤資料寫入。完整性及效能之折衷做法。

3. 其他寫入設定
- acks: 分為 -1 (最完整)、0 (高效率)、+1 (折衷)，詳見 2-1。
- max.in.flight.requests.per.connection: 設定為 1，可確保紀錄寫入順序。<br>
例：確保紀錄依 key 寫入，則需提供 key 且不中途更動 partition。
- retries (寫入失敗時重試)、serializer (序／反序列化寫入) 等。

---
### 5-2 資料讀取 (consume)
1. 參數提供
- 需言明 topic 及至少一台 broker (以連線至叢集)方可讀出。

2. consumer group
- consumer 可組成 group 共同讀取，成員可讀多份 partition，然同一 partition 只會被一成員讀取。<br>
藉此防範資料拆分，故當成員數＞partition，將導致有 consumer 工作閒置。
- kafka 自動建立 __consumer_offset 以儲放資料之讀取紀錄 (commit record)。<br>
key=\[group, topic, partition], value=\[offset (工作位置，即最新尚未讀取部分)]<br>
以 group (id) 為單位，故群組成員增減雖導致 rebalance (再平衡)，但不影響整體讀取工作。

3. 再平衡 (rebalance)
- 當 consumer 提出讀取要求，連線之 broker 即需統籌讀取工作。
- 若為多機台組成 consumer group，broker 則需負責工作分配，稱為協調者 (group coordinator)<br>
同時群組需選出 consumer leader，作為工作對接窗口。
- 工作步驟
  - heartbeat: 協調者定時發送信號，偵測群組成員是否異動。
  - 協調者將成員名單發送給 consumer leader，由 leader 完成工作分配 (partition assignment)。
  - 協調者將 leader 回傳之工作分配發送給所有群組成員。<br>
  再平衡觸發後，成員負責 partition 可能改變，__consumer_offset 便可提供讀取進度之查考。
- 觸發時機
  - 群組成員變動 (新增、刪除、忙線而被判定 dead，不再接收 assignment)
  - partition 變動 (新增)

4. 讀取模式<br>
提出讀取要求，一次 (要求) 之回應可能包含一至多筆紀錄 (record) 之讀取。

- automatic commit: `enable.auto.commit=true`<br>
對於讀取回應，以指定週期做 commit，效率穩定但缺乏彈性、無法處理中途漏讀資料。
- synchronous commit: `asynchronous=false`<br>
對於讀取回應，逐次同步化做 commit，資料完整但缺乏效率。
- asynchronous commit: `asynchronous=true+on_commit` (類似 callback)<br>
對於讀取回應，逐次非同步做 commit，為完整性及效能折衷。
- commit-specified:<br>
對於讀取回應中之逐筆資料做 commit。

5. 其他讀取設定
- rebalance listener: 再平衡觸發時之回應<br>
讀取工作即時 commit、列印觸發前後之分工變化 (on_revoke, on_assign)。
- auto.offset.reset: 讀取模式，分為最初 (earliest) 及創建後 (latest)<br>
缺乏紀錄時，則依模式由不同處讀取起；若有紀錄則以 __consumer_offset 為準。
