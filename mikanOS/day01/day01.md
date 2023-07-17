# day01
## A.00 開発環境のインストール
- MikanOSをビルドしたり、起動したりする開発環境の設定は以下のサイトを参照。　　
　　https://github.com/uchan-nos/mikanos-build
## A.01 WSLのインストール
- WSL（Windows Subsystem for Linux）は、Windows上にLinux環境を構築するソフト。
- WSLのインストール方法は以下を参照。  
　　https://learn.microsoft.com/ja-jp/windows/wsl/install-manual
## A.02 WSLでQEMUを使う準備
- パソコンの代わりに、ソフトウェアで仮想的なパソコンを再現できる「QEMU」というエミュレータを使う方法。
- WSLでQEMUを使うには、Windows側にXサーバをインストールしておく必要がある。
- XサーバとはLinuxでGUIを扱う中心的な部品。LinuxでGUIを扱うには、伝統的にX Windows Systemという仕組みが使われる。この仕組みを使うと、GUIアプリが動くマシンとその画面を表示するマシンを別にすることができる。
- Windowsに対応したXサーバの実装はいくつかあるが、VcXsrvが有力。
- VcXsrvをのダウンロードは以下のサイトから。  
  　 https://sourceforge.net/projects/vcxsrv/
## B.00 MikanOSの入手
- MikanOSのソースコードは、以下のサイトから入手可能。  
  　　https://github.com/uchan-nos/mikanos.git
- $HOME/workspaceは以下にクローンを作成する。  
```
mkdir $HOME/workspace
cd  $HOME/workspace
git clone https://github.com/uchan-nos/mikanos.git
```
## 1.1 ハローワールド
- 

```
  $ ld
```

