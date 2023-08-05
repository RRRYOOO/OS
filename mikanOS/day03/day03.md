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
## 3.1.2 QEMUモニタを使ったメモリダンプ方法
- メインメモリの中で指定したアドレス付近の値を表示する、メモリダンプの方法を記載する。
- メモリダンプを実施するにはxコマンドを使う。Xコマンドの書式は以下の通り。
  ```
  (qemu) x /fmt addr
  ```
  - /fmtに指定された書式にしたがってaddrを先頭とするメモリ領域の値を表示する。  
    /fmtは、/[個数][フォーマット][サイズ]と分解できる。
  - [個数]は、何個分表示するかを指定する。
  - [フォーマット]は、読み出した値をどのような形式で表示するかを以下の種類から選択できる。
    | オプション | 表示形式 | 
    | ------------- | -------- | 
    | x | 16進数表示 |
    | d | 10進数表示 |
    | i | 機械語命令を逆アセンブルして表示 |
  - [サイズ]は、何バイトを1単位として解釈するかを指定する。
  
    | オプション | 単位 | 
    | ------------- | -------- | 
    | b | 1バイト |
    | h | 2バイト |
    | w | 4バイト |
    | g | 8バイト |
- 0x067ae4c4から4バイトを16進数で表示する場合は、以下のようにコマンドを実行する。
  ```
  (qemu) x /4xb 0x067ae4c4
  00000000067ae4c4: 0xeb 0xfe 0x66 0x90
  ```
  - 上記はおそらく機械語命令であると思われる。
    試しに2命令分を逆アセンブルして表示してみる。
    ```
    (qemu)bx /2i 0x067ae4c4
    00000000067ae4c4:  jmp      0x67ae4c4
    00000000067ae4c6:  xchg     %ax,%ax
    ```
      - 「jmp 0x67ae4c4」は、0x67ae4c4にジャンプするという命令であるが、0x67ae4c4はその命令がある場所そのものであるので、結局は同じ場所をぐるぐると回ることになる。実はこのアセンブリ命令はwhile(1);をコンパイルしたものになる。
## 3.2 レジスタ
- CPUに内蔵されたレジスタについて記載する。
- CPUには、一般的に汎用レジスタと特殊レジスタが搭載されている。
- 汎用レジスタは、一般の演算に使用するレジスタである。
- 特殊レジスタは、その目的は様々で、CPUの設定を行うためのものやタイマなどのCPUに内蔵された機能を制御するためのものもある。
### 3.2.1 汎用レジスタ
- 汎用レジスタの主目的は、値を記憶することである。
- CPUの外部にあるメインメモリと対照的に、レジスタは容量が小さく読み書きが高速な点が特徴である。  
  容量については、メインメモリは16GB（2の34乗バイト）程度あるのに対して、x86-64アーキテクチャの汎用レジスタは128B（2の7乗）しかない。  
  読み書き速度については、メインメモリはだいたい100ナノ秒くらいかかるのに対して、レジスタは待ち時間なく書き込み可能である。（2GHzで動くCPUであれば0.5ナノ秒程度）  
- x86-64の汎用レジスタは以下の16個である。
  - RAX、RBX、RCX、RDX、RBP、RSI、RDI、RSP、R8～R15
- これらの汎用レジスタは、CPUの演算対象に指定できる。
  例えば、加算命令addには次のように2つのレジスタを指定できる。  
  ```
  add  rax,  rbx
  ; オペコード  オペランド1,  オペランド2
  ```
- 一般的にx86-64の演算命令は2つのオペランド(引数)を取り、左側が書き込み先、右側が読み込み元になる。
- x86-64の汎用レジスタのサイズはすべて8バイト(=64ビット)である。
- charやunit16_tといった8バイトより小さいサイズの型の変数（変数はメインメモリに配置される）をレジスタに読み出して使う場合には、汎用レジスタを小さなレジスタとしてアクセスできるような仕組みになっている。  
  例えば、AXレジスタはRAXレジスタの下位16ビットを表す名前になっていて、AXを読み書きすることでRAXの下位16ビットを読み書きできる。  
    ![Image 1](generalPurposeResister.png)
