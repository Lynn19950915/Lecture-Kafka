<h2 align="center">Lesson 02: 資訊流(1)</h2>

### 2-1 輸入源
1. 由 producer 負責，需言明 topic 及 (至少一台) broker 方可匯入資料。

2. 資料為逐筆寫入，以字典 (key-value) 方式寫入。在不中途更動 partition 之下，key 能確保儲入同份 partition。<br>
例: 寫入一筆紀錄 (record) 時，topic, broker 及 value 為必需提供；key 及 partition 則非必要。

3. 輸入模式 acks=<br>
- -1，等待所有叢集之回應，具備高完整性。<br>
- 0，不等待任何機台回應，具備高度效率。<br>
- +1，待 partition leader 回應，折衷作法。

---
### 2-2 輸出槽
1. 由 consumer 負責，需言明 topic 及 (至少一台) broker 方可讀出資料。

2. consumer 可組成 group 共同讀取，成員可讀多份 partition，然同一 partition 只會被一成員讀取。<br>
藉此防範資料拆分，故當成員數＞partition，將導致有 consumer 工作閒置。

3. __consumer_offset
- 用以儲放讀取紀錄 (commit record) 的資料表，是 Kafka 自動建立之 topic。
- key=\[group, topic, partition], value=\[offset(工作位置，即最新尚未讀取部分)]。
- 以 group 為單位，故群組成員之增減雖導致 rebalance (再平衡)，但不影響整體讀取工作。
- topic+partition 確保訊息的寫入與讀出均合乎順序 (FIFO, first-in first-out)。

4. 讀取模式<br>
- <1，當資料傳出便記錄，無法確保 consumer 讀取。<br>
- =1，資料送達時即記錄，確保資料恰被讀取一次。<br>
- \>1，資料讀取後才記錄，資料有可能被重複讀取。
