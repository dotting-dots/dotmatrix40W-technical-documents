# dotmatrix40 Wireless (dotmatrix40W) 技術資料

dotmatrix40 Wireless (dotmatrix40W) のハードウェア設計データおよび技術仕様を公開するリポジトリです。

---

## 📁 ディレクトリ構成

```text
.
├── LICENSE          # ライセンス情報
├── README.md        # 本ドキュメント (機能概要・ハードウェア仕様・ファイル一覧)
├── pcb/             # 基板設計データ (回路図・図面・BOM・3Dモデルなど)
└── case/            # 筐体設計データ (3Dプリント用STLデータ)
```

---

## 📄 格納ファイル一覧・説明

### 1. 基板関連データ (`pcb/`)

| ファイル名 | 説明 | 形式 / 用途 |
| :--- | :--- | :--- |
| `SCH_dotmatrix40W.pdf` | 回路図 (Schematic) | 回路全体の接続・ピンアサイン確認用 |
| `PCB_dotmatrix40W.pdf` | 基板レイアウト図 (PCB Layout) | 各層の配線パターンおよびシルク・実装図 |
| `BOM_dotmatrix40W.csv` | 部品表 (Bill of Materials) | 実装部品の型番・定数・パッケージ一覧 (LCSCの部品番号付き)|
| `DXF_dotmatrix40W.dxf` | 2D外形図面 | 基板外形およびネジ穴・スイッチ位置などの2Dデータ |
| `3D_dotmatrix40W.step` | 基板3Dモデル | 電子部品が実装された基板の3D CADデータ |

### 2. 筐体・3Dプリントデータ (`case/`)

| ファイル名 | 部品名称 | 説明 |
| :--- | :--- | :--- |
| `dotmatrix40_Wireless_frontbody.stl` | フロントボディ | キーボード上面およびスイッチ周りのメインフレーム |
| `dotmatrix40_Wireless_tiltbody_left.stl` | チルトボディ - 左 | 傾斜を持たせるボトムフレームの左側パーツ |
| `dotmatrix40_Wireless_tiltbody_right.stl` | チルトボディ - 右 | 傾斜を持たせるボトムフレームの右側パーツ |
| `dotmatrix40_Wireless_tiltbody_batterycase.stl` | 電池ケース | 単三電池ホルダー部分を保護・固定するパーツ |
| `dotmatrix40_knob_low_black.stl` | つまみキャップ | エンコーダ用ノブ(ロープロファイル) |
| `common_knob_cover_low_rev1.stl` | つまみカバー | エンコーダ用カバー(ロープロファイル) |
| `common_Filament_LED_cage.stl` | フィラメントLED保護ケージ | フィラメントLEDを保護するパーツ |

---

## 1. 全体概要

**dotmatrix40 Wireless (以下dotmatrix40W)** は、40%サイズ・46キーを備えたコンパクトなワイヤレスカスタムキーボードです。
Nordic社製のBluetooth SoC (nRF52840) を搭載し、BLE無線動作、ロープロファイルキースイッチ、ロータリーエンコーダ、フィラメントLEDといったハードウェア機能を備えています。

---

## 2. ハードウェア機能

### 2.1 メインコントローラー (MCU)
* **SoC:** Nordic nRF52840 (Raytac製 MDBT50Q-1MV2 モジュール)
* **特徴:** ARM Cortex-M4Fコアを採用し、Bluetooth 5 (BLE) 通信をネイティブサポート
* **クロックソース (LFCLK):** モジュール外付けとして XL1 (P0.00) / XL2 (P0.01) ピンに 32.768kHz のクリスタル発振器 (LFXTAL) を接続

### 2.2 キースイッチとレイアウト
* **キー数:** 46キー (4行 × 12列の変則オーソリニアベース)
* **対応スイッチ:** Kailh Choc V2 (ロープロファイルスイッチ・CPG135001S30 互換)
* **特徴:** 最下段 (Row 3) は親指での操作を考慮したレイアウト。1.5Uサイズのキー (SpaceやEnterなど) や中央部にエンコーダーを配置するスペースを確保

### 2.3 入力デバイス
* **ロータリーエンコーダ:** ALPS製 EC12E (EC12E2440301) 1基
* **用途:** スクロールや音量調整、LED輝度調整など回転動作による直感的なアナログインクリメンタル入力 (1回転あたり24パルス)

### 2.4 ライティングおよびインジケータ
* **PWMバックライト:** フィラメントLED (LED_filament_38mm) を搭載、PWM信号による滑らかな調光が可能
* **ステータスLED:** GPIO P0.26 に接続された状態通知用のLED (Blue) を搭載。主にBluetooth接続状態の表示に使用

