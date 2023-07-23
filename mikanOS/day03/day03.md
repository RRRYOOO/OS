# day03
## 3.1 QEMUモニタ
- QEMUモニタを使ったデバッグ方法を記載する。
- QEMUモニタは、QEMUの標準機能で、CPUの設定を表示したりメモリの中身を読み書きをすることができる。
- QEMUモニタはを使うには、QEMUが起動した後にrun_qemu.shを実行したターミナルに戻る。  
  （QEMUモニタをターミナルから使えるようにしてある。）
    ![Image 1](qemuMonitor1.png)
## 3.1.1 QEMUモニタを使ったレジスタ値の確認
- QEMUモニタで以下のコマンドを実行すると、CPUの各レジスタの現在の値が表示される。
  ```
  info registers
  ```
   ![Image 1](infoResisters.png)
  
