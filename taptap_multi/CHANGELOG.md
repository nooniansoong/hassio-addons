# 更新紀錄

## 0.3.4-multi.8（2026-10-16）

- **自動使用 Mosquitto**：`mqtt_server` 留空時，透過 Supervisor 的 MQTT 服務自動取得 Mosquitto 的主機、連接埠與專用內部帳密，設定中不需再填密碼（也就不會觸發「不安全的密鑰」修復警告）
- `mqtt_server`、`mqtt_user`、`mqtt_pass` 改為選填；只有連外部 broker 時才需要
- 升級後若要改用自動模式，請手動清空這三個欄位

## 0.3.4-multi.7（2026-10-15）

- **修正功率計算**：TS4 回報的是面板端的**輸入電流**（taptap 協定文件標為 "Current in"），先前被當成輸出電流，功率以 Vout × Iin 計算。優化器降壓時（遮蔭、組串不匹配）功率與每日發電量會少算，少算比例約等於 duty cycle。
  - `power` 改為 Vin × Iin（兩者皆為量測值），`energy` 每日發電量也由此積分
  - `current_in` 改為回報的量測電流
  - `current_out` 改為推算值 Iin ÷ duty（duty 過小時改用 Vin × Iin ÷ Vout）
- **新增 `power_out`**：推算的輸出功率 Vout × Iout（含優化器損耗），面板、組串與總計皆有，方便與逆變器量測值比對
- DOCS 新增「功率怎麼算」說明

## 0.3.4-multi.6（2026-10-07）

- 尚未收到 CCA 節點表前，依到達順序暫時猜測節點名稱（並記錄警告），避免沒有節點表的安裝完全沒有面板資料；收到節點表後，不在表中的節點改為忽略

## 0.3.4-multi.5（2026-10-07）

- 有設定序號的面板不再依到達順序猜測節點名稱，修正面板資料互換（例如 5F 與 6F 對調）的問題
- 重新列舉時移除過時的節點 ID 對應

## 0.3.4-multi.4（2026-10-05）

- 每日發電量在附加元件重啟、HA 重開或 MQTT 重連後不再歸零：每 60 秒及關閉時儲存積分值到 `/data/taptap_<name>_energy.json`，啟動時還原

## 0.3.4-multi.3（2026-10-05）

- 新增 `ha_device_per_node`（預設開啟）：每顆優化器、每個組串各自一個 HA 裝置，可分別指定區域；實體 ID 不變
- 修正 "discovered nodes" 日誌訊息

## 0.3.4-multi.2（2026-10-04）

- 改以上游預先建置的映像檔（ghcr.io/litinoveweedle/hassio-addon-taptap:0.3.4）為基礎，安裝與更新時間大幅縮短

## 0.3.4-multi.1（2026-10-04）

- 首版：一個附加元件同時連接多台 Tigo CCA（每台各一顆 RS485 轉換器）
- `instances` 設定：name、address/serial、port、modules
- 每個實例獨立的 HA 裝置、MQTT 主題、狀態檔與日誌前綴
