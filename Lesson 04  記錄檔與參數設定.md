<h2 align="center">Lesson 04 記錄檔與參數設定</h2>

### 4-1 記錄檔 (log-cleanup)
1. log 檔儲存資料寫入、讀取之動作記錄，隨資料龐生有兩種清理模式。

2. log.cleanup.policy=delete
- 預設於一般 user-topics，原理與 segment 分段相同，需設定驅動條件。
  - 容量清理 (log.retention.bytes)：僅保留特定容量大小之動作記錄，預設為 -1 (無限)。
  - 時間清理 (log.retention.hours)：僅保留最近時間區段之動作記錄，預設為168 (一週)。
- 兩者擇一即可，如 bytes=500 (MB)、hours=17520 (兩年，相當於無限)。
- 資料無法往前回溯，僅可處理限定範圍內之維護 (maintenance)。

3. log.cleanup.policy=compact
- 預設於 __consumer_offsets，以 key 為記錄維護主體，無時間、容量限制。
  - non-active: 僅保留 segment 中相同 key 之最後一筆讀、寫狀態。
  - active: 不清理正在寫入之 segment，故所有動作記錄均會保留。
- 例：只要涉及 active segment 就可能讀到重複之 key 資料，除非單看 non-active。
- 資料可以往前回溯，但無法被更改或重組 (re-order) 排序。

---
### 4-2 自定義參數
1. Kafka 之預設參數皆可藉指令手動更改，包含 topic, broker configuration。

2. topic configuration
- 針對整個 topic (資料表) 設定，可參考[官方文件](https://kafka.apache.org/documentation/#topicconfigs)。
- 常見參數
  - compression.type: 針對非位元 (non-binary) 資料，如json, xml 及文本 (text) 等。
  - max.message.bytes (default=1MB)、min.insync.replicas (default=1) 等。

3. broker configuration
- 針對單台 broker (環境) 設定，可參考[官方文件](https://kafka.apache.org/documentation/#brokerconfigs)。
- 常見類型
  |  | 重新啟動執行 | 針對機台數 | 設定型態 | 儲放位置 |
  | :---: | :---: | :---: | :---: | :---: |
  | read-only | Yes | 單台 | 靜態 | server.properties |
  | per-broker | No | 單台 | 動態 | zookeeper |
  | cluster-wide | No | 所有 cluster | 動態 | zookeeper |

---
### 4-3 自定義格式
1. Kafka-topics 格式
- 只可設定 topic configuration，需載明 zookeeper+topic 名稱。
- 功能指令
  | 操作 | 語法 |
  | :---: | :---: |
  | 增 (C) | --create／--config |
  | 查 (R) | --describe |
  | 改 (U) | --alter／--config |
  | 刪 (D) | --alter／--delete-config |

2. Kafka-configs 格式
- 可以設定 topic 及 broker configuration，需載明 zookeeper+entity 參數。<br>
例：--entity-type(topics/brokers)、--entity-name(topic name/broker id)。
- 功能指令
  | 操作 | 語法 |
  | :---: | :---: |
  | 查 (R) | --describe |
  | 改 (U) | --alter／--add-config |
  | 刪 (D) | --alter／--delete-config |
