# day04
## 4.1 make入門（osbook_day04a）
- makeとは、コンパイルやリンクなどの作業を自動化するツールである。
- makeを構成するのは、makeコマンドと指示書であるMakefileである。
- 「第3章 画面表示の練習とブートローダ」で登場したカーネルのコンパイラとリンクをMakefileで表現すると以下のようになる。
    ```
    TARGET = kernel.elf
    OBJS = main.o
    
    CXXFLAGS += -O2 -Wall -g --target=x86_64-elf -ffreestanding -mno-red-zone \
          -fno-exceptions -fno-rtti -std=c++17
    LDFLAGS  += --entry KernelMain -z norelro --image-base 0x100000 --static
    
    
    .PHONY: all
    all: $(TARGET)
    
    .PHONY: clean
    clean:
          rm -rf *.O
    
    kernel.elf:$(OBJS) Makefile
          ld.lld $(LDFLAGS) -o kernel.elf $(OBJS)
    
    %.o: %.cpp Makefile
          clang++ $(CPPFLAGS) $(CXXFLAGS) -c $<
    ```
    - Makefileの前半が変数定義、後半がルールの定義になっている。変数は自由に定義して使用することができる。  
      上記のコードで使用している変数の意味は以下の通り。
        | 変数名 | 意味 | 
        | ------------- | -------- | 
        | TARGET | このMakefileが生成する最終成果物 |
        | OBJS | TARGETを作るのに必要なオブジェクトファイル群 |
        | CXXFLAGS | コンパイルオプション |
        | LDFLAGS | リンクオプション |
    - 変数定義の後にはルールが記載される。  
      ルールとは、ターゲットとその前提となる必須項目、必須項目からターゲットを生成するレシピ（コマンド列）を集めたもの。
    - 必須項目とは、ターゲットを作り出すために必要となるファイル群のこと。  
      レシピとは、必須項目からターゲットを作る実際の手順のこと。
    - 一つのルールは以下のような書き方で記載される。レシピの行は必ず先頭をタブ文字で始める必要がある。
      ```
      ターゲット:必須項目
          レシピ
      ```
- 上記のMakefileには、「all」「clean」「kernel.elf」「%.o」の4つのターゲットがある。  
  ターゲット名は、makeコマンドの引数に指定することができる。  
  make cleanを実行すると、main.oのファイルが削除される。  
  makeを引数なしで実行するとMakefileの最初に現れるターゲットであるallが指定されたのと同じ動きをする。
- ターゲットのうち、allとcleanは実際のファイル名ではなく、単にルールを表す名前として使っている。これらのターゲットのことを偽物のターゲット（phony target）と呼び、.PHONY宣言を行う。  
   .PHONY宣言は、allやcleanといったファイルが実在する場合に効果を発揮する。
- ルールは、次のように再帰的に処理が実行される。
    - すべての必須項目に対して、それぞれをターゲットとするルールを実行する。
    - その後、次の規則にしたがってレシピを処理する。
        - ターゲットが必須項目より新しい場合は、何もしない
        - ターゲットが実在しない or 必須項目より古い場合は、レシピを実行する
- allから始まって再帰的にルールが実行されるフローは以下の通り。
  ```
  all: kernel.elf
  -->必須項目（kernel.elf）のルール（kernel.elf: main.o Malefile）を実行。
      -->必須項目（main.o）のルール（main.0: main.cpp Malefile）を実行。
          -->必須項目（main.cpp）のルールを実行しようとするがないので何もしない。
          -->必須項目（Makefile）のルールを実行しようとするがないので何もしない。
          -->レシピを実行。clang++でmain.oを生成。
      -->必須項目（Makefile）のルールを実行しようとするがないので何もしない。
      -->レシピを実行。ld.lldでkernel.rlfを生成。
  -->レシピなし。何もしない。
  ```
- 何故かレシピがスキップされて必要なコマンドが実行されない場合は、以下のコマンドでファイルが新しいとか古いとかを無視してすべてのレシピを再実行させることができる。
  ```
  make -B
  ```
