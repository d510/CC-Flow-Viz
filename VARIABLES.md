# cc_flow_viz 変数・定数定義書

作成日: 2026-03-21

---

## 定数（CONFIG）

コード内で `const CONFIG = { ... }` として一箇所にまとめる。

### シミュレーション制御

| 定数名 | 値 | 単位 | 説明 |
|---|---|---|---|
| `SIM_START_HOUR` | 9 | 時 | シミュレーション開始時刻 |
| `SIM_END_HOUR` | 18 | 時 | 新規入電受付終了時刻 |
| `REAL_SEC_PER_SIM_HOUR` | 22.2 | 秒 | 実時間1秒あたりのシミュレーション時間（9時間÷200秒×60≒2.7分/秒） |
| `FRAME_RATE` | 60 | fps | p5.js の描画フレームレート |
| `RANDOM_SEED` | 42 | - | 乱数シード（固定） |
| `SIMULATION_SPEED` | 1 | 倍率 | 再生速度倍率（将来の拡張用） |

### 入電・キュー

| 定数名 | 値 | 単位 | 説明 |
|---|---|---|---|
| `TOTAL_CALLS_PER_DAY` | 300 | 件 | 1日の総入電数（再入電を除くユニーク入電） |
| `MAX_QUEUE_LENGTH` | 30 | 名 | キュー最大長。満杯時は即放棄 |
| `MAX_WAIT_MINUTES` | 3 | 分（シミュ） | これを超えると放棄 |
| `HOURLY_RATES` | 下記参照 | - | 時間帯別入電割合 |

```javascript
HOURLY_RATES: {
  9:  0.250,   // 09:00〜10:00
  10: 0.150,   // 10:00〜11:00
  11: 0.120,   // 11:00〜12:00
  12: 0.140,   // 12:00〜13:00
  13: 0.110,   // 13:00〜14:00
  14: 0.080,   // 14:00〜15:00
  15: 0.060,   // 15:00〜16:00
  16: 0.050,   // 16:00〜17:00
  17: 0.040,   // 17:00〜18:00
}
```

### 再入電

| 定数名 | 値 | 単位 | 説明 |
|---|---|---|---|
| `RETRY_RATE` | 0.30 | - | 放棄者のうち再入電する割合 |
| `RETRY_DELAY_MIN` | 5 | 分（シミュ） | 再入電までの最短待ち時間 |
| `RETRY_DELAY_MAX` | 15 | 分（シミュ） | 再入電までの最長待ち時間 |

### オペレータ

| 定数名 | 値 | 単位 | 説明 |
|---|---|---|---|
| `NEW_COUNT` | 10 | 名 | 新人オペレータ数 |
| `VET_COUNT` | 20 | 名 | ベテランオペレータ数 |
| `NEW_WAGE` | 1500 | 円/時 | 新人時給 |
| `VET_WAGE` | 2000 | 円/時 | ベテラン時給 |
| `NEW_CPH_MIN` | 2 | 件/時 | 新人CPH下限 |
| `NEW_CPH_MAX` | 4 | 件/時 | 新人CPH上限 |
| `VET_CPH_MIN` | 5 | 件/時 | ベテランCPH下限 |
| `VET_CPH_MAX` | 7 | 件/時 | ベテランCPH上限 |
| `NEW_BREAK_MINUTES` | 30 | 分（シミュ） | 新人休憩時間（全員9:00〜18:00勤務、休憩時間のみ異なる） |
| `VET_BREAK_MINUTES` | 60 | 分（シミュ） | ベテラン休憩時間（全員9:00〜18:00勤務、休憩時間のみ異なる） |
| `MAX_SIMULTANEOUS_BREAK` | 6 | 名 | 同時休憩最大人数 |
| `BREAK_START_HOUR` | 12 | 時 | 休憩開始時刻 |
| `BREAK_END_HOUR` | 13 | 時 | 休憩終了時刻 |

### 表示

| 定数名 | 値 | 単位 | 説明 |
|---|---|---|---|
| `DOT_SIZE` | 8 | px | ドットの一辺サイズ |
| `CANVAS_WIDTH` | 1000 | px | キャンバス幅 |
| `CANVAS_HEIGHT` | 600 | px | キャンバス高さ |
| `OPERATOR_COLS` | 6 | 列 | オペレータグリッドの列数 |
| `OPERATOR_ROWS` | 5 | 行 | オペレータグリッドの行数 |

