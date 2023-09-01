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
 - 

 