### 3.2.1 特殊レジスタ
- 特殊レジスタの役割はレジスタによってそれぞれで、汎用レジスタ同様に値を記憶するものもあれば、汎用レジスタにはない機能を持つものもいる。代表的なものを記載する。
- RIPは、次に実行する命令のメモリアドレスを保持していて、命令の実行に伴って変化する。  
  演算系の命令を実行した場合は、次の命令を指すように増えるだけだが、jmpやcallのような分岐命令の場合は、オペランドで指定されたアドレスがRIPに書き込まれる。
- RFLAGSは、様々なフラグを集めたレジスタで、各ビット毎に異なる役割を持つ。例えば、ビット0がキャリーフラグ(CF)、ビット6がゼロフラグ(ZF)である。  
  CFは加算がオーバーフローした場合に1になる。ZFは命令の実行結果が0になると1になる。  
  フラグレジスタは、演算系の命令やcmpなどのフラグレジスタに影響を与える命令の直後に、jzやcmovzなどのフラグレジスタの内容によって動作が変わる命令を配置する。
- CR0は、CPUの重要な設定を集めたレジスタである。  
  CR0のビット0(PE)に1を書き込むと、CPUは保護モードに遷移する。ビット31(PG)に1を書き込むとページングが有効になる。
## 3.2 初めてのカーネル（osbook_day03a）
- 今回は、ブートローダはUEFIアプリとして、カーネルはELFバイナリとして別々のファイルとして開発し、ブートローダからカーネルを呼び出す形式にする。
- 最初に作成するカーネルは、何もしないで永久ループするプログラムとする。
- ここで定義するKernelMain()がブートローダから呼び出される関数になるが、このような関数をエントリポイントと呼ぶ。
  ```
  extern "C" void KernelMain() {
    while(1) __asm__("hlt");
  }
  ```
  - 「extern "C"」は、C言語形式で関数を定義することを意味する。C++では、引数の個数や方が異なる同名の関数を定義できるように名前修飾（マングリング）が行われる。名前修飾とは、宣言した関数名を関数名に引数の情報が結合された名前に変換する仕組みのことである。  
    C言語のプログラムからC++で定義した関数を呼び出すには、名前修飾された関数名を使う必要があるが、難しい名前になるため非現実的である。そのため、extern "C"を関数定義の先頭に付けることで名前修飾を防ぎ、通常の関数名で呼び出し可能となる。
  - 「__asm__()」は、インラインアセンブラのための記法で、C言語プログラムの中にアセンブリ言語の命令を埋め込むときに使う。「__asm__("hlt")」は、hlt命令を埋め込むという使い方になる。
  - hlt命令は、CPUを停止させる命令で、CPUが省電力な状態になる。hltを実行せずに永久ループすると、CPUが100%の使用率になる。hltにより、CPUは省電力モードになり動作が止まるが、割り込みがあると動作が再開する。
- このソースコードからカーネルファイルを作るには以下のようにコンパイルとリンクを行う。
  ```
  cd $HOME/workspace/mikanos
  git checkout osbook_day03a
  cd kernel
  clang++ -O2 -Wall -g --target=x86_64-elf -ffreestanding -mno-red-zone -fno-exceptions -fno-rtti -std=c++17 -c main.cpp
  ld.lld --entry KernelMain -z norelro --image-base 0x100000 --static -o kernel.elf main.o
  ```
  - 1行目では、clang++コマンドでソースコードをコンパイルし、オブジェクトファイルを作る。
  - コンパイラに指定したオプションの意味は以下の通り。
    | オプション | 意味 | 
    | ------------- | -------- | 
    | -O2 | レベル2の最適化を行う。 |
    | -Wall | 警告をたくさん出す。 |
    | -g | デバッグ情報付きでコンパイルする。 |
    | --target=x86_64-elf | x86_64向けの機械語を生成する。出力ファイル形式はELFとする。 |
    | -ffreestanfing | フリースタンディング環境向けにコンパイルする。 |
    | -mno-red-zone | Red Zone機能を無効にする。 |
    | -fno-exceptions | C++の例外機能を使わない。 |
    | -fno-rtti | C++の動的型情報を使わない。 |
    | -srd=c++17 | C++のバージョンをC++17とする。 |
    | -c | コンパイルのみする。リンクはしない。 |
    - -ffreestandingがフリースタンディング環境向けにコンパイルを行うための指定。C++の動作環境は大きく2種類、ホスト環境（hosted environment）とフリースタンディング環境（freestanding environment）が規定されている。ホスト環境はOSの上で動くプログラムのための環境で、フリースタンディング環境はOSがない環境のことである。OS開発の場合は、フリースタンディング環境をセレクトする必要がある。
    - -mno-red-zone、-fno-exceprtions、-dno-rttiはOSを作るときにはとりあえずつけておくといいオプション群である。
  - 次にリンカld.lldによってオブジェクトファイルから実行可能ファイルを作る。
  - リンカに指定したオプションは以下の通り。
    | オプション | 意味 | 
    | ------------- | -------- | 
    | --entry KernelMain | KernelMain()をエントリポイントとする。 |
    | -z norelro | リロケーション情報を読み込み専用にする機能を使わない。 |
    | --image-base 0x100000 | 出力されたバイナリのベースアドレスを0x100000番地とする。 |
    | -o kernel.elf | 出力ファイル名をkernel.elfとする。 |
    | --static | 静的リンクを行う。 |
