<h2 align="center">Lesson 03: 複製、分片與分段</h2>

### 3-1 partition
1. 優、缺點
- 平行儲存處理，具備更高吞吐量 (throughput)。
- 叢集開檔繁多造成延遲；且當 leader 故障時，將有更多 (同步, insynchronous) 備選者。
- 若中途變更設定，無法確保 key 寫入同份 partition。

2. 數量上以不超過 10 為限，單台 broker 負責一至二份 partition。

---
### 3-2 replication
1. 優、缺點
- 備份複製保存，具備高彈性 (resilience) 及叢集容錯率。
- 多主機運作、回應易延遲 (latency)；且占據硬體儲放空間。
- 若中途變更設定，可能影響叢集負載及空間效能。

2. 數量上以 2~3 為原則，視 broker 數量決定 replication factor。

---
### 3-3 segment
1. 為了訊息記錄 (log)、管理之便，單一 partition 可再細分為多個 segment。

2. segment 可設定分段條件
- 容量分段 (log.segment.bytes)：達特定容量則新增一 segment，預設為 1GB。
- 時間分段 (log.roll.hours/ms)：達特定時間則新增一 segment，預設為 1 週。

3. 承上，資料 (record) 在實際寫入 segment 時，也同時攜帶 offset (位置) 及 timestamp (時間)兩項參數。<br>
故從 topic 擷取出的一筆資料，除 topic, broker, partition 及 key-value 外，尚有 offset 及 timestamp 等訊息。

4. 除正在寫入之 segment 為 active 外，其餘均為 non-active。