- Makefileの中の%.oや%.cppは、ファイル名のパターンである。パターンルールで統一して記載しておくと、OBJSに含まれるオブジェクトファイルが増えても記載を追加しなくて済む。%にあたる部分のことを「幹（stem）」という。
- レシピの中などで使用される特殊な変数がいくつかある。これらの変数はmakeが自動的に定義する変数である。主なものを以下に示す。
    | 変数名 | 説明 | 
    | ------------- | -------- |
    | \$< | 必須項目の先頭1つ |
    | \$^ | 必須項目すべてをスペース区切りで並べたもの |
    | \$@ | ターゲット（拡張子を含む） |
    | \$* | パターンルールにおける幹（stem） |
    - 「\$*」の実例を以下に示す。
        | ターゲットのパターン | 実際のターゲット | \$*の値 | 
        | ------------- | -------- | -------- | 
        | %.o | foo.o | foo |
        | a.%.b | dir/a.foo.b | dir/foo |
- %.oやkernel.elfのレシピ内でMakefileを使用していないが、必須項目にMakefileを指定している理由は、Makefileが更新された場合にはビルドをし直すべきだという考えがあるためである。Makefileを必須項目に登録しておけば、Makefileが更新された場合にレシピが再実行される。
## 4.2 ピクセルを自在に描く（osbook_day04b）
- 画面上の位置を指定して好きな色を描画できるような機能を開発する。
- ピクセル描画に必要な情報をまとめるためのFrameBufferConfig構造体の定義を以下に示す。
#### <frame_buffer_config.cpp（フレームバッファの構成情報を表す構造体）>
  ```
  #pragma once
    
  #include <stdint.h>
    
  enum PixelFormat {
    kPixelRGBResv8BitPerColor,
    kPixelBGRResv8BitPerColor,
  };
    
  struct FrameBufferConfig {
    uint8_t* frame_buffer;
    uint32_t pixels_per_scan_line;
    uint32_t horizontal_resolution;
    uint32_t vertical_resolution;
    enum PixelFormat pixel_format;
  };
  ```
  - FrameBufferConfig構造体は、以下の情報を保存する。
    | メンバ | 説明 | 
    | ------------- | -------- |
    | frame_buffer | フレームバッファ領域へのポインタ |
    | pixels_per_scan_line | フレームバッファの余白を含めた横方向のピクセル数 |
    | horizontal_resolution | 水平方向の解像度 |
    | vertical_resolution | 垂直方向の解像度 |
    | pixel_format | ピクセルのデータ形式 |
  - フレームバッファは3現職の光る強さを整数値で並べたものであり、フレームバッファにどういう色の順番でそれぞれ何ビットで並べるかということをpixel_formatが表している。
  - UEFIの規格ではピクセルのデータ形式は次の4種類がある。
  　- PixelRedGreenBlueReserved8BitPerColor
    - PixelBlueGreenRedReserved8BitPerColor
    - PixelBitMask
    - PixelBltOnly
  - 本プログラムでは、最初の2種類だけサポートすることとする。
  - ブートローダ側の変更は以下の通り。
#### <Main.c（ブートローダはOS本体に描画に必要な情報を渡す）>
```
struct FrameBufferConfig config = {
  (UINT8*)gop->Mode->FrameBufferBase,
  gop->Mode->Info->PixelsPerScanLine,
  gop->Mode->Info->HorizontalResolution,
  gop->Mode->Info->VerticalResolution,
  0
};
switch (gop->Mode->Info->PixelFormat) {
  case PixelRedGreenBlueReserved8BitPerColor:
    config.pixel_format = kPixelRGBResv8BitPerColor;
    break;
 case PixelBlueGreenRedReserved8BitPerColor:
    config.pixel_format = kPixelBGRResv8BitPerColor;
    break;
 default:
    Print(L"Unimplemented pixel format: %d\n", gop->Mode->Info->PixelFormat);
    Halt();
}

typedef void EntryPointType(const struct FrameBufferConfig*);
EntryPointType* entry_point = (EntryPointType*)entry_addr;
entry_point(&config);
```
 - UEFIのGOPから取得した情報を、先ほど作った構造体にコピーして、その構造体へのポインタをKernelMain()の引数に渡す。
