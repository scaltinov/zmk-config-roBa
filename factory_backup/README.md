# Factory Firmware Backup

購入直後(出荷時)に XIAO-SENSE のブートローダーモードからダンプした `CURRENT.UF2`。

## 内容

- `roBa_L_factory.uf2` — 左側 (Seeed XIAO nRF52840 Sense, 1908736 bytes)
- `roBa_R_factory.uf2` — 右側 (同上, 1908736 bytes)
- `roBa_L_INFO_UF2.txt` / `roBa_R_INFO_UF2.txt` — ブートローダー情報

## 注意

サイズ 1.9MB = flash 全体ダンプ(SoftDevice + Bootloader + Application + 空領域)。
自前ビルドの ZMK uf2 (350-525KB, Application のみ) と単純比較不可。

## 巻き戻し手順

1. リセットボタン 2回連打で `XIAO-SENSE` ドライブをマウント
2. 該当の factory.uf2 を `/Volumes/XIAO-SENSE/` にコピー
3. 自動再起動

## ダンプ日

2026-05-22