### 2.5 電源・バッテリー回路
* **バッテリー駆動:** 単三電池用ホルダー (BAT-SMD_KEYSTONE-53) 搭載 (単3形電池 1本)
* **昇圧・電源管理:** MCP1640 (Boostコンバータ) を搭載し電池電圧を安定化・昇圧してシステムに供給。SSP7615 (LDOレギュレータ) と組み合わせて適切なシステム電圧を生成
* **内部レギュレータ (nRF52840):** REG0は VDDH と VDD を直結しているため非動作。REG1は DC/DCモードで動作
* **USB給電・保護:** USB Type-C端子搭載。USBLC6-2SC6 (ESD保護) や TPS25200 (eFuse/ロードスイッチ) によりUSB接続時の過電流や静電気から回路を保護
* **バッテリー電圧監視:**
  * **計測制御:** P0.03 出力がHighの場合にバッテリーレベルが P0.02 に印加 (Lowの場合はHi-Z状態)
  * **ADC入力:** P0.02 への入力は分圧なし (10kΩ経由)、ADC Vrefは3.3V
  * **残量計算:** ZMK上ではエネループ等のニッケル水素電池に配慮し V_max(100%)=1.35V、V_min(0%)=1.00V として設定
  * **保護機能:** 0.90Vを下回った際の低電圧シャットダウン機能 (過放電防止) を実装

---

## 3. ソフトウェア機能 (ZMK Firmwareベース)

### 3.1 マトリックス設定
* **スキャン方式:** `col2row` (ダイオードの向き: 列から行)
* **マトリックスサイズ:** 4行 × 12列 (最大48論理キーのうち物理実装は46キー)
* **GPIOアサイン:**
  * **Rows (4ピン):** P0.04, P0.05, P0.06, P0.07 (アクティブハイ・プルダウン設定)
  * **Cols (12ピン):** P0.25, P0.24, P1.00, P0.22, P0.23, P0.20, P0.21, P0.19, P0.17, P0.16, P0.13, P0.14

### 3.2 ペリフェラルとライティングの割り当て
* **ロータリーエンコーダ:** P0.08 (A相), P1.08 (B相) を使用 (内蔵プルアップ有効)
* **フィラメントLED (PWM):** PWM0 ペリフェラルを使用、P0.09 ピンからPWM信号を出力

### 3.3 キーマップとレイヤー構成 (デフォルト)
1. **Default Layer (レイヤー 0)**
   * 一般的なQWERTY配列ベースの入力レイヤー
   * **エンコーダ:** 上下スクロール (UP / DOWN)
2. **First Layer (レイヤー 1)**
   * ファンクションキー (F1〜F12)、テンキー、特殊記号を配置
   * **エンコーダ:** 左右スクロール / カーソル移動 (RIGHT / LEFT)
3. **Second Layer (レイヤー 2)**
   * Bluetooth接続切り替え・リセット (`BT_CLR`) 等のシステム管理レイヤー
   * **エンコーダ:** バックライト輝度調整 (`BL_INC` / `BL_DEC`)

### 3.4 省電力機能
乾電池駆動を前提とした省電力設定を実装しています。

* **ディープスリープ (`CONFIG_ZMK_SLEEP=y`)**
  * 無操作から約15分30秒後にディープスリープへ移行
  * Bluetooth接続を切断し、超低消費電力モードで待機
  * **復帰方法:** 任意のキーを押下 (復帰後の最初の1打はホストへ送出されません)
* **Bluetooth送信出力削減 (`CONFIG_BT_CTLR_TX_PWR_MINUS_4=y`)**
  * BLE無線の送信出力を標準 (0 dBm) から -4 dBm に低減

---

## 4. 開発者向け特記事項 (新規FW開発・他FW移行時)

1. **ピンアサインの互換性:** `dotmatrix40W.dts` に記載されたGPIOピン番号 (nRF52のポート・ピン) を引き継いでください。Port 1のピン (P1.00, P1.08 等) の指定方法に注意してください。
2. **電源制御ピンの初期化:** `EXT_POWER` や固定レギュレータ (`extra_power`: P0.03 等) などの電源管理ピンを起動直後の初期化ルーチンで適切にアサート (High/Low) する必要があります。

---

## 🔗 参考リンク

* **ZMK Firmware:** [dotting-dots/zmk-config-dotmatrix40W](https://github.com/dotting-dots/zmk-config-dotmatrix40W)
* **ビルドガイド・取扱説明書:** [dotmatrix40 Wireless documents](https://valuable-lancer-1ce.notion.site/dotmatrix40-Wireless-documents-2ce71a70e30a802caf0ef0f63cc6e76c)
* **入手先:** [dotmatrix40 Wireless \| dotting dots (STORES)](https://dotting-dots.stores.jp/items/69be90566d097bce89c039fa)
