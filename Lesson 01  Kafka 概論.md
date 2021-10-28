<h2 align="center">Lesson 01: Kafka 概論</h2>

### 1-1 導論
1. Kafka 簡介
- 資料在寫入、讀取間，如果發生端口的故障將會導致損失；Kafka 即是扮演`中繼的暫存角色`。
- 因此，它本身並`不負責運算及長期儲存` (非資料庫)，而是做為訊息收／發的一個中繼傳輸點。
- 常適用於讀寫工作龐雜，且亟需同步化、高可用及延展性的商業運作中。

2. 基本角色
- 輸入源 (source)：稱為 `producer。負責將資料寫入`，可為 Oracle, PyCharm 等。
- 輸出槽 (sink)：稱為 `consumer。負責將資料讀出`，可為 PyCharm, ES (elastic-search) 等。

3. 與 HDFS 之比較
- 均可做 `map reduce` (=partition, 資料分片)，透過平行運算增進處理效率。
- 均可做 `replication` (資料複製)，確保運作彈性及安全。
- 同以 `zookeeper` 管理多台叢集，並有 mirror-maker 確保資料的同步。
- HDFS 之叢集有主 (namenode)、從 (datanode) 之分；Kafka 則在 partition, consumer 安排 leader 角色，前者可以同步各個副本、而後者可協調 partition 的讀取。

---
### 1-2 核心名詞簡介
1. topic
- 形如資料庫中的`資料表`，是以 topic-name 作為區別。
- 由一至多台 broker 組成，即所謂`「多台叢集 (cluster)」`。

2. broker
- `主機伺服器`，可以是實體或虛擬機環境 (server)，以 id 作為區別。
- 為 topic 之實作對象，連上單一 broker 即可跟所有叢集的機台對接。
- 由一至多個 partition 所組成，即所謂`「資料分片 (map reduce)」`。

3. partition
- 作資料分散儲存用，為逐筆紀錄 (record) 寫入之處。
- 限定讀取於不同成員，故 N 份 partition 最多給 N 台 consumer group 讀取。
- 已寫入者無法變更。確保單一 partition 具備順序及 key 的一致性。

4. replication
- 作資料複製備份用，為異地備源保存之處。
- 限定實作於不同機台，故 N 台叢集 (cluster) 最多能有 N 份 replication。<br>
例：5 brokers 叢集，最多可設置 R.F.=5 (即各 broker 都具其他台之複製備份)，若設定 R.F.=3，則可容許 5-3 台 broker 發生離線或故障，藉此提升運作彈性。