- このカーネルファイルをブートローダから起動させるには、ブートローダがこのファイルをメインメモリに読み出す必要がある。  
  今回は、ブートローダの実行ファイルLoader.efiとカーネルの実行ファイルKernel.elfを両方ともUSBメモリに書き込んでおき、UEFIの機能を使ってブートローダからカーネルを読み出して起動させる。
- ブートローダでカーネルファイルを読み込む流れは、ファイルを開き、ファイル全体を格納できる十分なメモリを確保し、ファイルの内容を読み取る手順になっている。
  ```
  EFI_FILE_PROTOCOL* kernel_file;
  root_dir->Open(
      root_dir, &kernel_file, L"\\kernel.elf",
      EFI_FILE_MODE_READ, 0);
  
  UINTN file_info_size = sizeof(EFI_FILE_INFO) + sizeof(CHAR16) * 12;
  UINT8 file_info_buffer[file_info_size];
  kernel_file->GetInfo(
      kernel_file, &gEfiFileInfoGuid,
      &file_info_size, file_info_buffer);
  
  EFI_FILE_INFO* file_info = (EFI_FILE_INFO*)file_info_buffer;
  UINTN kernel_file_size = file_info->FileSize;
  
  EFI_PHYSICAL_ADDRESS kernel_base_addr = 0x100000;
  gBS->AllocatePages(
      AllocateAddress, EfiLoaderData,
      (kernel_file_size + 0xfff) / 0x1000, &kernel_base_addr);
  kernel_file->Read(kernel_file, &kernel_file_size, (VOID*)kernel_base_addr);
  Print(L"Kernel: 0x%0lx (%lu bytes)\n", kernel_base_addr, kernel_file_size);
  ```
- カーネルファイルを読み込む処理は、メモリマップを書き込むファイルを開くのと同様である。
- カーネルファイル全体を読むこむためのメモリを確保する。ファイルのサイズを知る必要があるので、kernel_file->GetInfo()を使ってカーネルファイルのファイル情報を取得する。
- この関数の第4引数にはEFI_INFO型を十分確保できる大きさのメモリ領域を指定する必要がある。ここでは、EFI_INFO型の構造体のサイズに加えてファイル名の文字数+NULL文字のサイズを足しこんだサイズを渡す必要がある。
  ![Image 1](EFI_FILE_INFO.png)
- kernel-file->GetInfo()が完了すると、file_info_bufferにはEFI_FILE_INFO型のデータが書かれた状態になる。file_info_bufferをEFI_FILE_INFO型にキャストすると、構造体の各メンバを取得できるようになる。
- 

## その他
### edk2でbuildが実行できなくなった場合
- edk2でbuildが実行できなくなった場合は、一度以下のコマンド実行してからbuildする。
  ```
  source edksetup.sh
  ```
