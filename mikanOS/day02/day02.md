# day02
## 2.1 EDK2入門
- EDK2は、UEFI BIOS自体の開発にも、UEFI BIOS上で動くアプリケーションの開発にも使用できる開発キットである。
- EDK2は、開発環境のインストールが完了していれば、すでに$HOME/edk2にダウンロードされている。
- EDK2の主なファイル構成は以下である。
```
edk2/
  edksetup.sh       環境変数設定用スクリプト
  Conf/
    target.txt      ビルド設定
    tools_def.txt   ツールチェーンの設定
  MdePkg/           EDKの中心的ライブラリのパッケージディレクトリ
  ...Pkg/           その他のパッケージディレクトリ
```
- edksetup.shは、EDK2のビルドコマンドが動くようにするための準備スクリプトである。ビルドを行う前に実行する。
- Confディレクトリには、何をビルドするかを設定するtarget.txtとビルドに用いるコンパイラを設定するtools_def.txtを配置する。これらのファイルはedksetup.shを初めて実行したときにひな形が生成される。
- MdePkgは、他のプログラムからよく利用される基本ライブラリである。
- AppPkgは、UEFIアプリケーションをいくつか含んでいる。
- OvmfPkgじゃUEFI BIOSのオープンソース実装であるOVMFが収められている。
## 2.2 EDK2でハローワールド（osbook_day02q）
- 第1章で作成したC言語のハローワールドをEDK2ライブラリを使って書き直してみる。
- 今後、USBメモリからメインメモリにOSを読みこむのに使うブートローダに進化させるため、「MikanLoader」という名前でアプリケーションを作成していく。
- ソースコードはMikanOSのリポジトリのosbook_day02aタグである。ファイルの構成は以下。
  ```
  $HOME/workspace/mikanos/MikanLoaderPkg/
    MIkanLoaderPkg.dec    パッケージ宣言ファイル
    MIkanLoaderPkg.dsc    パッケージ記述ファイル
    Loader.inf            コンポーネント定義ファイル
    Main.c                ソースコード
  ```
#### <Loader.inf>
```
 [Defines]
   <中略>
   ENTRY_POINT          =  UefiMain
```
- 「Loader.inf」のENTRY_POINT設定には、このUEFIアプリケーションのエントリポイントを記載する。
- エントリポイントとは、起動時に最初に実行される関数のことである。通常のCプログラムではmain()という名前は固定だが、EDK2ではUEFIアプリケーションごとに自由にエントリポイント名を指定できる。
#### <Main.c>
```
#include  <Uefi.h>
#include  <Library/UefiLib.h>

EFI_STATUS EFIAPI UefiMain(
    EFI_HANDLE image_handle,
    EFI_SYSTEM_TABLE *system_table) {
  Print(L"Hello, Mikan World!\n");
  while (1);
  return EFI_SUCCESS;
}
```
- ハローワールドプログラムの本体はMain.cファイルである。
- 第1章のhello.cで書いたものの大部分はEDK2のライブラリが提供しているため、Main.cでは#includeでそれを取り込むだけでよくなったためシンプルになった。
- Uefi.hじゃEDK2にむくまれるヘッダファイルで、実体は$HOME/edk2/MdePkg/Include/Uefi.hにある。
- Print()関数は、C言語のprintf()関数と似た、文字列の表示関数である。printf()と違うのは引数にワイド文字を渡さなければならないところである。文字列の前にLと書いてるのが、これはワイド文字から構成された文字列を表現するためのやり方である。UEFIで文字表示するにはワイド文字にすると覚えておくこと。
- 上記のソースコードをビルドする手順を以下に示す。
  - MikanOSのリポジトリでビルドしたいバージョンを呼び出しておく。
    - git checkoutは、指定したバージョン(タグ)のソースコードを呼び出し、ファイルとして配置するコマンドである。 
    ```
    cd $HOME/workspace/mikanos
    git checkout osbook_day02a
    ```
  - MikanLoaderPkgにシンボリックリンクを張る。
    - $HOME/edk2の中に、$HOME/workspace/mikanos/MikanLoaderPkgを指すシンボリックリンクを作成する。
    ```
    cd $HOME/edk2
    ln -fs $HOME/workspace/mikanos/MikanLoaderPkg ./
    ```
  - edksetup.shを読み込む。
    - sourceコマンドでedksetup.shファイルを読み込むと、Conf/target.txtファイルが自動的に生成される。
    ```
    source edksetup.sh
    ```
  - 生成されたConf/target.txtを編集する。
    - Conf/target.txtでMikanLoaderPkgをビルド対象として指定する。以下の表のように設定する。
      | 設定項目 | 設定値 |
      ----|---- 
      | ACTIVE_PLATFORM | MikanLoaderPkg/MikanLoaderPkg.dsc |
      | TARGET | DEBUG |
      | TARGET_ARCH | X64 |
      | TOOL_CHAIN_TAG | CLANG38 |
  - 設定が完了したらEDK2付属のbuildコマンドでビルドする。
    - ビルドが完了すると、$HOME/edk2/Build/MikanLoaderX64/DEBUG_CLANG38/X64?loader.efiに目的のファイルが出力される。
      ```
      cd $HOME/edk2
      build
      ```
  - 出力されたファイルをBOOTX64.EFIにリネイムして以下のコマンドで実行する。
    - QEMU上に「Hello,Mikan World!」 と表示される。
    
    
