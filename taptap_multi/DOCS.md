# TapTap Multi：多台 Tigo CCA 本地監控

這是 [litinoveweedle/hassio-addons](https://github.com/litinoveweedle/hassio-addons/tree/main/taptap) 的 **TapTap 附加元件（v0.3.4）** 的本地修改版，可以在**同一個附加元件裡同時監聽多台 Tigo CCA**（每台 CCA 各接一顆 RS485 轉換器，例如 Elfin EW11）。

## 運作方式

- 底層程式完全使用原作者的版本：
  - [taptap](https://github.com/litinoveweedle/taptap) v0.2.6
  - [taptap-mqtt](https://github.com/litinoveweedle/taptap-mqtt) v0.2.6
- 只改了附加元件這一層：`instances` 底下每一個項目，都會各自產生一份設定檔，並啟動一個獨立的 taptap-mqtt 程序。
- 每個 instance 的 `name` 會作為：
  - HA 裡的裝置名稱，以及實體 ID 的開頭（例如 `sensor.tigo_p01_power`、`sensor.tigo2_p12_power`）
  - MQTT 主題 `taptap/<name>/...`
  - 各自的狀態檔 `/data/taptap_<name>.json`
- Log 裡每一行前面都會加上 `[name]`，可以分辨是哪一台 CCA 的訊息。
- 任何一個 instance 異常結束時，會**先平順關閉其他所有 instance，再讓整個附加元件結束**。打開「看門狗（Watchdog）」的話，HA 會自動把它重新啟動。

## 硬體（每台 CCA 各一顆 EW11）

| EW11 | 接到 |
|---|---|
| A / B | CCA **GATEWAY** 端子的 A / B（跟原本 TAP 的線並聯） |
| − | 電源的 −，**同時也拉一條到** CCA GATEWAY 的 − |
| + | 電源的 +（5–18 V） |

EW11 設定：
- **Serial**：38400、8N1、Flow Control 設 Disable、**Protocol 設 None**、Cli 設 Disable
- **Communication**：TCP Server、Local Port（例如 8899）、Timeout 設 0、**Max Accept 設 2 以上**（方便用 `nc` 測試）

改完之後**一定要重新啟動 EW11**，再用以下指令確認資料正確：

```bash
nc <EW11 IP> 8899 | xxd | head
```

應該要看到 `00 ff ff 7e 07 … 7e 08`、`ff 7e 07 … 7e 08` 這種結構。如果只看到 `e0 1c fc 00`，代表鮑率還是 115200（EW11 出廠預設值）。

## 設定範例

```yaml
log_level: info
mqtt_server: 192.168.11.155        # 或填 core-mosquitto
mqtt_port: 1883
mqtt_qos: 1
mqtt_timeout: 5
mqtt_user: panda
mqtt_pass: "********"
taptap_topic_prefix: taptap
taptap_update: 10
taptap_timeout: 180
instances:
  - name: tigo                       # 第一台 CCA（沿用 tigo 這個名稱，原本的實體 ID 就不會變）
    address: 192.168.11.168
    port: 8899
    modules:
      - "A:P01:4-EFE783N"
      - "A:P02:4-EFE69CY"
      # ...
  - name: tigo2                      # 第二台 CCA
    address: 192.168.11.169
    port: 8899
    modules:
      - "C:P12:"
      - "C:P13:"
      # ...
ha_discovery_prefix: homeassistant
ha_birth_topic: homeassistant/status
ha_nodes_availability_online: true
ha_nodes_availability_identified: false
ha_strings_availability_online: true
ha_strings_availability_identified: false
ha_stats_availability_online: false
ha_stats_availability_identified: false
ha_nodes_sensors_recorder:
  - energy
ha_strings_sensors_recorder:
  - energy
ha_stats_sensors_recorder:
  - energy
```

### `instances` 各欄位說明

| 欄位 | 必填 | 說明 |
|---|---|---|
| `name` | ✔ | 只能用英數字和 `_`，不能重複。會成為 HA 裝置名稱和實體 ID 的開頭。 |
| `address` | 二擇一 | RS485 轉 Ethernet/WiFi 轉換器的 **IPv4 位址**。taptap-mqtt 不接受主機名稱。 |
| `serial` | 二擇一 | 如果是 USB RS485 轉接器，填裝置路徑（例如 `/dev/ttyUSB0`）。 |
| `port` | | 轉換器的 TCP 埠號，預設 502。 |
| `modules` | ✔ | 至少要有一筆。格式是 `字串:名稱:序號`：<br>• **字串**可以省略。同一個 instance 裡有兩個以上字串時，會產生每串的統計實體。<br>• **序號**可以先留空，例如 `":P01:"`。 |

### 序號對應

- 序號要等 TAP 廣播出來才看得到，多半在夜間，**最多可能要等 24 小時**。
- 偵測到還沒設定的序號時，Log 會顯示 `Discovered unconfigured node serial 4-XXXXXXX…`，並暫時把它分配到第一個空著的名稱。
- 建議照 Tigo App 或雲端 Layout 上每片面板的序號，把對應關係正確填好。

## 從原本的 TapTap 附加元件轉移

1. **先停用（或解除安裝）原本的 TapTap**，避免兩個附加元件同時發布到 `taptap/tigo`。
2. 第一台 CCA 的 instance 一樣命名為 **`tigo`**。實體的 unique_id 是由名稱計算出來的，所以原本的實體和歷史紀錄都會保留。
3. 把原本的 `taptap_modules` 清單搬到 `instances[0].modules`。

## 安裝

### 方法一：加入附加元件儲存庫（建議）

1. 設定 → 附加元件 → 附加元件商店 → 右上角 ⋮ → **儲存庫**，加入 `https://github.com/nooniansoong/hassio-addons`。
2. 在商店裡找到 **TapTap Multi** → 安裝。第一次安裝會在 HA 上從原始碼建置映像檔，大約需要幾分鐘。
3. 填好設定 → 啟動。建議打開「看門狗（Watchdog）」。

之後儲存庫有新版本時，HA 會直接顯示可以更新。

### 方法二：本地附加元件

1. 用 Samba 或「Advanced SSH & Web Terminal」，把整個 `taptap_multi` 資料夾複製到 HA 的 **`/addons/`** 底下。完成後路徑應該是 `/addons/taptap_multi/config.yaml`。
2. 設定 → 附加元件 → 附加元件商店 → 右上角 ⋮ → **檢查更新**。
3. 「本地附加元件」底下會出現 **TapTap Multi**。點進去 → 安裝，之後的步驟同方法一。

## 授權

Apache 2.0。原始附加元件與 taptap-mqtt 作者為 Dominik Strnad（litinoveweedle），taptap 協定研究者為 Will Glynn（willglynn）。
