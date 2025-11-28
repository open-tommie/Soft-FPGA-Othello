# Soft-FPGA-Othello

![soft-fpga-logo](./images/soft_fpga_logo-70x50.png)

- 2025/11/28 __まだ作成中です。__ 動いてません。
- 2025/11/28 [Soft-FPGA-TD4](https://github.com/open-tommie/Soft-FPGA-TD4)からforkしました。

- ブレッドボード図（予定）

[![Sketch](./images/soft-FPGA-TD4-Sketch-198x200.png)](./images/soft-FPGA-TD4-Sketch-1815x1832.png)

- Verilogで実装したオセロゲームをverilatorでC++ソースへ変換し、`Raspberry Pi Pico2` 上で動作させる予定です。
- __FPGAは不要__ です。pico2のみで動きます。
- ハードウェアFPGAなしでverilogソースが動く！だから __`soft-FPGA`__ です。
- `soft-FPGA`は`Software Defined FPGA`の略です。`Software Defined Radio`からインスパイアされて私が作った造語です。

![pico2-01](./images/pico2-01.jpg)

---

## 開発環境

- 2025/11/28 開発環境は変更する予定です。
- Dockerでビルド、実行できるようにしたい。

---

## `soft-FPGA`ビルド手順

- git cloneでファイル取得
  - WSL2 Ubuntu24で実行する場合はWindowsからわかりやすいディレクトリ上で実行すると良い。
    - 例：WSL2 Ubuntu24の`/mnt/c/pico2`はWindowsの`C:\pico2\`
  
  ```text
  git clone https://github.com/open-tommie/Soft-FPGA-TD4.git
  ```

- Windows11: ファイルエクスプローラでクローンしたSoft-FPGA-TD4フォルダーを開き、`VSVodeワークスペースファイル`をダブルクリック

  ```text
  ./Soft-FPGA-TD4/verilator-TD4/02-verilater-TD4.code-workspace
  ```

- VSCode: 右下の`□ Compile`アイコンをクリックしてビルドする。成功すると以下のようなログを出力する。

  ```text
  [101/101] Linking CXX executable test.elf
  *  Terminal will be reused by tasks, press any key to close it. 
  ```

- VSCode: buildフォルダーに`test.uf2`が作成されたことを確認する。

  ![build-ok](./images/build-ok.png)

- UF2ファイルをpico2で実行する：
  - pico2の`BOOTSEL`ボタンを押したまま、USBケーブルでPCと接続します。
  - 接続後、`BOOTSEL`ボタンを離します。
    - PC側で、Pico 2が __マスストレージデバイス（RPI-RP2）__ として認識されます。
    - 認識されない場合: ケーブルを抜き、再び`BOOTSEL`を押しながら接続を試してください。
  - UF2ファイルをコピー
    - UF2ファイルを、RPI-RP2ドライブにドラッグアンドドロップします。
    - コピーが完了すると、Pico2が自動的にリブートします。

（TBD)

- リブートすると4秒後にLEDが点滅します
- TeraTermでログ出力を確認

---

## verilatorでverilogをC++変換

(TBD)

- `MSYS2`でシェルスクリプトを実行します。
  - [do-varilator.sh](./verilator-TD4/do-verilator.sh)
  - ./obj_dirにC++ソースを出力します。
  - main.cppは出力しない
- 実行例：

```bash
tommie@PC40-MSYS:/c/pico2/Soft-FPGA-TD4/verilator-TD4$ ./do-verilator.sh
#!/bin/bash -v
# Verilatorを使ってVerilogコードをC++に変換
#
# MSYS2環境で実行することを想定
# 実行方法: ./do-verilator.sh

verilator --no-timing --cc TD4.v --exe main.cpp --top-module TD4 --threads 1 --threads-dpi none -o VTD4_sim
- V e r i l a t i o n   R e p o r t: Verilator 5.040 2025-08-30 rev UNKNOWN.REV
- Verilator: Built from 0.000 MB sources in 0 modules, into 0.000 MB in 0 C++ files needing 0.000 MB
- Verilator: Walltime 0.002 s (elab=0.000, cvt=0.000, bld=0.000); cpu 0.000 s on 1 threads; alloced 5.172 MB
```

---

## Debug Probeで書き込み

(TBD)

- 書き込み
- ログの確認
  
---

## pico2接続

（TBD)

- [Raspberry Pi Pico2のピン配置](https://datasheets.raspberrypi.com/pico/Pico-2-Pinout.pdf)

- ログ解説

---

## 解説

- [verilator公式マニュアル(英文)](https://veripool.org/guide/latest/)
  - verilatorで変換したC++ソースはLinux向けになっています。
  - そのままではpico2 SDK C++ではコンパイルエラーがでます。
  - エラーがでないようにして、pico2で実行できるようにしました。

- TD4 verilogソース
  - [TD4.v](./verilator-TD4/TD4.v)
  - fork元のsimpleTD4を修正して使っています。
  - fork元の解説が充実しているので是非、参照して下さい。
    - [simpleTD4](https://github.com/asfdrwe/simpleTD4)
      - Verilogで実装したシンプルな4bit CPU TD4の実装。
      - [Sipeed Tang Nano](https://tangnano.sipeed.com/en/) で動作。
      - [TD4_details_jp.md](https://github.com/asfdrwe/simpleTD4/blob/master/TD4_details_jp.md) にTD4.vの解説があります。

- [main.cpp](./verilator-TD4/main.cpp)
  - main.cppの処理概要は以下の通りです。
    - pico2のGPIOを初期化
    - DIP SWの値をTD4:IN_PORTレジスタへ設定
    - TD4クロック実行
    - TD4:OUT_PORTをLEDへ出力
  - `top`はverilatorが出力するTD4.vのモデルインスタンス
    - verilogシミュレーション用のインスタンス
  - 以下を繰り返す
    - DIPスイッチの値をIN_PORTレジスタへ設定する
      - 4bit DIPスイッチの状態をpico2のGPIOから読む
      - GPIOの値をtop->in_port(IN_PORTレジスタ)へ設定する
    - top->eval(): 評価：TD4の状態を更新する
    - OUT_PORTレジスタの値をLEDへ出力する
      - top->out_port(OUT_PORTレジスタ)の値をpico2のGIPOでLEDへ出力する
    - top->clock = !top->clock: クロックを反転する
    - sleep: 時間まち
  - top->eval()の処理時間はデバッグ出力なしで6μ秒ぐらい
    - 逆算すると、TD4の最大クロック周波数は50kHzぐらいの見込み。
    - デバッグ文を出力すると遅くなる
    - GPIOで遅くなる。

---

## FAQ

### Q1-verilator出力のC++ソースを修正する必要あるか？

- A: __ありません。__
  - [詳細はこちら：FAQ-Q1](./etc/92-FAQ.md#q1-verilator出力のcソースを修正する必要はあるか)

---

## メモ、雑感

[メモ](./etc/91-memo.md)

---

![logo](./images/soft-fpga-monolith-logo.jpg)

---
