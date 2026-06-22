# zmk-config-LalaPadGen2

LaLaPad Gen2 向け ZMK カスタムファームウェア設定。

---

## ハプティックフィードバック Mod

右トラックパッド下に DRV2605L + LRA を搭載し、ドラッグ開始・解除にクリック感を付与する。

### BOM

| 部品 | 型番 / 仕様 | 数量 | 備考 |
|---|---|---|---|
| LRA（横振動モーター） | JYLRA061228XH | 1 | 12×6×2.85mm、X軸、200Hz、定格 2.0V |
| ハプティックドライバ | DRV2605L | 1 | I2C 制御、アドレス 0x5A |
| 配線 | 細線（AWG30 相当） | 適量 | ケーブル引き回し穴（3×4mm）経由 |

### 配線

DRV2605L と Seeeduino XIAO BLE の接続：

| DRV2605L | XIAO BLE | 備考 |
|---|---|---|
| VDD | 3V3 | |
| GND | GND | |
| SDA | D4 (P0.26) | IQS9151 と I2C バス共有 |
| SCL | D5 (P0.27) | IQS9151 と I2C バス共有 |
| IN | GND | I2C モード使用のため不要 |
| OUT+ | LRA + | |
| OUT− | LRA − | |

IQS9151 IRQ ピン: XIAO D6 (P1.11)

### LRA ポケット寸法（3D プリント）

| 項目 | 値 |
|---|---|
| 内寸（L×W×D） | 12.2 × 6.2 × 2.8 mm |
| 底面厚 | 1.2 mm |
| ケーブル引き回し穴 | 3 × 4 mm |

各辺 0.1mm クリアランス（LRA 公称 12×6×2.85mm に対し深さ方向は 2.8mm で圧入気味）。

### 組み立てノート

- **ケース印刷**: リポジトリの `LRA_case.stl` を 3D プリントして使用する（推奨素材: PETG）
- **固定方法**: LRA ポケットの底面にカプトンテープを貼り、その上に両面テープで LRA を固定する
- **DRV2605L モジュール加工**: 基板上の FFC ケーブルコネクタは実装不要のため取り外してから使用すること

### ファームウェア設定

`lalapadgen2_right.overlay` の DRV2605L ノード：

```dts
drv2605: drv2605@5a {
    compatible = "ti,drv2605";
    reg = <0x5a>;
    actuator-type = "lra";
    rated-voltage-mv = <2000>;
    overdrive-voltage-mv = <2800>;
    library = <6>;
    status = "okay";
};
```

`lalapadgen2_right.conf` の主要ハプティック設定：

```conf
CONFIG_DRV2605=y
CONFIG_INPUT_IQS9151_HAPTIC_FEEDBACK_ENABLE=y
CONFIG_INPUT_IQS9151_HAPTIC_LONG_PRESS_EFFECT=6      # ドラッグ開始時
CONFIG_INPUT_IQS9151_HAPTIC_DRAG_LOCK_RELEASE_EFFECT=6  # ドラッグロック解除時
```

エフェクト番号は library 6（LRA）の 1〜117 から選択可能。`overdrive-voltage-mv` は XIAO BLE の 3.3V レール上限を超えないよう 3000mV 以下に抑えること。