-カーネル側の変更は以下の通り。
#### <main.cpp（WritePixel()を使って画面を描画する）>
```
extern "C" void KernelMain(const FrameBufferConfig& frame_buffer_config) {
  for (int x = 0; x < frame_buffer_config.horizontal_resolution; ++x) {
    for (int y = 0; y < frame_buffer_config.vertical_resolution; ++y) {
      WritePixel(frame_buffer_config, x, y, {255, 255, 255});
    }
  }
  for (int x = 0; x < 200; ++x) {
    for (int y = 0; y < 100; ++y) {
      WritePixel(frame_buffer_config, 100 + x, 100 + y, {0, 255, 0});
    }
  }
  while (1) __asm__("hlt");
}
```
 - 引数で構造体のポインタを参照（const FrameBufferConfig&）で受け取っている。参照型はC++固有の文法で、C言語にはない。参照型の引数を持つ関数をC言語から呼び出すには、参照の代わりにポインタを指定すればよい。  
   これはC++自体の仕様ではなく、 使用しているコンパイラの仕様であるSystem V AMD64 ABIで決まっている。
- 指定したピクセル座標描画する関数のWritePixel()の実装を以下に示す。
#### <main.cpp（WritePixel()）>
```
struct PixelColor {
  uint8_t r, g, b;
};

/** WritePixelは1つの点を描画します．
 * @retval 0   成功
 * @retval 非0 失敗
 */
int WritePixel(const FrameBufferConfig& config,
               int x, int y, const PixelColor& c) {
  const int pixel_position = config.pixels_per_scan_line * y + x;
  if (config.pixel_format == kPixelRGBResv8BitPerColor) {
    uint8_t* p = &config.frame_buffer[4 * pixel_position];
    p[0] = c.r;
    p[1] = c.g;
    p[2] = c.b;
  } else if (config.pixel_format == kPixelBGRResv8BitPerColor) {
    uint8_t* p = &config.frame_buffer[4 * pixel_position];
    p[0] = c.b;
    p[1] = c.g;
    p[2] = c.r;
  } else {
    return -1;
  }
  return 0;
}
```
 - WritePixel()は、指定したピクセル座標(xとy)に指定した色(c)を描画する関数である。ピクセルのデータ形式に基づいて、光の3原色をフレームバッファに書き込む。
 - pixel_positionには、ピクセルの座標をフレームバッファ先頭からの位置に変換した値が設定される。  
   以下の式から、座標をフレームバッファ先頭からの位置に変換する。
   - 座標をフレームバッファ先頭からの位置 = (余白を含めた横方向のピクセル数 * y) + x
 - ピクセル座標とフレームバッファ先頭からの位置の関係は以下の通り。

   　 ![Image 1](buffer_and_pixel_coordinate.png)
 - PixelRedGreenBlueReserved8BitPerColorとPixelBlueGreenRedReserved8BitPerColorは両方とも各色8ビットで表されるが、色の並びが異なる。  
   前者は赤→緑→青→予約領域、後者は青→緑→赤→予約領域となる。
 - 予約領域は8ビットで構成される。原色の3色だけでは24ビット(3バイト)と中途半端になるため、8ビットの予約領域を追加して32ビット(4バイト)にそろえている。
 - 1ピクセルは32ビットすなわち4バイトなので、あるピクセル座標の位置をフレームバッファ先頭からの計算するには、ピクセルとしての位置（pixel_position)に4を掛ける。
 - 上記のコードでは、まず画面全体を白（255,255,255）で塗りつぶした後に、200×100の緑色の四角形を描画している。  
   実行結果は以下の通り。
   
   　 ![Image 1](pixel_drawing_green.png)

