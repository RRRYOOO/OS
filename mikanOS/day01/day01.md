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
- MikanOSのソースコードには、各時点のバージョンにタグが付与されている。タグはosbook_dayXXの形式で名前付けされている。
- タグの一覧を確認するコマンドは以下。（タグ一覧から抜ける場合はqを押す）
```
cd $HOME/workspace/mikanos
git tag -l
```
- この中から希望する時点のソースコードを得ることができる。例えば、osbook_day26aの内容を得るためには次のコマンドを実行する。
```
git checkout osbook_day26a
```
## 1.1 ハローワールド
- バイナリで起動後に"Hallo, World!"が表示されるプログラムを作成する。
- バイナリファイルを作成するには、バイナリエディタを使用する。WSL上のUbuntuで行う場合は、Windows用のバイナリエディタを使う。
- Windows用のバイナリエディタはいくつかあるが、Binary Editor Bzが有力。
- Binary Editor Bzでコードを作成し、BOOTX64.EFIの名前で保存する。  
    https://github.com/RRRYOOO/OS/blob/main/mikanOS/day01/BOOTX64.efi
  ## 1.1 ハローワールド