---

## 変数（実行時の状態）

### シミュレーション全体

| 変数名 | 型 | 初期値 | 説明 |
|---|---|---|---|
| `simMinutes` | number | 0 | シミュレーション開始からの経過分（9:00=0, 18:00=540） |
| `isRunning` | boolean | true | 再生中フラグ |
| `isFinished` | boolean | false | シミュレーション終了フラグ |
| `nextCustomerId` | number | 0 | 次の顧客に割り当てるID（インクリメント） |
| `nextCallId` | number | 0 | 次のコール（ドット）に割り当てるID（インクリメント） |

### メトリクス（累積）

| 変数名 | 型 | 初期値 | 説明 |
|---|---|---|---|
| `totalCalls` | number | 0 | 総入電数（再入電含む） |
| `totalAnswered` | number | 0 | 受電数（対応完了） |
| `totalAbandoned` | number | 0 | 放棄数 |
| `totalRetry` | number | 0 | 再入電発生数 |
| `uniqueCustomers` | number | 0 | ユニーク顧客数 |
| `maxQueueLength` | number | 0 | 最大同時待ち人数（最大値を記録） |
| `totalWaitTime` | number | 0 | 放棄・受電した顧客の待ち時間合計（平均計算用） |
| `totalCost` | number | 0 | 累積人件費（円） |

### ドット（顧客）オブジェクト

配列 `dots[]` に格納。1要素が1コール（ドット）を表す。

| プロパティ | 型 | 説明 |
|---|---|---|
| `callId` | number | コールの一意ID |
| `customerId` | number | ユニーク顧客ID（再入電は元の顧客IDを引き継ぐ） |
| `isRetry` | boolean | 再入電フラグ（true=オレンジ表示） |
| `state` | string | 現在の状態（下記参照） |
| `spawnTime` | number | 出現時刻（シミュ分） |
| `queueEnterTime` | number | キュー入りした時刻（シミュ分） |
| `serveStartTime` | number | 対応開始時刻（シミュ分） |
| `assignedOperatorId` | number \| null | 対応オペレータID |
| `x` | number | 現在のX座標（px） |
| `y` | number | 現在のY座標（px） |
| `targetX` | number | 移動先X座標（px） |
| `targetY` | number | 移動先Y座標（px） |

**state の取りうる値**

| 値 | 説明 |
|---|---|
| `"INCOMING"` | 入電中（画面左から出現直後） |
| `"QUEUE"` | 待機中 |
| `"SERVING"` | 対応中 |
| `"DONE"` | 完了 |
| `"ABANDONED"` | 放棄 |

### オペレータオブジェクト

配列 `operators[]` に格納。

| プロパティ | 型 | 説明 |
|---|---|---|
| `id` | number | オペレータID（0〜29） |
| `type` | string | `"N"`（新人）or `"V"`（ベテラン） |
| `wage` | number | 時給（円） |
| `cph` | number | 個人CPH（初期化時にランダム決定・固定） |
| `serviceDuration` | number | 1件あたり対応時間（分）= 60 ÷ cph |
| `state` | string | `"idle"` / `"serving"` / `"break"` |
| `currentCallId` | number \| null | 対応中のコールID |
| `serveStartTime` | number \| null | 対応開始時刻（シミュ分） |
| `breakStartTime` | number \| null | 休憩開始時刻（シミュ分） |
| `breakDuration` | number | 休憩時間（シミュ分） |
| `totalWorkMinutes` | number | 累積稼働時間（分）※コスト計算用 |

---

## 状態遷移まとめ

```
ドット:
  INCOMING → QUEUE       （キューに空きあり）
  INCOMING → ABANDONED   （キュー満杯）
  QUEUE    → SERVING     （オペレータ空きあり）
  QUEUE    → ABANDONED   （MAX_WAIT_MINUTES超過）
  SERVING  → DONE        （serviceDuration経過）
  ABANDONED → INCOMING   （RETRY_RATE確率で、オレンジとして再出現）

オペレータ:
  idle  → serving  （QUEUE にドットあり）
  serving → idle   （serviceDuration経過）
  idle  → break    （休憩帯 かつ 同時休憩上限未満）
  break → idle     （breakDuration経過）
```