## 4.3 C++の機能を使って書き直す（osbook_day04c）
- ここでは、C++の言語機能である仮想関数を使って、WritePixel()相当の機能を持ちつつ、関数の外側でピクセルのデータ形式を判定するように変更する。
- C++のクラスを使ってコードを変更する。クラスはデータとその操作をひとまとまりにしたもので、クラスのようにデータとそれを操作する手続きを合わせたものを抽象データ型という。抽象データ型は、外部のユーザからデータの中身の詳細を隠蔽し、外部に公開された手続きによる操作を強制することで、インターフェースと実装を分離できるという特徴がある。
- クラスを使うことで、ピクセルのデータ形式に依存しない「ピクセル描画インターフェース」と「ピクセルのデータ形式にしたがって実際に描画する実装」を分離する。
  #### <main.cpp（PixelWriterクラスの実装（親クラス））>
  ```
  class PixelWriter {
   public:
    PixelWriter(const FrameBufferConfig& config) : config_{config} {
    }
    virtual ~PixelWriter() = default;
    virtual void Write(int x, int y, const PixelColor& c) = 0;

   protected:
    uint8_t* PixelAt(int x, int y) {
      return config_.frame_buffer + 4 * (config_.pixels_per_scan_line * y + x);
    }

   private:
    const FrameBufferConfig& config_;
  };
  ```
  - 上記はインターフェースにあたる部分のソースコードで、PixelWriterクラスを1つ定義している。
  - このクラスはピクセルを描画するためのWriter()関数を含む。この関数のプロトタイプ宣言の後ろに"= 0"があるのは、この関数が純粋仮想関数であることを表している。純粋仮想関数は、実際の処理の内容は決まっていないけど、戻り値と引数の仕様、関数名は決まっている関数である。子クラスで処理の実装をオーバーライドすることもある。
  - 戻り値の型が何も書いていない、クラス名と同じ名前のPixelWriter()関数はコンストラクタと呼び、クラスのインスタンスをメモリ上に構築するために呼ばれるのがコンストラクタである。
  - 頭に"\~"がある~PixelWriter()関数はデストラクタと呼び、インスタンスを破棄する際に呼ばれる。
  - コンストラクタPixelWriter::PixelWriterはフレームバッファの構成情報を受け取り、クラスのメンバ変数config_にコピーする。構成情報をコンストラクタで受け取っておくことでPixelWriter::Write()を呼び出す際に構成情報を渡す必要がなくなる。
  #### <main.cpp（PixelWriterクラスを継承したクラス群（子クラス））>
  ```
  class RGBResv8BitPerColorPixelWriter : public PixelWriter {
   public:
    using PixelWriter::PixelWriter;

    virtual void Write(int x, int y, const PixelColor& c) override {
      auto p = PixelAt(x, y);
      p[0] = c.r;
      p[1] = c.g;
      p[2] = c.b;
    }
  };

  class BGRResv8BitPerColorPixelWriter : public PixelWriter {
   public:
    using PixelWriter::PixelWriter;

    virtual void Write(int x, int y, const PixelColor& c) override {
      auto p = PixelAt(x, y);
      p[0] = c.b;
      p[1] = c.g;
      p[2] = c.r;
    }
  };
  ```
  - 上記は、インターフェースに対する実装に相当する部分のソースコードである。
  - インターフェースであるPixelWrterクラスを継承して、2種類のピクセル形式それぞれに対応するRGBResv8BitPerColorPixelWriterクラスとBGRResv8BitPerColorPixelWriterクラスを定義している。
  - 継承とは、あるクラスを基にして機能の差分を書くやり方の1つで、今回は、基となるPixelWriterクラスのWrite()に実装を与えるために継承している。親クラスの関数を子クラスで上書きすることをオーバーライドと呼ぶ。
  - 2つのクラスにはコンストラクタの定義が見当たらないが、using宣言によって、親クラスのコンストラクタをそのまま子クラスのコンストラクタとして使うことができる。
  #### <main.cpp（PixelWriterクラスの使用）>
  ```
  char pixel_writer_buf[sizeof(RGBResv8BitPerColorPixelWriter)];
  PixelWriter* pixel_writer;
  
  extern "C" void KernelMain(const FrameBufferConfig& frame_buffer_config) {
    switch (frame_buffer_config.pixel_format) {
      case kPixelRGBResv8BitPerColor:
        pixel_writer = new(pixel_writer_buf)
          RGBResv8BitPerColorPixelWriter{frame_buffer_config};
        break;
      case kPixelBGRResv8BitPerColor:
        pixel_writer = new(pixel_writer_buf)
          BGRResv8BitPerColorPixelWriter{frame_buffer_config};
        break;
    }

    for (int x = 0; x < frame_buffer_config.horizontal_resolution; ++x) {
      for (int y = 0; y < frame_buffer_config.vertical_resolution; ++y) {
        pixel_writer->Write(x, y, {255, 255, 255});
      }
    }
    for (int x = 0; x < 200; ++x) {
      for (int y = 0; y < 100; ++y) {
        pixel_writer->Write(x, y, {0, 255, 0});
      }
    }
    while (1) __asm__("hlt");
  }
  ```
  - 上記は定義したクラスを使っている部分のソースコードである。pixel_writerとpixel_writer_bufは、グローバル変数である。
  - このプログラムではまず、ピクセルのデータ形式に基づいて、2つの子クラスの適する方のインスタンスを生成し、そのインスタンスへのポインタをpixel_writer変数に設定する。その後、pixel_writerを使って1度画面を白に塗りつぶしたあと、200×100の緑の四角を描画する。
  - newの使い方について記載する。一般的なnewの使い方は、「new クラス名」で、引数は取らない。一般のnewは指定したクラスのインスタンスをヒープ領域に生成する。ヒープ領域は、関数の実行が終了しても破壊されない領域であり、関数を抜けても生存する必要があるデータはnew演算子で確保する。new演算子やmalloc()を使わず普通に定義した変数はスタック領域に配置される。スタック領域に配置された変数は、その変数が定義されたブロックを抜けると破棄される。  
    new演算子は、メモリ領域を確保した後にコンストラクタを呼び出す。クラスのコンストラクタをプログラマが明示的に呼び出す方法はないので、クラスのインスタンスを作るにはnew演算子を使うしか方法がない。  
    一般のnewは変数をヒープに確保するといったが、これはOSがメモリを管理できるようになって初めて可能になる。メモリ管理がない場合でもクラスのインスタンスを作成するには、配置newを使う。配置newはメモリの確保を行わず、その代わりに引数に指定したメモリ領域上にインスタンスを生成する。そのメモリ領域に対してコンストラクタを呼び出す。OSにメモリ管理機能がなくても、配列を使えば好きな大きさのメモリを確保することができるので、配列と配置newを組み合わせることでクラスのインスタンス生成が可能になる。
    #### <main.cpp（配置newの演算子の定義）>
    ```
    void* operator new(size_t size, void* buf) {
      return buf;
    }
  
    void operator delete(void* obj) noexcept {
    }
    ```
    - 配置newは、\<new\>をインクルードするか、自分で定義する必要がある。上記に配置newの実装を記載する。
    - C++ではoperatorキーワードを使うと演算子を定義することができる。一般のnewの引数はsizeだけだが、配置newでは追加の引数があり、そこにメモリ領域へのポインタが渡される。配置newはメモリ領域自体の確保は必要ないので、引数で受け取ったメモリ領域へのポインタbufをそのまま返すだけでよい。
    - operator deleteの方は配置newとは関係ない、普通のdelete演算子である。deleteは使用していないが、~PixelWriter()が子の演算子を要求するようなので定義している。 
  - 配置newを使って生成した子クラスのインスタンスへのポインタを親クラスPixelWriterを指すポインタ型であるpixel_writerに代入する、継承関係がある2つのクラスでは、子クラスのポインタを親クラスのポインタに代入することで、あたかも親クラスであるかのように子クラスを操作することができる。
 
    ![Image 1](pixel_drawing_green_refactor.png)
  
## その他
### make実行時に「 fatal error: 'cstdint' file not found」のエラーが発生する場合
- 以下のコマンドを実行して、makeを実行する。
  ```
  source $HOME/osbook/devenv/buildenv.sh
  ```
  - 自作OSで<cstdint>を使うには、<cstdint>のありかをClangに伝える必要がある。それを手軽に行うためにbuildenv.shというスクリプトファイルが準備されている。このスクリプトをsourceコマンドで実行する。（day03より）   
 

