# day01
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
- 
  