### edk2でbuildで実行ファイルが生成されない場合
- EDK2付属のbuildコマンドでビルドを試みても、$HOME/edk2/Build/MikanLoaderX64/DEBUG_CLANG38/X64/Loader.efiが生成されないトラブルが発生した。buildコマンド実行後のターミナルの表示は以下のようになっていた。
  ```
  $ build
  Build environment: Linux-5.11.0-34-generic-x86_64-with-glibc2.29
  Build start time: 16:07:18, Sep.13 2021
  
  WORKSPACE        = /home/mikan/edk2
  EDK_TOOLS_PATH   = /home/mikan/edk2/BaseTools
  CONF_PATH        = /home/mikan/edk2/Conf
  PYTHON_COMMAND   = /usr/bin/python3.8
  
  
  Architecture(s)  = X64
  Build target     = DEBUG
  Toolchain        = CLANG38
  
  Active Platform          = /home/mikan/edk2/MikanLoaderPkg/MikanLoaderPkg.dsc
  
  Processing meta-data . done!
  Building ... /home/mikan/edk2/MdePkg/Library/UefiApplicationEntryPoint/UefiApplicationEntryPoint.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/UefiLib/UefiLib.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/UefiDevicePathLib/UefiDevicePathLib.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/UefiRuntimeServicesTableLib/UefiRuntimeServicesTableLib.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/BasePrintLib/BasePrintLib.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/UefiMemoryAllocationLib/UefiMemoryAllocationLib.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/UefiBootServicesTableLib/UefiBootServicesTableLib.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/BaseDebugLibNull/BaseDebugLibNull.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/BaseLib/BaseLib.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/BasePcdLibNull/BasePcdLibNull.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MdePkg/Library/BaseMemoryLib/BaseMemoryLib.inf [X64]
  make: Nothing to be done for 'tbuild'.
  Building ... /home/mikan/edk2/MikanLoaderPkg/Loader.inf [X64]
  make: Nothing to be done for 'tbuild'.
  
  - Done -
  Build end time: 16:07:20, Sep.13 2021
  Build total time: 00:00:01
  ```
- 前回ビルドを実施したコードから変更がない場合、上記のようにビルドが実施されない模様。
- コードを一部変更して再ビルドを実施すると正常にビルドが完了した。
  
### gitにファイルをプッシュしようとしたらエラーになる場合
- osbookday03aのファイルをプッシュしようとしたら以下のようなエラーが発生した。
```
$ git push origin my-changes
・・・
remote: error: File disk.img is 200.00 MB; this exceeds GitHub's file size limit of 100.00 MB
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
・・・
```
  - エラー発生の原因は、100MBを超えるファイルをプッシュしようとしていること。（Githubには100MBを超えるファイルはアップロードできない。）
  
#### 解決手順
1. 100MBを超えるファイルを特定する
    - 以下のコマンドを打って、100MBを超えるファイルを特定する。
    ```
    find . -size +100M -ls
    ```
2. 100MBを超えるファイルのコミットを取り消す
   - 直前のコミットであれば、以下のコマンドで取り消せる。
    ```
    git reset --hard HEAD
    ```
    - 直前でない場合は、コミット履歴を調べて100MBを超えるファイルをコミットしたリビジョンの1個前まで戻す。  
      （100MBのファイルがまだコミットされていない状態に戻す。）
      - コミット履歴を調べる。
      ```
      git log
    
      ～～コミット履歴がズラッと出るので、該当リビジョンを確認～～
      commit 13ec04b062a0cd9bb1a20e6a0b921ec7cf7396c0
      ↑これが該当リビジョンの1個前
      ```
      - 100MBを超えるファイルをコミットしたリビジョンの1個前まで戻す。
      ```
      git reset --soft 13ec04b062a0cd9bb1a20e6a0b921ec7cf7396c0
      ``` 
3. 100MBを超えるファイルを除外する
   - 100MBを越えているファイルを、コミット対象から除外する。複数ある場合は、すべてに対して行う。
   ```
   git reset (100MBを超えるファイルのパス)
   ```
4. コミット&プッシュ
   - あとはいつもの通りにコミット、プッシュを行う。
  
#### 参考文献
- [GitHubに100MB超えのファイルをプッシュしてエラーになった](https://qiita.com/kohei_wd/items/cba2437174350b63ad5a)
- [【Git】100MB越えのファイルをプッシュしようとしたらエラーになる件の解決方法](https://ios-docs.dev/100mb/)
