# 30日でできる！OS自作入門奮闘記
買ったのでやる気を出すために日記を書くことにしました。頑張って続けられるように頑張ります。（いつもいってるような^^;）
## day1
前説を読んでやる気を高めました。   
バイナリエディタをダウンロードしてきました。   
[ここ](http://www.zob.ne.jp/~c.mos/soft/bz.html)はリンク切れなので、[こっち](https://www.vcraft.jp/soft/bz.html)でダウンロードできました。   
フロッピーがなかったので[ここ](https://www.soonlabo.com/30%E6%97%A5%E3%81%A7%E3%81%A7%E3%81%8D%E3%82%8Bos%E8%87%AA%E4%BD%9C%E5%85%A5%E9%96%80%E3%83%91%E3%83%BC%E3%83%88%EF%BC%91%EF%BC%881%E6%97%A5%E7%9B%AE%EF%BC%89/)の記事をみながらviratual boxを使用しました。   
またcd-romが使えないので[ここ](https://github.com/jacoloves/30nichideosjisaku/tree/master/01_day/helloos0)で本書にか書かれているバッチファイルをコピーしてきます。   
quem等のツール関連は[サポートページ](http://hrb.osask.jp/)を読みましょう。  
後続されるアセンブラのソースコードもしかりです。   
hellos.nasのコードは本書にあるのでぜひ手打ちしてみることをお勧めします。（かなり大変ですが^^;）   
実際書いたアセンブラのソースで恐ろしく打つのがめんどくさかったです。   
```
DB 0xeb, 0x4e, 0x90, 0x48, 0x45, 0x4c, 0x4c, 0x4f
DB 0x49, 0x50, 0x4c, 0x00, 0x02, 0x01, 0x01, 0x00
DB 0x02, 0xe0, 0x00, 0x40, 0x0b, 0xf0, 0x09, 0x00
DB 0x12, 0x00, 0x02, 0x00, 0x00, 0x00, 0x00, 0x00
DB 0x40, 0x0b, 0x00, 0x00, 0x00, 0x00, 0x29, 0xff
DB 0xff, 0xff, 0xff, 0x48, 0x45, 0x4c, 0x4c, 0x4f
DB 0x2d, 0x4f, 0x53, 0x20, 0x20, 0x20, 0x46, 0x41
DB 0x54, 0x31, 0x32, 0x20, 0x20, 0x20, 0x00, 0x00
RESB   16
DB 0xb8, 0x00, 0x00, 0x8e, 0xd0, 0xbc, 0x00, 0x7c
DB 0x8e, 0xd8, 0x8e, 0xc0, 0xbe, 0x74, 0x7c, 0x8a
DB 0x04, 0x83, 0xc6, 0x01, 0x3c, 0x00, 0x74, 0x09
DB 0xb4, 0x0e, 0xbb, 0x0f, 0x00, 0xcd, 0x10, 0xeb
DB 0xee, 0xf4, 0xed, 0xfd, 0x0a, 0x0a, 0x68, 0x65 
DB 0x6c, 0x6c, 0x6f, 0x2c, 0x20, 0x77, 0x6f, 0x72
DB 0x6c, 0x64, 0x0a, 0x00, 0x00, 0x00, 0x00, 0x00
RESB   368
DB 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x55, 0xaa
DB 0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
RESB   4600
DB 0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
RESB   1469432
```

さらにアセンブラを書いていきます   
こっちはだいぶ見やすいですね！   
```
; hello-os
; TAB=4

; FAT12 foramt floppy

        DB      0xeb, 0x4e, 0x90
        DB      "HELLOTPL"
        DW      512
        DB      1
        DW      1
        DB      2
        DW      224
        DW      2880
        DB      0xf0
        DW      9
        DW      18
        DW      2
        DD      0
        DD      2880
        DB      0,0,0x29
        DD      0xffffffff
        DB      "HELLO-OS   "
        DB      "FAT12   "
        RESB    18

; program

        DB      0xb8, 0x00, 0x00, 0x8e, 0xd0, 0xbc, 0x00, 0x7c
        DB      0x8e, 0xd8, 0x8e, 0xc0, 0xbe, 0x74, 0x7c, 0x8a
        DB      0x04, 0x83, 0xc6, 0x01, 0x3c, 0x00, 0x74, 0x09
        DB      0xb4, 0x0e, 0xbb, 0x0f, 0x00, 0xcd, 0x10, 0xeb
        DB      0xee, 0xf4, 0xeb, 0xfd

; message

        DB      0x0a, 0x0a
        DB      "hello, world2"
        DB      0x0a
        DB      0

        RESB    0x1fe-$

        DB      0x55, 0xaa

; boot sector

        DB      0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
        RESB    4600
        DB      0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
        RESB    1469432
```
hello,world2が出力されました！  
1日目はこれで完了です。中々にハードですね汗
## day2
二日目もアセンブラを書くところからスタートです。   
MOV命令やHLT命令などを詳しく学習しました。   
以下はその際記載したソース   
```
; hello-os
; TAB=4

        ORG     0x7c00

; FAT12 foramt floppy

        JMP     entry
        DB      0x90
        DB      "HELLOTPL"
        DW      512
        DB      1
        DW      1
        DB      2
        DW      224
        DW      2880
        DB      0xf0
        DW      9
        DW      18
        DW      2
        DD      0
        DD      2880
        DB      0,0,0x29
        DD      0xffffffff
        DB      "HELLO-OS   "
        DB      "FAT12   "
        RESB    18

; program

entry:
        MOV     AX,0
        MOV     SS,AX
        MOV     SP,0x7c00
        MOV     DS,AX
        MOV     ES,AX

        MOV     SI,msg
putloop:
        MOV     AL,[SI]
        ADD     SI,1
        CMP     AL,0
        JE      fin
        MOV     AH,0x0e
        MOV     BX,15
        INT     0x10
        JMP     putloop
fin:
        HLT
        JMP     fin

; message
msg:
        DB      0x0a, 0x0a
        DB      "hello, world3"
        DB      0x0a
        DB      0

        RESB    0x7dfe-$

        DB      0x55, 0xaa

; boot sector

        DB      0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
        RESB    4600
        DB      0xf0, 0xff, 0xff, 0x00, 0x00, 0x00, 0x00, 0x00
        RESB    1469432
```

**0x00007c00-oxoooo7dff**はブートセクタが読み込まれたアドレスらしいです。   
だから0x7c00を指定しているのか。   
helloos4では各種バッチファイルをいじってosを作りました。   
**ipl.nas**というアセンブラソースファイルを作りました。helloos.nasとの違いはboot sectorの部分がないことです。   
makefileを作成してさらに使いやすくしました。
```
# file making rule

default:
	..\..\z_tools\make.exe img

ipl.bin : ipl.nas Makefile
	..\..\z_tools\nask.exe ipl.nas ipl.bin ipl.lst

helloos.img : ipl.bin Makefile
	..\..\z_tools\edimg.exe imgin:..\..\z_tools\fdimg0at.tek \
		wbinimg src:ipl.bin len:512 from:0 to:0 imgout:helloos.img

img :
	..\..\z_tools\make.exe -r helloos.img

asm :
	..\..\z_tools\make.exe -r ipl.bin

run :
	..\..\z_tools\make.exe img
	cmd.exe /C copy helloos.img ..\..\z_tools\qemu\fdimage0.bin
	..\..\z_tools\make.exe -C ..\..\z_tools\qemu

install :
	..\..\z_tools\make.exe img
	..\..\z_tools\imgtol.com w a: helloos.img

clean :
	rm -f ipl.bin
	rm -f ipl.lst

src_only :
	make clean
	rm -f helloos.img
```

途中meke runコマンドで下記エラーが出てしまい苦労しました。   
```
copy helloos.img ..\..\z_tools\qemu\fdimage0.bin
process_begin: CreateProcess((null), copy helloos.img ..\..\z_tools\qemu\fdimage0.bin, ...) failed.
make (e=2): 指定されたファイルが見つかりません。
..\..\z_tools\make.exe: *** [run] Error 2
```

[この記事](http://ng-r.hateblo.jp/entry/2019/05/24/013306)をみてソースを修正しました。  
```
cmd.exe /C copy helloos.img ..\..\z_tools\qemu\fdimage0.bin
```

原因は恐らくgitbashとwindowsのcopyコマンドが干渉しているからかなと思います。   
要するにディレクトリから直接batファイルたたけば書籍に書かれているソースでも問題なく叩けたのかなと思います。

## day3
アセンブラソースコードをいじりました。
```
; hello-os
; TAB=4

        ORG     0x7c00

; FAT12 foramt floppy

        JMP     entry
        DB      0x90
        DB      "HELLOTPL"
        DW      512
        DB      1
        DW      1
        DB      2
        DW      224
        DW      2880
        DB      0xf0
        DW      9
        DW      18
        DW      2
        DD      0
        DD      2880
        DB      0,0,0x29
        DD      0xffffffff
        DB      "HELLO-OS   "
        DB      "FAT12   "
        RESB    18

; program

entry:
        MOV     AX,0
        MOV     SS,AX
        MOV     SP,0x7c00
        MOV     DS,AX

; read disk

        MOV     AX,0x0820
        MOV     ES,AX
        MOV     CH,0
        MOV     DH,0
        MOV     CL,2

        MOV     AH,0x02
        MOV     AL,1
        MOV     BX,0
        MOV     DL,0x00
        INT     0x13
        JC      error

; read finished but nothing task, it is slept

fin:
        HLT
        JMP     fin

error:
        MOV      SI,msg
putloop:
        MOV     AL,[SI]
        ADD     SI,1
        CMP     AL,0
        JE      fin
        MOV     AH,0x0e
        MOV     BX,15
        INT     0x10
        JMP     putloop
; message
msg:
        DB      0x0a, 0x0a
        DB      "load error"
        DB      0x0a
        DB      0

        RESB    0x7dfe-$

        DB      0x55, 0xaa
```

makeファイルも書き換えました。
```
TOOLPATH = ..\..\z_tools\\

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f


# default operation
default:
	$(MAKE) img

# file making rule

ipl.bin : ipl.nas Makefile
	$(NASK) ipl.nas ipl.bin ipl.lst

helloos.img : ipl.bin Makefile
	$(EDIMG) imgin:..\..\z_tools\fdimg0at.tek \
		wbinimg src:ipl.bin len:512 from:0 to:0 imgout:helloos.img

# command

img :
	$(MAKE) helloos.img

asm :
	$(MAKE) ipl.bin

run :
	$(MAKE) img
	$(COPY) helloos.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: helloos.img

clean :
	-$(DEL) ipl.bin
	-$(DEL) ipl.lst

src_only :
	$(MAKE) clean
	-$(DEL) helloos.img
```

下記のようにTOLLPATHとMAKE変数の間に改行が入っている理由は「\」はコードが続いていると思われてしまうので改行で**MAKEはTOOLPATH**の続きではありませんと教えています。
```
TOOLPATH = ..\..\z_tools\\

MAKE     = $(TOOLPATH)make.exe -r
```

エラーになったらやり直せるようにosを改良しました。
```
; hello-os
; TAB=4

        ORG     0x7c00

; FAT12 foramt floppy

        JMP     entry
        DB      0x90
        DB      "HELLOTPL"
        DW      512
        DB      1
        DW      1
        DB      2
        DW      224
        DW      2880
        DB      0xf0
        DW      9
        DW      18
        DW      2
        DD      0
        DD      2880
        DB      0,0,0x29
        DD      0xffffffff
        DB      "HELLO-OS   "
        DB      "FAT12   "
        RESB    18

; program

entry:
        MOV     AX,0
        MOV     SS,AX
        MOV     SP,0x7c00
        MOV     DS,AX

; read disk

        MOV     AX,0x0820
        MOV     ES,AX
        MOV     CH,0
        MOV     DH,0
        MOV     CL,2

        MOV     SI,0
retry:
        MOV     AH,0x02
        MOV     AL,1
        MOV     BX,0
        MOV     DL,0x00
        INT     0x13
        JNC     fin
        ADD     SI,1
        CMP     SI,5
        JAE     error
        MOV     AH,0x00
        MOV     DL,0x00
        INT     0x13
        JMP     retry

; read finished but nothing task, it is slept

fin:
        HLT
        JMP     fin

error:
        MOV      SI,msg
putloop:
        MOV     AL,[SI]
        ADD     SI,1
        CMP     AL,0
        JE      fin
        MOV     AH,0x0e
        MOV     BX,15
        INT     0x10
        JMP     putloop
; message
msg:
        DB      0x0a, 0x0a
        DB      "load error"
        DB      0x0a
        DB      0

        RESB    0x7dfe-$

        DB      0x55, 0xaa
```

ここはスラスラすすむこことができました。   
18セクタ先まで読み込むアセンブラのソースを書きました。
```
entry:
        MOV     AX,0
        MOV     SS,AX
        MOV     SP,0x7c00
        MOV     DS,AX

; read disk

        MOV     AX,0x0820
        MOV     ES,AX
        MOV     CH,0
        MOV     DH,0
        MOV     CL,2
readloop:
        MOV     SI,0
retry:
        MOV     AH,0x02
        MOV     AL,1
        MOV     BX,0
        MOV     DL,0x00
        INT     0x13
        JNC     next
        ADD     SI,1
        CMP     SI,5
        JAE     error
        MOV     AH,0x00
        MOV     DL,0x00
        INT     0x13
        JMP     retry
next:
        MOV     AX,ES
        ADD     AX,0x0020
        MOV     ES,AX
        ADD     CL,1
        CMP     CL,18
        JBE     readloop
```

さらにアセンブラを加筆しました。   
今回は10シリンダ分読み込ませています。   
```
; hello-os
; TAB=4

;cylinders
CYLS    EQU     10

        ORG     0x7c00

; FAT12 foramt floppy

        JMP     entry
        DB      0x90
        DB      "HARIBOTE"
        DW      512
        DB      1
        DW      1
        DB      2
        DW      224
        DW      2880
        DB      0xf0
        DW      9
        DW      18
        DW      2
        DD      0
        DD      2880
        DB      0,0,0x29
        DD      0xffffffff
        DB      "HELLO-OS   "
        DB      "FAT12   "
        RESB    18

; program

entry:
        MOV     AX,0
        MOV     SS,AX
        MOV     SP,0x7c00
        MOV     DS,AX

; read disk

        MOV     AX,0x0820
        MOV     ES,AX
        MOV     CH,0
        MOV     DH,0
        MOV     CL,2
readloop:
        MOV     SI,0
retry:
        MOV     AH,0x02
        MOV     AL,1
        MOV     BX,0
        MOV     DL,0x00
        INT     0x13
        JNC     next
        ADD     SI,1
        CMP     SI,5
        JAE     error
        MOV     AH,0x00
        MOV     DL,0x00
        INT     0x13
        JMP     retry
next:
        MOV     AX,ES
        ADD     AX,0x0020
        MOV     ES,AX
        ADD     CL,1
        CMP     CL,18
        JBE     readloop
        MOV     CL,1
        ADD     DH,1
        CMP     DH,2
        JB      readloop
        MOV     DH,0
        ADD     CH,1
        CMP     CH,CYLS
        JB      readloop
```

次はOSを書きました。 
まずはアセンブラのから  
```
fin:
        HLT
        JMP     fin
```

次にMakefile
```
TOOLPATH = ..\..\z_tools\\

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f


# default operation
default:
	$(MAKE) img

# file making rule

ipl.bin : ipl.nas Makefile
	$(NASK) ipl.nas ipl.bin ipl.lst

haribote.sys : haribote.nas Makefile
	$(NASK) haribote.nas haribote.sys haribote.lst

haribote.img : ipl.bin haribote.sys Makefile
	$(EDIMG)   imgin:..\..\z_tools/fdimg0at.tek \
		wbinimg src:ipl.bin len:512 from:0 to:0 \
		copy from:haribote.sys to:@: \
		imgout:haribote.img

# command

img :
	$(MAKE) haribote.img

run :
	$(MAKE) img
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: haribote.img

clean :
	-$(DEL) ipl.bin
	-$(DEL) ipl.lst
	-$(DEL) haribote.sys
	-$(DEL) haribote.lst

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img
```

なんというか一番苦労させられたのはMakefileの修正です。   
結局エラーの原因はharibote.nasに名前を**harinote.nas**にしてたからという誤字が原因でした。   

今回はharibote.nasとipl.nasに少し手を加えました。
haribote.nas
```
; haribote-os
; TAB=4

        ORG     0xc200
fin:
        HLT
        JMP     fin
```

ipl.nas
```
next:
        MOV     AX,ES
        ADD     AX,0x0020
        MOV     ES,AX
        ADD     CL,1
        CMP     CL,18
        JBE     readloop
        MOV     CL,1
        ADD     DH,1
        CMP     DH,2
        JB      readloop
        MOV     DH,0
        ADD     CH,1
        CMP     CH,CYLS
        JB      readloop

; read finished and execute haribote.sys!
        JMP 0xc200
```

次は画面モードを開発をしました。   
haribote.nasとipl10.nas(ipl.nasから名前を変更しています。)とMakefileを修正しました。   
まずharibote.nasが
```
; haribote-os
; TAB=4

        ORG     0xc200

; screen mode
        MOV     AL,0x13
        MOV     AH,0x00
        INT     0x10
```

AH=0x00がビデオBIOSとの切り替えに使われるらしいです。   
ipl10.nasは
```
next:
        MOV     AX,ES
        ADD     AX,0x0020
        MOV     ES,AX
        ADD     CL,1
        CMP     CL,18
        JBE     readloop
        MOV     CL,1
        ADD     DH,1
        CMP     DH,2
        JB      readloop
        MOV     DH,0
        ADD     CH,1
        CMP     CH,CYLS
        JB      readloop

; read finished and execute haribote.sys!
        MOV     [0x0ff0],CH
        JMP     0xc200
```
JMP     0xc200の前にMOV     [0x0ff0],CHを追加しました。   
Makefile
```
TOOLPATH = ..\..\z_tools\\

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f


# default operation
default:
	$(MAKE) img

# file making rule

ipl10.bin : ipl10.nas Makefile
	$(NASK) ipl10.nas ipl10.bin ipl10.lst

haribote.sys : haribote.nas Makefile
	$(NASK) haribote.nas haribote.sys haribote.lst

haribote.img : ipl10.bin haribote.sys Makefile
	$(EDIMG)   imgin:..\..\z_tools/fdimg0at.tek \
		wbinimg src:ipl10.bin len:512 from:0 to:0 \
		copy from:haribote.sys to:@: \
		imgout:haribote.img

# command

img :
	$(MAKE) haribote.img

run :
	$(MAKE) img
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: haribote.img

clean :
	-$(DEL) ipl10.bin
	-$(DEL) ipl10.lst
	-$(DEL) haribote.sys
	-$(DEL) haribote.lst

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img
```

ipl~と記載があった部分をipl10~に変更しています。   
続いてはキーボードの状態をBIOSに教えてもらうプログラムを作成しました。   
haribote.nasを修正しました。   
```
; haribote-os
; TAB=4

; BOOT_INFO
CYLS    EQU     0x0ff0
LEDS    EQU     0x0ff1
VMODE   EQU     0x0ff2
SCRNX   EQU     0x0ff4
SCRNY   EQU     0x0ff6
VRAM    EQU     0x0ff8

        ORG     0xc200

; screen mode
        MOV     AL,0x13
        MOV     AH,0x00
        INT     0x10
        MOV     BYTE [VMODE],8
        MOV     WORD [SCRNX],320
        MOV     WORD [SCRNY],200
        MOV     DWORD [VRAM],0x000a0000

; keybord's LED state is teached BIOS

        MOV     AH,0x02
        INT     0x16
        MOV     [LEDS],AL
fin:
        HLT
        JMP     fin
```

いよいよ最大の鬼門、32bit化です。   
ソースコードを写経しながら眺めるスタンスは変わらないのですが、一つだけツールの問題で環境を多少いじらないといけないエラーにぶつかることがあります。   
環境によってはその問題が起きないのですが私の環境では発生して苦労したので共有します。   
```
C:\Users\hariboteos\day_03\harib00i>..\..\z_tools\make.exe bootpack.hrb
..\..\z_tools\\cc1.exe -I..\..\z_tools\haribote\\ -Os -Wall -quiet -o bootpack.gas bootpack.c
bootpack.c:8:2: warning: no newline at end of file
..\..\z_tools\\gas2nask.exe -a bootpack.gas  bootpack.nas
..\..\z_tools\\nask.exe bootpack.nas bootpack.obj bootpack.lst
..\..\z_tools\\obj2bim.exe @..\..\z_tools\\haribote\haribote.rul out:bootpack.bim stack:3136k map:bootpack.map \
        bootpack.obj
Rule file error : can't open file : ../z_tools/haribote/harilibc.lib
..\..\z_tools\make.exe: *** [bootpack.bim] Error 6
```

上記のエラーは**make bootpack.hrb**と叩くと発生します。   
恐らくobj2bim.exeがディレクトリ探索する際、**../z_tools/haribote**配下を見に行く仕様になってるから発生するエラーだと思われます。   
私のように./〇〇/day_03/harib00iみたいなディレクトリ構造をしてるとエラーになってしまうので発生したら、**コンソール起動している一段上のディレクトリにz_toolsフォルダ格納する**ことでエラーが解消されます。

全体的に作り替えなくてはいけないところも多いですがぜひ頑張ってチャレンジしてみてください！   

asmhead.nas
```
; haribote-os boot asm
; TAB=4

BOTPAK  EQU     0x00280000
DSKCAC  EQU     0x00100000
DSKCAC0 EQU     0x00008000

; BOOT_INFO
CYLS    EQU     0x0ff0
LEDS    EQU     0x0ff1
VMODE   EQU     0x0ff2
SCRNX   EQU     0x0ff4
SCRNY   EQU     0x0ff6
VRAM    EQU     0x0ff8

        ORG     0xc200

; screen mode

        MOV     AL,0x13
        MOV     AH,0x00
        INT     0x10
        MOV     BYTE [VMODE],8
        MOV     WORD [SCRNX],320
        MOV     WORD [SCRNY],200
        MOV     DWORD [VRAM],0x000a0000

; keybord's LED state is teached BIOS

        MOV     AH,0x02
        INT     0x16
        MOV     [LEDS],AL

; PICが一切の割り込みを受け付けないようにする
;	AT互換機の仕様では、PICの初期化をするなら、
;	こいつをCLI前にやっておかないと、たまにハングアップする
;	PICの初期化はあとでやる

        MOV     AL,0xff
        OUT     0x21,AL
        NOP
        OUT     0xa1,AL

        CLI

; Set A20GATE so that CPU can access more than 1MB of memory

        CALL    waitkbdout
        MOV     AL,0xd1
        OUT     0x64,AL
        CALL    waitkbdout
        MOV     AL,0xdf
        OUT     0x60,AL
        CALL    waitkbdout

; move protect mode

[INSTRSET "i486p"]

        LGDT    [GDTR0]
        MOV     EAX,CR0
        AND     EAX,0x7fffffff
        OR      EAX,0x00000001
        MOV     CR0,EAX
        JMP     pipelineflush
pipelineflush:
        MOV     AX,1*8
        MOV     DS,AX
        MOV     ES,AX
        MOV     FS,AX
        MOV     GS,AX
        MOV     SS,AX

; bootpack trancfer

        MOV     ESI,bootpack
        MOV     EDI,BOTPAK
        MOV     ECX,512*1024/4
        CALL    memcpy

; ついでにディスクデータも本来の位置へ転送

; boot sector

        MOV     ESI,0x7c00
        MOV     EDI,DSKCAC
        MOV     ECX,512/4
        CALL    memcpy

; remaining

        MOV     ESI,DSKCAC0+512
        MOV     EDI,DSKCAC+512
        MOV     ECX,0
        MOV     CL,BYTE [CYLS]
        IMUL    ECX,512*18*2/4
        SUB     ECX,512/4
        CALL    memcpy

; bootpack
;

; bootpack run

        MOV     EBX,BOTPAK
        MOV     ECX,[EBX+16]
        ADD     ECX,3
        SHR     ECX,2
        JZ      skip
        MOV     ESI,[EBX+20]
        ADD     ESI,EBX
        MOV     EDI,[EBX+12]
        CALL    memcpy
skip:
        MOV     ESP,[EBX+12]
        JMP     DWORD 2*8:0x0000001b

waitkbdout:
        IN      AL,0x64
        AND     AL,0x02
        JNZ    waitkbdout
        RET

memcpy:
        MOV     EAX,[ESI]
        ADD     ESI,4
        MOV     [EDI],EAX
        ADD     EDI,4
        SUB     ECX,1
        JNZ     memcpy
        RET
; memcpy can also be written with string instructions, unless you forget the address size prefix

        ALIGNB  16
GDT0:
        RESB    8
        DW      0xffff,0x0000,0x9200,0x00cf
        DW      0xffff,0x0000,0x9a28,0x0047

        DW      0
GDTR0:
        DW      8*3-1
        DD      GDT0

        ALIGNB  16
bootpack:
```

Makefile
```
TOOLPATH = ..\..\z_tools\\

INCPATH  = ..\..\z_tools\haribote\\

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
CC1		 = $(TOOLPATH)cc1.exe -I$(INCPATH) -Os -Wall -quiet
GAS2NASK = $(TOOLPATH)gas2nask.exe -a
OBJ2BIM  = $(TOOLPATH)obj2bim.exe
BIM2HRB  = $(TOOLPATH)bim2hrb.exe
RULEFILE = $(TOOLPATH)haribote\haribote.rul
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f


# default operation
default:
	$(MAKE) img

# file making rule

ipl10.bin : ipl10.nas Makefile
	$(NASK) ipl10.nas ipl10.bin ipl10.lst

asmhead.bin : asmhead.nas Makefile
	$(NASK) asmhead.nas asmhead.bin asmhead.lst

bootpack.gas : bootpack.c Makefile
	$(CC1) -o bootpack.gas bootpack.c

bootpack.nas : bootpack.gas Makefile
	$(GAS2NASK) bootpack.gas  bootpack.nas 

bootpack.obj : bootpack.nas Makefile
	$(NASK) bootpack.nas bootpack.obj bootpack.lst

bootpack.bim : bootpack.obj Makefile
	$(OBJ2BIM) @$(RULEFILE) out:bootpack.bim stack:3136k map:bootpack.map \
		bootpack.obj
# 3MB+64KB=3136KB

bootpack.hrb : bootpack.bim Makefile
	$(BIM2HRB) bootpack.bim bootpack.hrb 0

haribote.sys : asmhead.bin bootpack.hrb Makefile
	cat asmhead.bin >  haribote.sys
	cat bootpack.hrb >> haribote.sys

haribote.img : ipl10.bin haribote.sys Makefile
	$(EDIMG)   imgin:..\..\z_tools\fdimg0at.tek \
		wbinimg src:ipl10.bin len:512 from:0 to:0 \
		copy from:haribote.sys to:@: \
		imgout:haribote.img

# command

img :
	$(MAKE) haribote.img

run :
	$(MAKE) img
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: haribote.img

clean :
	-$(DEL) *.bin
	-$(DEL) *.lst
	-$(DEL) *.gas
	-$(DEL) *.obj
	-$(DEL) bootpack.nas
	-$(DEL) bootpack.map
	-$(DEL) bootpack.bim
	-$(DEL) bootpack.hrb
	-$(DEL) haribote.sys

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img
```
 今日の最後にHLTを追加する処理を実装しました。   
新規で追加するモジュールもありますがここまできた皆さんならそこまで難しくないのではないでしょうか。   
naskfunc.nas
```
TOOLPATH = ..\..\z_tools\\

INCPATH  = ..\..\z_tools\haribote\\

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
CC1		 = $(TOOLPATH)cc1.exe -I$(INCPATH) -Os -Wall -quiet
GAS2NASK = $(TOOLPATH)gas2nask.exe -a
OBJ2BIM  = $(TOOLPATH)obj2bim.exe
BIM2HRB  = $(TOOLPATH)bim2hrb.exe
RULEFILE = $(TOOLPATH)haribote\haribote.rul
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f


# default operation
default:
	$(MAKE) img

# file making rule

ipl10.bin : ipl10.nas Makefile
	$(NASK) ipl10.nas ipl10.bin ipl10.lst

asmhead.bin : asmhead.nas Makefile
	$(NASK) asmhead.nas asmhead.bin asmhead.lst

bootpack.gas : bootpack.c Makefile
	$(CC1) -o bootpack.gas bootpack.c

bootpack.nas : bootpack.gas Makefile
	$(GAS2NASK) bootpack.gas  bootpack.nas 

bootpack.obj : bootpack.nas Makefile
	$(NASK) bootpack.nas bootpack.obj bootpack.lst

naskfunc.obj : naskfunc.nas Makefile
	$(NASK) naskfunc.nas naskfunc.obj naskfunc.lst

bootpack.bim : bootpack.obj naskfunc.obj Makefile
	$(OBJ2BIM) @$(RULEFILE) out:bootpack.bim stack:3136k map:bootpack.map \
		bootpack.obj naskfunc.obj
# 3MB+64KB=3136KB

bootpack.hrb : bootpack.bim Makefile
	$(BIM2HRB) bootpack.bim bootpack.hrb 0

haribote.sys : asmhead.bin bootpack.hrb Makefile
	cat asmhead.bin >  haribote.sys
	cat bootpack.hrb >> haribote.sys

haribote.img : ipl10.bin haribote.sys Makefile
	$(EDIMG)   imgin:..\..\z_tools\fdimg0at.tek \
		wbinimg src:ipl10.bin len:512 from:0 to:0 \
		copy from:haribote.sys to:@: \
		imgout:haribote.img

# command

img :
	$(MAKE) haribote.img

run :
	$(MAKE) img
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: haribote.img

clean :
	-$(DEL) *.bin
	-$(DEL) *.lst
	-$(DEL) *.gas
	-$(DEL) *.obj
	-$(DEL) bootpack.nas
	-$(DEL) bootpack.map
	-$(DEL) bootpack.bim
	-$(DEL) bootpack.hrb
	-$(DEL) haribote.sys

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img
```

bootpack.c
```
void io_hlt(void);

void HariMain(void)
{

fin:
    io_hlt();
    goto fin;
    
}
```

Makefile
```
# file making rule

ipl10.bin : ipl10.nas Makefile
	$(NASK) ipl10.nas ipl10.bin ipl10.lst

asmhead.bin : asmhead.nas Makefile
	$(NASK) asmhead.nas asmhead.bin asmhead.lst

bootpack.gas : bootpack.c Makefile
	$(CC1) -o bootpack.gas bootpack.c

bootpack.nas : bootpack.gas Makefile
	$(GAS2NASK) bootpack.gas  bootpack.nas 

bootpack.obj : bootpack.nas Makefile
	$(NASK) bootpack.nas bootpack.obj bootpack.lst

naskfunc.obj : naskfunc.nas Makefile
	$(NASK) naskfunc.nas naskfunc.obj naskfunc.lst

bootpack.bim : bootpack.obj naskfunc.obj Makefile
	$(OBJ2BIM) @$(RULEFILE) out:bootpack.bim stack:3136k map:bootpack.map \
		bootpack.obj naskfunc.obj
# 3MB+64KB=3136KB
```

これにて本日は終了です。   
凄くハードでしたがお疲れさまでした。 

**実装過程**   
・　[f19327e0cd072bdbc70833d803d423437a7c9823](f19327e0cd072bdbc70833d803d423437a7c9823)


VRAMに何か書き込みたいのでnaskfunc.nasをいじります。
naskfunc.nas
```
; naskfunc
; TAB=4

[FORMAT "WCOFF"]
[INSTRSET "i486p"]
[BITS 32]

; object file

[FILE "naskfunc.nas"]

        GLOBAL  _io_hlt,_write_mem8

; function

[SECTION .test]

_io_hlt:
        HLT
        RET

_write_mem8:    ; void write_mem8(int addr, int data);
        MOV     ECX,[ESP+4]     ; [ESP+4] in addr read ECX
        MOV     AL,[ESP+8]      ; [ESP+8] in data read AL
        MOV     [ECX],AL
        RET
```

_write_mem8を呼び出すためにbootpack.cも改造します。
bootpack.c
```
void io_hlt(void);
void write_mem8(int addr, int data);

void HariMain(void)
{
    int i;

    for (i = 0xa0000; i <= 0xaffff; i++) {
        write_mem8(i, 15);
    }

    for (;;) {
        io_hlt();
    }
    
}
```

次はしましまに改造するためにbootpack.cに少し手を加えます。   
bootpack.c
```
void HariMain(void)
{
    int i;

    for (i = 0xa0000; i <= 0xaffff; i++) {
        write_mem8(i, i & 0x0f);
    }

    for (;;) {
        io_hlt();
    }
    
}
```

ここからはポインタについて見ていく回なのでざっと書いていきます。   
青い画面が表示されます。   
bootpack.c
```
void HariMain(void)
{
    int i;
    char *p;

    for (i = 0xa0000; i <= 0xaffff; i++) {
        p =i;
        *p = 1 & 0x0f;
    }

    for (;;) {
        io_hlt();
    }
    
}
```
次はしましまの画面が表示されます。   
bootpack.c
```
void HariMain(void)
{
    int i;
    char *p;

    p = (char *) 0xa0000;

    for (i = 0; i <= 0xffff; i++) {
        *(p + i) = i & 0x0f;
    }

    for (;;) {
        io_hlt();
    }
    
}
```
次も企保的に瓦井ですが*(p + i)の部分がp[i]になっています。   
bootpack.c
```
void HariMain(void)
{
    int i;
    char *p;

    p = (char *) 0xa0000;

    for (i = 0; i <= 0xffff; i++) {
        p[i] = i & 0x0f;
    }

    for (;;) {
        io_hlt();
    }
    
}
```

**実装過程**   
・　[cb092449d4b3f7dee5bd09004a736b6905d3a889](cb092449d4b3f7dee5bd09004a736b6905d3a889)

## day4
day5と言っていますがこれは書籍の4日目ではなく筆者がOS作りを初めて4日目がということを指します。煩わしくて申し訳ありません。   
今回は色をカラフルにしてあげるところからスタートします。C言語をゴリゴリ実装していくのでとても楽しみです。   
bootpack.c
```
void io_hlt(void);
void io_cli(void);
void io_out8(int port, int data);
int io_load_eflags(void);
void io_store_eflags(int eflags);

void init_palette(void);
void set_palette(int start, int end, unsigned char *rgd);

void HariMain(void)
{
    int i;
    char *p;

    init_palette();

    p = (char *) 0xa0000;

    for (i = 0; i <= 0xffff; i++) {
        p[i] = i & 0x0f;
    }

    for (;;) {
        io_hlt();
    }
    
}

void init_palette(void)
{
    static unsigned char table_rgb[16 * 3] = {
        0x00, 0x00, 0x00,   /* 0:black */
        0xff, 0x00, 0x00,   /* 1:light red */
        0x00, 0xff, 0x00,   /* 2:light green */
        0xff, 0xff, 0x00,   /* 3:light yellow */
        0x00, 0x00, 0xff,   /* 4:light blue */
        0xff, 0x00, 0xff,   /* 5:light puple */
        0x00, 0xff, 0xff,   /* 6:light sky blue */
        0xff, 0xff, 0xff,   /* 7:white */
        0xc6, 0xc6, 0xc6,   /* 8:light grey */
        0x84, 0x00, 0x00,   /* 9:dark green */
        0x00, 0x84, 0x00,   /* 10:dark green */
        0x84, 0x84, 0x00,   /* 11:dark yellow */
        0x00, 0x00, 0x84,   /* 12:dark blue */
        0x84, 0x00, 0x84,   /* 13:dark puple */
        0x00, 0x84, 0x84,   /* 14:dark sky blue */
        0x84, 0x84, 0x84    /* 15:dark grey */
    };
    set_palette(0, 15, table_rgb);
    return;
}

void set_palette(int start, int end, unsigned char *rgb)
{
    int i, eflags;
    eflags = io_load_eflags();
    io_cli();
    io_out8(0x03c8, start);
    for (i = start; i <= end; i++) {
        io_out8(0x03c9, rgb[0] / 4);
        io_out8(0x03c9, rgb[1] / 4);
        io_out8(0x03c9, rgb[2] / 4);
        rgb += 3;
    }
    io_store_eflags(eflags);
    return;
}
```


続いては、    
void io_cli(void)    
void io_out8(int port, int data)    
int io_load_eflags(void)    
void io_store_eflags(int eflags)    
の4つを実装していくためにnaskfunc.nasを修正します。    
naskfunc.nas
```
; naskfunc
; TAB=4

[FORMAT "WCOFF"]
[INSTRSET "i486p"]
[BITS 32]
[FILE "naskfunc.nas"]

        GLOBAL  _io_hlt, _io_cli, _io_sci, _io_stihlt
        GLOBAL  _io_in8, _io_in16, _io_in32
        GLOBAL  _io_out8, _io_out16, _io_out32
        GLOBAL  _io_load_eflags, _io_store_eflags

; object file

[FILE "naskfunc.nas"]

; function

[SECTION .text]

_io_hlt:
        HLT
        RET

_io_cli:
        CLI
        RET

_io_sti:
        STI
        HLT
        RET

_io_in8:
        MOV     EDX,[ESP+4]
        MOV     EAX,0
        IN      AL,DX
        RET

_io_in16:
        MOV     EDX,[ESP+4]
        MOV     EAX,0
        IN      AX,DX
        RET

_io_in32:
        MOV     EDX,[ESP+4]
        IN      EAX,DX
        RET
        
_io_out8:
        MOV     EDX,[ESP+4]
        MOV     AL,[ESP+8]
        OUT     DX,AL
        RET

_io_out16:
        MOV     EDX,[ESP+4]
        MOV     EAX,[ESP+8]
        OUT     DX,AX
        RET

_io_out32:
        MOV     EDX,[ESP+4]
        MOV     EAX,[ESP+8]
        OUT     DX,EAX
        RET

_io_load_eflags:
        PUSHFD
        POP     EAX
        RET

_io_store_eflags:
        MOV     EAX,[ESP+4]
        PUSH    EAX
        POPFD
        RET
```

これで画面のしましまのいろが変わりました。    
続いて四角形を描画していきます。ここではdefineでvramの番地をあらかじめ決めています。    
bootpack.c
```
void io_hlt(void);
void io_cli(void);
void io_out8(int port, int data);
int io_load_eflags(void);
void io_store_eflags(int eflags);

void init_palette(void);
void set_palette(int start, int end, unsigned char *rgd);
void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1);

#define COL8_000000  0
#define COL8_FF0000  1
#define COL8_00FF00  2
#define COL8_FFFF00  3
#define COL8_0000FF  4
#define COL8_FF00FF  5
#define COL8_00FFFF  6
#define COL8_FFFFFF  7
#define COL8_C6C6C6  8
#define COL8_840000  9
#define COL8_008400  10
#define COL8_848400  11
#define COL8_000084  12
#define COL8_840084  13
#define COL8_008484  14
#define COL8_848484  15


void HariMain(void)
{
    int i;
    char *p;

    init_palette();

    p = (char *) 0xa0000;

    boxfill8(p, 320, COL8_FF0000, 20, 20, 120, 120);
    boxfill8(p, 320, COL8_00FF00, 70, 50, 170, 150);
    boxfill8(p, 320, COL8_0000FF, 120, 80, 220, 180);

    for (;;) {
        io_hlt();
    }
    
}

void init_palette(void)
{
    static unsigned char table_rgb[16 * 3] = {
        0x00, 0x00, 0x00,   /* 0:black */
        0xff, 0x00, 0x00,   /* 1:light red */
        0x00, 0xff, 0x00,   /* 2:light green */
        0xff, 0xff, 0x00,   /* 3:light yellow */
        0x00, 0x00, 0xff,   /* 4:light blue */
        0xff, 0x00, 0xff,   /* 5:light puple */
        0x00, 0xff, 0xff,   /* 6:light sky blue */
        0xff, 0xff, 0xff,   /* 7:white */
        0xc6, 0xc6, 0xc6,   /* 8:light grey */
        0x84, 0x00, 0x00,   /* 9:dark green */
        0x00, 0x84, 0x00,   /* 10:dark green */
        0x84, 0x84, 0x00,   /* 11:dark yellow */
        0x00, 0x00, 0x84,   /* 12:dark blue */
        0x84, 0x00, 0x84,   /* 13:dark puple */
        0x00, 0x84, 0x84,   /* 14:dark sky blue */
        0x84, 0x84, 0x84    /* 15:dark grey */
    };
    set_palette(0, 15, table_rgb);
    return;
}

void set_palette(int start, int end, unsigned char *rgb)
{
    int i, eflags;
    eflags = io_load_eflags();
    io_cli();
    io_out8(0x03c8, start);
    for (i = start; i <= end; i++) {
        io_out8(0x03c9, rgb[0] / 4);
        io_out8(0x03c9, rgb[1] / 4);
        io_out8(0x03c9, rgb[2] / 4);
        rgb += 3;
    }
    io_store_eflags(eflags);
    return;
}

void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1)
{
    int x, y;
    for (y= y0; y <= y1; y++) {
        for (x = x0; x <= x1; x++)
            vram[y * xsize + x] = c;
    }
    return;
}
```

これで四角形が出てきました。   
つぎはタスクバーを表示させるところです。   
四角形を出すところで定義したbpxfill8関数を使っていきます。   

bootpack.c
```
void HariMain(void)
{
    char *vram;
    int xsize, ysize;

    init_palette();
    vram = (char *) 0xa0000;
    xsize = 320;
    ysize = 200;

    boxfill8(vram, xsize, COL8_008484,  0,          0,           xsize - 1,   ysize - 29);
    boxfill8(vram, xsize, COL8_C6C6C6,  0,          ysize - 28,  xsize - 1,   ysize - 28);
    boxfill8(vram, xsize, COL8_FFFFFF,  0,          ysize - 27,  xsize - 1,   ysize - 27);
    boxfill8(vram, xsize, COL8_C6C6C6,  0,          ysize - 26,  xsize - 1,   ysize -  1);

    boxfill8(vram, xsize, COL8_FFFFFF,  3,          ysize - 24,  59,          ysize - 24);
    boxfill8(vram, xsize, COL8_FFFFFF,  2,          ysize - 24,   2,          ysize -  4);
    boxfill8(vram, xsize, COL8_848484,  3,          ysize -  4,  59,          ysize -  4);
    boxfill8(vram, xsize, COL8_848484, 59,          ysize - 23,  59,          ysize -  5);
    boxfill8(vram, xsize, COL8_000000,  2,          ysize -  3,  59,          ysize -  3);
    boxfill8(vram, xsize, COL8_000000, 60,          ysize - 24,  60,          ysize -  3);

    boxfill8(vram, xsize, COL8_848484, xsize - 47,  ysize - 24,  xsize -  4,   ysize - 24);
    boxfill8(vram, xsize, COL8_848484, xsize - 47,  ysize - 23,  xsize - 47,   ysize -  4);
    boxfill8(vram, xsize, COL8_FFFFFF, xsize - 47,  ysize -  3,  xsize -  4,   ysize -  3);
    boxfill8(vram, xsize, COL8_FFFFFF, xsize -  3,  ysize - 24,  xsize -  3,   ysize -  3);

    for (;;) {
        io_hlt();
    }
    
}
```

これでOSっぽくなりました。   
達成感半端ないです！    

本では五日目に突入しました。   
asmhead.nasの値を取得するために少しbootpack.cを改造します。   
bootpack.c
```
void HariMain(void)
{
    char *vram;
    int xsize, ysize;
    short *binfo_scrnx, *binfo_scrny;
    int *binfo_vram;

    init_palette();
    binfo_scrnx = (short *) 0x0ff4;
    binfo_scrny = (short *) 0x0ff6;
    binfo_vram = (int *) 0x0ff8;
    xsize = *binfo_scrnx;
    ysize = *binfo_scrny;
    vram = (char *) *binfo_vram;
```

次は構造体を使用してHariMain関数の行数を削りました。   

```
struct BOOTINFO {
    char cyls, leds, vmode, reserve;
    short scrnx, scrny;
    char *vram;
};

void HariMain(void)
{
    char *vram;
    int xsize, ysize;
    struct BOOTINFO *binfo;

    init_palette();
    binfo = (struct BOOTINFO *) 0x0ff0;
    xsize = (*binfo).scrnx;
    ysize = (*binfo).scrny;
    vram = (*binfo).vram;
```

あまり削れてないような。
さら削るためにアロー演算子を使ってみます   

```
void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) 0x0ff0;
 
    init_palette();
    init_screen(binfo->vram, binfo->scrnx, binfo->scrny);

    for (;;) {
        io_hlt();
    }
    
}
```

またinit_screen関数を定義してboxfill8(vram, x, COL8_008484,  0,          0,       x - 1,    y - 29);の文を小分けにしました。

```
void init_screen(char *vram, int x, int y)
{
    boxfill8(vram, x, COL8_008484,  0,          0,       x - 1,    y - 29);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 28,  x - 1,    y - 28);
    boxfill8(vram, x, COL8_FFFFFF,  0,          y - 27,  x - 1,    y - 27);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 26,  x - 1,    y -  1);

    boxfill8(vram, x, COL8_FFFFFF,  3,          y - 24,  59,       y - 24);
    boxfill8(vram, x, COL8_FFFFFF,  2,          y - 24,   2,       y -  4);
    boxfill8(vram, x, COL8_848484,  3,          y -  4,  59,       y -  4);
    boxfill8(vram, x, COL8_848484, 59,          y - 23,  59,       y -  5);
    boxfill8(vram, x, COL8_000000,  2,          y -  3,  59,       y -  3);
    boxfill8(vram, x, COL8_000000, 60,          y - 24,  60,       y -  3);

    boxfill8(vram, x, COL8_848484, x - 47,      y - 24,  x -  4,   y - 24);
    boxfill8(vram, x, COL8_848484, x - 47,      y - 23,  x - 47,   y -  4);
    boxfill8(vram, x, COL8_FFFFFF, x - 47,      y -  3,  x -  4,   y -  3);
    boxfill8(vram, x, COL8_FFFFFF, x -  3,      y - 24,  x -  3,   y -  3);
    return;
}
```

プロト関数も忘れずにつけましょう。   
```
void init_palette(void);
void set_palette(int start, int end, unsigned char *rgd);
void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1);
void init_screen(char *vram, int x, int y);
```

**実装過程**   
・　[57a54599e9dedb9733ffbfdcb8a75f72732b31c7](57a54599e9dedb9733ffbfdcb8a75f72732b31c7)

## day5
文字を出力させるためにbootpack.cを修正ます。   
今回はAを出します。   

```
void io_hlt(void);
void io_cli(void);
void io_out8(int port, int data);
int io_load_eflags(void);
void io_store_eflags(int eflags);

void init_palette(void);
void set_palette(int start, int end, unsigned char *rgd);
void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1);
void init_screen(char *vram, int x, int y);
void putfont8(char *vram, int xsize, int x, int y, char c, char *font);

#define COL8_000000  0
#define COL8_FF0000  1
#define COL8_00FF00  2
#define COL8_FFFF00  3
#define COL8_0000FF  4
#define COL8_FF00FF  5
#define COL8_00FFFF  6
#define COL8_FFFFFF  7
#define COL8_C6C6C6  8
#define COL8_840000  9
#define COL8_008400  10
#define COL8_848400  11
#define COL8_000084  12
#define COL8_840084  13
#define COL8_008484  14
#define COL8_848484  15

struct BOOTINFO {
    char cyls, leds, vmode, reserve;
    short scrnx, scrny;
    char *vram;
};

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) 0x0ff0;
    static char font_A[16] = {
        0x00, 0x18, 0x18, 0x18, 0x24, 0x24, 0x24,
        0x24, 0x7e, 0x42, 0x42, 0xe7, 0x00, 0x00    
    };
 
    init_palette();
    init_screen(binfo->vram, binfo->scrnx, binfo->scrny);
    putfont8(binfo->vram, binfo->scrnx, 10, 10, COL8_FFFFFF, font_A);

    for (;;) {
        io_hlt();
    }
    
}

void init_palette(void)
{
    static unsigned char table_rgb[16 * 3] = {
        0x00, 0x00, 0x00,   /* 0:black */
        0xff, 0x00, 0x00,   /* 1:light red */
        0x00, 0xff, 0x00,   /* 2:light green */
        0xff, 0xff, 0x00,   /* 3:light yellow */
        0x00, 0x00, 0xff,   /* 4:light blue */
        0xff, 0x00, 0xff,   /* 5:light puple */
        0x00, 0xff, 0xff,   /* 6:light sky blue */
        0xff, 0xff, 0xff,   /* 7:white */
        0xc6, 0xc6, 0xc6,   /* 8:light grey */
        0x84, 0x00, 0x00,   /* 9:dark green */
        0x00, 0x84, 0x00,   /* 10:dark green */
        0x84, 0x84, 0x00,   /* 11:dark yellow */
        0x00, 0x00, 0x84,   /* 12:dark blue */
        0x84, 0x00, 0x84,   /* 13:dark puple */
        0x00, 0x84, 0x84,   /* 14:dark sky blue */
        0x84, 0x84, 0x84    /* 15:dark grey */
    };
    set_palette(0, 15, table_rgb);
    return;
}

void set_palette(int start, int end, unsigned char *rgb)
{
    int i, eflags;
    eflags = io_load_eflags();
    io_cli();
    io_out8(0x03c8, start);
    for (i = start; i <= end; i++) {
        io_out8(0x03c9, rgb[0] / 4);
        io_out8(0x03c9, rgb[1] / 4);
        io_out8(0x03c9, rgb[2] / 4);
        rgb += 3;
    }
    io_store_eflags(eflags);
    return;
}

void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1)
{
    int x, y;
    for (y= y0; y <= y1; y++) {
        for (x = x0; x <= x1; x++)
            vram[y * xsize + x] = c;
    }
    return;
}

void init_screen(char *vram, int x, int y)
{
    boxfill8(vram, x, COL8_008484,  0,          0,       x - 1,    y - 29);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 28,  x - 1,    y - 28);
    boxfill8(vram, x, COL8_FFFFFF,  0,          y - 27,  x - 1,    y - 27);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 26,  x - 1,    y -  1);

    boxfill8(vram, x, COL8_FFFFFF,  3,          y - 24,  59,       y - 24);
    boxfill8(vram, x, COL8_FFFFFF,  2,          y - 24,   2,       y -  4);
    boxfill8(vram, x, COL8_848484,  3,          y -  4,  59,       y -  4);
    boxfill8(vram, x, COL8_848484, 59,          y - 23,  59,       y -  5);
    boxfill8(vram, x, COL8_000000,  2,          y -  3,  59,       y -  3);
    boxfill8(vram, x, COL8_000000, 60,          y - 24,  60,       y -  3);

    boxfill8(vram, x, COL8_848484, x - 47,      y - 24,  x -  4,   y - 24);
    boxfill8(vram, x, COL8_848484, x - 47,      y - 23,  x - 47,   y -  4);
    boxfill8(vram, x, COL8_FFFFFF, x - 47,      y -  3,  x -  4,   y -  3);
    boxfill8(vram, x, COL8_FFFFFF, x -  3,      y - 24,  x -  3,   y -  3);
    return;
}

void putfont8(char *vram, int xsize, int x, int y, char c, char *font)
{
    int i;
    char *p, d;
    for (i = 0; i < 16; i++) {
        p = vram + (y + i) * xsize + x;
        d = font[i];
        if ((d & 0x80) != 0) { p[0] = c; }
        if ((d & 0x40) != 0) { p[1] = c; }
        if ((d & 0x20) != 0) { p[2] = c; }
        if ((d & 0x10) != 0) { p[3] = c; }
        if ((d & 0x08) != 0) { p[4] = c; }
        if ((d & 0x04) != 0) { p[5] = c; }
        if ((d & 0x02) != 0) { p[6] = c; }
        if ((d & 0x01) != 0) { p[7] = c; }
    }
    return ;
}
```

make runしてAが出ました！   
やりました！   

つぎは文字を大量に羅列したいのでhankaku.txtというファイルをします。これは大きなファイルのでここに記載しません。githubからダウンロードしてきてください。   
また、hankaku.txtをhankaku.objに変更するためにMakefileを書き換えます。   

```Makefile
TOOLPATH = ..\..\z_tools\\

INCPATH  = ..\..\z_tools\haribote\\

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
CC1		 = $(TOOLPATH)cc1.exe -I$(INCPATH) -Os -Wall -quiet
GAS2NASK = $(TOOLPATH)gas2nask.exe -a
OBJ2BIM  = $(TOOLPATH)obj2bim.exe
MAKEFONT = $(TOOLPATH)makefont.exe
BIN2OBJ  = $(TOOLPATH)bin2obj
BIM2HRB  = $(TOOLPATH)bim2hrb.exe
RULEFILE = $(TOOLPATH)haribote\haribote.rul
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f


# default operation
default:
	$(MAKE) img

# file making rule

ipl10.bin : ipl10.nas Makefile
	$(NASK) ipl10.nas ipl10.bin ipl10.lst

asmhead.bin : asmhead.nas Makefile
	$(NASK) asmhead.nas asmhead.bin asmhead.lst

bootpack.gas : bootpack.c Makefile
	$(CC1) -o bootpack.gas bootpack.c

bootpack.nas : bootpack.gas Makefile
	$(GAS2NASK) bootpack.gas  bootpack.nas 

bootpack.obj : bootpack.nas Makefile
	$(NASK) bootpack.nas bootpack.obj bootpack.lst

naskfunc.obj : naskfunc.nas Makefile
	$(NASK) naskfunc.nas naskfunc.obj naskfunc.lst

hankaku.bin : hankaku.txt Makefile
	$(MAKEFONT) hankaku.txt hankaku.bin

hankaku.obj : hankaku.bin Makefile
	$(BIN2OBJ) hankaku.bin hankaku.obj _hankaku

bootpack.bim : bootpack.obj naskfunc.obj hankaku.obj Makefile
	$(OBJ2BIM) @$(RULEFILE) out:bootpack.bim stack:3136k map:bootpack.map \
		bootpack.obj naskfunc.obj hankaku.obj
# 3MB+64KB=3136KB

bootpack.hrb : bootpack.bim Makefile
	$(BIM2HRB) bootpack.bim bootpack.hrb 0

haribote.sys : asmhead.bin bootpack.hrb Makefile
	cat asmhead.bin >  haribote.sys
	cat bootpack.hrb >> haribote.sys

haribote.img : ipl10.bin haribote.sys Makefile
	$(EDIMG)   imgin:..\..\z_tools\fdimg0at.tek \
		wbinimg src:ipl10.bin len:512 from:0 to:0 \
		copy from:haribote.sys to:@: \
		imgout:haribote.img

# command

img :
	$(MAKE) haribote.img

run :
	$(MAKE) img
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: haribote.img

clean :
	-$(DEL) *.bin
	-$(DEL) *.lst
	-$(DEL) *.gas
	-$(DEL) *.obj
	-$(DEL) bootpack.nas
	-$(DEL) bootpack.map
	-$(DEL) bootpack.bim
	-$(DEL) bootpack.hrb
	-$(DEL) haribote.sys

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img
```

ABC123と出力させるためにHariMainを書き換えます。
```c
void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) 0x0ff0;
    extern char hankaku[4096];
 
    init_palette();
    init_screen(binfo->vram, binfo->scrnx, binfo->scrny);
    putfont8(binfo->vram, binfo->scrnx,  8, 8, COL8_FFFFFF, hankaku + 'A' * 16);
    putfont8(binfo->vram, binfo->scrnx, 16, 8, COL8_FFFFFF, hankaku + 'B' * 16);
    putfont8(binfo->vram, binfo->scrnx, 24, 8, COL8_FFFFFF, hankaku + 'C' * 16);
    putfont8(binfo->vram, binfo->scrnx, 40, 8, COL8_FFFFFF, hankaku + '1' * 16);
    putfont8(binfo->vram, binfo->scrnx, 48, 8, COL8_FFFFFF, hankaku + '2' * 16);
    putfont8(binfo->vram, binfo->scrnx, 56, 8, COL8_FFFFFF, hankaku + '3' * 16);

    for (;;) {
        io_hlt();
    }
    
}
```

これでABC123が画面に出力されました！   
すごいすごい！   

いちいち一文字ずつ関数を呼び出すのはめんどくさいので新しく関数を定義してみます。   
```c
void io_hlt(void);
void io_cli(void);
void io_out8(int port, int data);
int io_load_eflags(void);
void io_store_eflags(int eflags);

void init_palette(void);
void set_palette(int start, int end, unsigned char *rgd);
void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1);
void init_screen(char *vram, int x, int y);
void putfont8(char *vram, int xsize, int x, int y, char c, char *font);
void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s);

#define COL8_000000  0
#define COL8_FF0000  1
#define COL8_00FF00  2
#define COL8_FFFF00  3
#define COL8_0000FF  4
#define COL8_FF00FF  5
#define COL8_00FFFF  6
#define COL8_FFFFFF  7
#define COL8_C6C6C6  8
#define COL8_840000  9
#define COL8_008400  10
#define COL8_848400  11
#define COL8_000084  12
#define COL8_840084  13
#define COL8_008484  14
#define COL8_848484  15

struct BOOTINFO {
    char cyls, leds, vmode, reserve;
    short scrnx, scrny;
    char *vram;
};

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) 0x0ff0;
    extern char hankaku[4096];
 
    init_palette();
    init_screen(binfo->vram, binfo->scrnx, binfo->scrny);
    putfonts8_asc(binfo->vram, binfo->scrnx,   8,  8, COL8_FFFFFF, "ABC 123");
    putfonts8_asc(binfo->vram, binfo->scrnx,  31, 31, COL8_000000, "Haribote OS.");
    putfonts8_asc(binfo->vram, binfo->scrnx,  30, 30, COL8_FFFFFF, "Haribote OS.");

    for (;;) {
        io_hlt();
    }
    
}

void init_palette(void)
{
    static unsigned char table_rgb[16 * 3] = {
        0x00, 0x00, 0x00,   /* 0:black */
        0xff, 0x00, 0x00,   /* 1:light red */
        0x00, 0xff, 0x00,   /* 2:light green */
        0xff, 0xff, 0x00,   /* 3:light yellow */
        0x00, 0x00, 0xff,   /* 4:light blue */
        0xff, 0x00, 0xff,   /* 5:light puple */
        0x00, 0xff, 0xff,   /* 6:light sky blue */
        0xff, 0xff, 0xff,   /* 7:white */
        0xc6, 0xc6, 0xc6,   /* 8:light grey */
        0x84, 0x00, 0x00,   /* 9:dark green */
        0x00, 0x84, 0x00,   /* 10:dark green */
        0x84, 0x84, 0x00,   /* 11:dark yellow */
        0x00, 0x00, 0x84,   /* 12:dark blue */
        0x84, 0x00, 0x84,   /* 13:dark puple */
        0x00, 0x84, 0x84,   /* 14:dark sky blue */
        0x84, 0x84, 0x84    /* 15:dark grey */
    };
    set_palette(0, 15, table_rgb);
    return;
}

void set_palette(int start, int end, unsigned char *rgb)
{
    int i, eflags;
    eflags = io_load_eflags();
    io_cli();
    io_out8(0x03c8, start);
    for (i = start; i <= end; i++) {
        io_out8(0x03c9, rgb[0] / 4);
        io_out8(0x03c9, rgb[1] / 4);
        io_out8(0x03c9, rgb[2] / 4);
        rgb += 3;
    }
    io_store_eflags(eflags);
    return;
}

void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1)
{
    int x, y;
    for (y= y0; y <= y1; y++) {
        for (x = x0; x <= x1; x++)
            vram[y * xsize + x] = c;
    }
    return;
}

void init_screen(char *vram, int x, int y)
{
    boxfill8(vram, x, COL8_008484,  0,          0,       x - 1,    y - 29);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 28,  x - 1,    y - 28);
    boxfill8(vram, x, COL8_FFFFFF,  0,          y - 27,  x - 1,    y - 27);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 26,  x - 1,    y -  1);

    boxfill8(vram, x, COL8_FFFFFF,  3,          y - 24,  59,       y - 24);
    boxfill8(vram, x, COL8_FFFFFF,  2,          y - 24,   2,       y -  4);
    boxfill8(vram, x, COL8_848484,  3,          y -  4,  59,       y -  4);
    boxfill8(vram, x, COL8_848484, 59,          y - 23,  59,       y -  5);
    boxfill8(vram, x, COL8_000000,  2,          y -  3,  59,       y -  3);
    boxfill8(vram, x, COL8_000000, 60,          y - 24,  60,       y -  3);

    boxfill8(vram, x, COL8_848484, x - 47,      y - 24,  x -  4,   y - 24);
    boxfill8(vram, x, COL8_848484, x - 47,      y - 23,  x - 47,   y -  4);
    boxfill8(vram, x, COL8_FFFFFF, x - 47,      y -  3,  x -  4,   y -  3);
    boxfill8(vram, x, COL8_FFFFFF, x -  3,      y - 24,  x -  3,   y -  3);
    return;
}

void putfont8(char *vram, int xsize, int x, int y, char c, char *font)
{
    int i;
    char *p, d;
    for (i = 0; i < 16; i++) {
        p = vram + (y + i) * xsize + x;
        d = font[i];
        if ((d & 0x80) != 0) { p[0] = c; }
        if ((d & 0x40) != 0) { p[1] = c; }
        if ((d & 0x20) != 0) { p[2] = c; }
        if ((d & 0x10) != 0) { p[3] = c; }
        if ((d & 0x08) != 0) { p[4] = c; }
        if ((d & 0x04) != 0) { p[5] = c; }
        if ((d & 0x02) != 0) { p[6] = c; }
        if ((d & 0x01) != 0) { p[7] = c; }
    }
    return ;
}

void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s)
{
    extern char hankaku[4096];
    for (; *s != 0x00; s++) {
        putfont8(vram, xsize, x, y, c, hankaku + *s * 16);
        x += 8;
    }
    return;
}
```

これで文字の出力が楽になりました。   

次は変数をみれるように改造します。   
sprintfを使います。   
```c
#include <stdio.h>

void io_hlt(void);
void io_cli(void);
void io_out8(int port, int data);
int io_load_eflags(void);
void io_store_eflags(int eflags);

void init_palette(void);
void set_palette(int start, int end, unsigned char *rgd);
void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1);
void init_screen(char *vram, int x, int y);
void putfont8(char *vram, int xsize, int x, int y, char c, char *font);
void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s);

#define COL8_000000  0
#define COL8_FF0000  1
#define COL8_00FF00  2
#define COL8_FFFF00  3
#define COL8_0000FF  4
#define COL8_FF00FF  5
#define COL8_00FFFF  6
#define COL8_FFFFFF  7
#define COL8_C6C6C6  8
#define COL8_840000  9
#define COL8_008400  10
#define COL8_848400  11
#define COL8_000084  12
#define COL8_840084  13
#define COL8_008484  14
#define COL8_848484  15

struct BOOTINFO {
    char cyls, leds, vmode, reserve;
    short scrnx, scrny;
    char *vram;
};

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) 0x0ff0;
    char s[40];
 
    init_palette();
    init_screen(binfo->vram, binfo->scrnx, binfo->scrny);
    putfonts8_asc(binfo->vram, binfo->scrnx,   8,  8, COL8_FFFFFF, "ABC 123");
    putfonts8_asc(binfo->vram, binfo->scrnx,  31, 31, COL8_000000, "Haribote OS.");
    putfonts8_asc(binfo->vram, binfo->scrnx,  30, 30, COL8_FFFFFF, "Haribote OS.");
    sprintf(s, "scrnx = %d", binfo->scrnx);
    putfonts8_asc(binfo->vram, binfo->scrnx, 16, 64, COL8_FFFFFF, s);

    for (;;) {
        io_hlt();
    }
    
}
```

次はマウスカーソルを表示させたいのでさらにソースコードに手を加えていきます。   
```c
#include <stdio.h>

void io_hlt(void);
void io_cli(void);
void io_out8(int port, int data);
int io_load_eflags(void);
void io_store_eflags(int eflags);

void init_palette(void);
void set_palette(int start, int end, unsigned char *rgd);
void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1);
void init_screen(char *vram, int x, int y);
void putfont8(char *vram, int xsize, int x, int y, char c, char *font);
void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s);
void init_mouse_cursor8(char *mouse, char bc);
void putblock8_8(char *vram, int vxsize, int pxsize,
    int pysize, int px0, int py0, char *buf, int bxsize);

#define COL8_000000  0
#define COL8_FF0000  1
#define COL8_00FF00  2
#define COL8_FFFF00  3
#define COL8_0000FF  4
#define COL8_FF00FF  5
#define COL8_00FFFF  6
#define COL8_FFFFFF  7
#define COL8_C6C6C6  8
#define COL8_840000  9
#define COL8_008400  10
#define COL8_848400  11
#define COL8_000084  12
#define COL8_840084  13
#define COL8_008484  14
#define COL8_848484  15

struct BOOTINFO {
    char cyls, leds, vmode, reserve;
    short scrnx, scrny;
    char *vram;
};

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) 0x0ff0;
    char s[40], mcursor[256];
    int mx, my;
 
    init_palette();
    init_screen(binfo->vram, binfo->scrnx, binfo->scrny);
    mx = (binfo->scrnx - 16) / 2;
    my = (binfo->scrny - 28 - 16) / 2;
    init_mouse_cursor8(mcursor, COL8_008484);
    putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
    sprintf(s, "(%d, %d)", mx, my);
    putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

    for (;;) {
        io_hlt();
    }
    
}

void init_palette(void)
{
    static unsigned char table_rgb[16 * 3] = {
        0x00, 0x00, 0x00,   /* 0:black */
        0xff, 0x00, 0x00,   /* 1:light red */
        0x00, 0xff, 0x00,   /* 2:light green */
        0xff, 0xff, 0x00,   /* 3:light yellow */
        0x00, 0x00, 0xff,   /* 4:light blue */
        0xff, 0x00, 0xff,   /* 5:light puple */
        0x00, 0xff, 0xff,   /* 6:light sky blue */
        0xff, 0xff, 0xff,   /* 7:white */
        0xc6, 0xc6, 0xc6,   /* 8:light grey */
        0x84, 0x00, 0x00,   /* 9:dark green */
        0x00, 0x84, 0x00,   /* 10:dark green */
        0x84, 0x84, 0x00,   /* 11:dark yellow */
        0x00, 0x00, 0x84,   /* 12:dark blue */
        0x84, 0x00, 0x84,   /* 13:dark puple */
        0x00, 0x84, 0x84,   /* 14:dark sky blue */
        0x84, 0x84, 0x84    /* 15:dark grey */
    };
    set_palette(0, 15, table_rgb);
    return;
}

void set_palette(int start, int end, unsigned char *rgb)
{
    int i, eflags;
    eflags = io_load_eflags();
    io_cli();
    io_out8(0x03c8, start);
    for (i = start; i <= end; i++) {
        io_out8(0x03c9, rgb[0] / 4);
        io_out8(0x03c9, rgb[1] / 4);
        io_out8(0x03c9, rgb[2] / 4);
        rgb += 3;
    }
    io_store_eflags(eflags);
    return;
}

void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1)
{
    int x, y;
    for (y= y0; y <= y1; y++) {
        for (x = x0; x <= x1; x++)
            vram[y * xsize + x] = c;
    }
    return;
}

void init_screen(char *vram, int x, int y)
{
    boxfill8(vram, x, COL8_008484,  0,          0,       x - 1,    y - 29);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 28,  x - 1,    y - 28);
    boxfill8(vram, x, COL8_FFFFFF,  0,          y - 27,  x - 1,    y - 27);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 26,  x - 1,    y -  1);

    boxfill8(vram, x, COL8_FFFFFF,  3,          y - 24,  59,       y - 24);
    boxfill8(vram, x, COL8_FFFFFF,  2,          y - 24,   2,       y -  4);
    boxfill8(vram, x, COL8_848484,  3,          y -  4,  59,       y -  4);
    boxfill8(vram, x, COL8_848484, 59,          y - 23,  59,       y -  5);
    boxfill8(vram, x, COL8_000000,  2,          y -  3,  59,       y -  3);
    boxfill8(vram, x, COL8_000000, 60,          y - 24,  60,       y -  3);

    boxfill8(vram, x, COL8_848484, x - 47,      y - 24,  x -  4,   y - 24);
    boxfill8(vram, x, COL8_848484, x - 47,      y - 23,  x - 47,   y -  4);
    boxfill8(vram, x, COL8_FFFFFF, x - 47,      y -  3,  x -  4,   y -  3);
    boxfill8(vram, x, COL8_FFFFFF, x -  3,      y - 24,  x -  3,   y -  3);
    return;
}

void putfont8(char *vram, int xsize, int x, int y, char c, char *font)
{
    int i;
    char *p, d;
    for (i = 0; i < 16; i++) {
        p = vram + (y + i) * xsize + x;
        d = font[i];
        if ((d & 0x80) != 0) { p[0] = c; }
        if ((d & 0x40) != 0) { p[1] = c; }
        if ((d & 0x20) != 0) { p[2] = c; }
        if ((d & 0x10) != 0) { p[3] = c; }
        if ((d & 0x08) != 0) { p[4] = c; }
        if ((d & 0x04) != 0) { p[5] = c; }
        if ((d & 0x02) != 0) { p[6] = c; }
        if ((d & 0x01) != 0) { p[7] = c; }
    }
    return ;
}

void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s)
{
    extern char hankaku[4096];
    for (; *s != 0x00; s++) {
        putfont8(vram, xsize, x, y, c, hankaku + *s * 16);
        x += 8;
    }
    return;
}

void init_mouse_cursor8(char *mouse, char bc)
{
    static char cursor[16][16] = {
        "**************..",
        "*OOOOOOOOOOO*...",
        "*OOOOOOOOOO*....",
        "*OOOOOOOOO*.....",
        "*OOOOOOOO*......",
        "*OOOOOO*........",
        "*OOOOOO*........",
        "*OOOOOOO*.......",
        "*OOOO**OOO*.....",
        "*OOO*..*OOO*....",
        "*OO*....*OOO*...",
        "*O*......*OOO*..",
        "**........*OOO*.",
        "*..........*OOO*",
        "............*OO*",
        ".............***"
    };
    int x, y;

    for (y = 0; y < 16; y++) {
        for (x = 0; x < 16; x++) {
            if (cursor[y][x] == '*') {
                mouse[y * 16 + x] = COL8_000000;
            }
            if (cursor[y][x] == 'O') {
                mouse[y + 16 + x] = COL8_FFFFFF;
            }
            if (cursor[y][x] == '.') {
                mouse[y * 16 + x] = bc;
            }
        }
    }
    return;
}

void putblock8_8(char *vram, int vxsize, int pxsize,
    int pysize, int px0, int py0, char *buf, int bxsize)
{
    int x, y;
    for (y = 0; y < pysize; y++) {
        for (x = 0; x < pxsize; x++) {
            vram[(py0 + y) * vxsize + (px0 + x)] = buf[y * bxsize + x];
        }
    }
    return; 
}
```

これでマウスカーソルが出力されました。

**実装過程**   
・　[917d43b0ce87e6d93f346fbf9b6b742aa09c6ecd](917d43b0ce87e6d93f346fbf9b6b742aa09c6ecd)

## day6
割り込み命令を作るために、GDT/IDTから作成していきます。   
bootpack.c
```c
struct SEGMENT_DESCRIPTOR {
    short limit_low, base_low;
    char base_mid, access_right;
    char limit_high, base_high;
};

struct GATE_DESCRIPTOR {
    short offset_low, selector;
    char dw_count, access_right;
    short offset_high;
};

void init_gdtidt(void);
void set_segmdesc(struct SEGMENT_DESCRIPTOR *sd, unsigned int limit, int base, int ar);
void set_gatedesc(struct GATE_DESCRIPTOR *gd, int offset, int selector, int ar);
void load_gdtr(int limit, int addr);
void load_idtr(int limit, int addr);
```

```c
void init_gdtidt(void) {
    struct SEGMENT_DESCRIPTOR *gdt = (struct SEGMENT_DESCRIPTOR *) 0x00270000;
    struct GATE_DESCRIPTOR *idt = (struct GATE_DESCRIPTOR *) 0x0026f800;
    int i;

    for (i = 0; i < 8192; i++) {
        set_segmdesc(gdt + i, 0, 0, 0);
    }

    set_segmdesc(gdt + 1, 0xffffffff, 0x00000000, 0x4092);
    set_segmdesc(gdt + 2, 0x0007ffff, 0x00280000, 0x409a);
    load_gdtr(0xffff, 0x00270000);

    for (i = 0; i < 256; i++) {
        set_gatedesc(idt + i, 0, 0, 0);
    }

    load_idtr(0x7ff, 0x0026f800);

    return;
}

void set_segmdesc(struct SEGMENT_DESCRIPTOR *sd, unsigned int limit, int base, int ar)
{
    if (limit > 0xffff) {
        ar |= 0x8000;
        limit /= 0x1000;
    }

    sd->limit_low = limit & 0xffff;
    sd->base_low = base & 0xffff;
    sd->base_mid = (base >> 16) & 0xff;
    sd->access_right = ar & 0xff;
    sd->limit_high = ((limit >> 16) & 0x0f) |  ((ar >> 8) & 0xf0);
    sd->base_high = (base >> 24) & 0xff;
    return;
}

void set_gatedesc(struct GATE_DESCRIPTOR *gd, int offset, int selector, int ar)
{
    gd->offset_low = offset & 0xffff;
    gd->selector = selector;
    gd->dw_count = (ar >> 8) & 0xff;
    gd->access_right = ar & 0xff;
    gd->offset_high = (offset >> 16) & 0xffff;
    return;
}
```

nasfunc.nas
```
GLOBAL  _load_gdtr, _load_idtr

_load_gdtr:
        MOV     AX,[ESP+4]
        MOV     [ESP+6],AX
        LGDT    [ESP+6]
        RET

_load_idtr:
        MOV     AX,[ESP+4]
        MOV     [ESP+6],AX
        LIDT    [ESP+6]
        RET
```

make runをしても前回と何も変わらないですが、これが6日目の布石になるみたいです。

ここからは6日目の内容になります。    
まずはbootpack.cが長すぎるのでファイルを分割しました。   
ついでにMakefileも改造しています。

bootpack.c
```c
TOOLPATH = ..\..\z_tools\\

INCPATH  = ..\..\z_tools\haribote\\

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
CC1		 = $(TOOLPATH)cc1.exe -I$(INCPATH) -Os -Wall -quiet
GAS2NASK = $(TOOLPATH)gas2nask.exe -a
OBJ2BIM  = $(TOOLPATH)obj2bim.exe
MAKEFONT = $(TOOLPATH)makefont.exe
BIN2OBJ  = $(TOOLPATH)bin2obj
BIM2HRB  = $(TOOLPATH)bim2hrb.exe
RULEFILE = $(TOOLPATH)haribote\haribote.rul
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f


# default operation
default:
	$(MAKE) img

# file making rule

ipl10.bin : ipl10.nas Makefile
	$(NASK) ipl10.nas ipl10.bin ipl10.lst

asmhead.bin : asmhead.nas Makefile
	$(NASK) asmhead.nas asmhead.bin asmhead.lst

bootpack.gas : bootpack.c Makefile
	$(CC1) -o bootpack.gas bootpack.c

bootpack.nas : bootpack.gas Makefile
	$(GAS2NASK) bootpack.gas  bootpack.nas 

bootpack.obj : bootpack.nas Makefile
	$(NASK) bootpack.nas bootpack.obj bootpack.lst

naskfunc.obj : naskfunc.nas Makefile
	$(NASK) naskfunc.nas naskfunc.obj naskfunc.lst

hankaku.bin : hankaku.txt Makefile
	$(MAKEFONT) hankaku.txt hankaku.bin

hankaku.obj : hankaku.bin Makefile
	$(BIN2OBJ) hankaku.bin hankaku.obj _hankaku

graphic.gas : graphic.c Makefile
	$(CC1) -o graphic.gas graphic.c

graphic.nas : graphic.gas Makefile
	$(GAS2NASK) graphic.gas graphic.nas

graphic.obj : graphic.nas Makefile
	$(NASK) graphic.nas graphic.obj graphic.lst

dsctbl.gas : dsctbl.c Makefile
	$(CC1) -o dsctbl.gas dsctbl.c

dsctbl.nas : dsctbl.gas Makefile
	$(GAS2NASK) dsctbl.gas dsctbl.nas

dsctbl.obj : dsctbl.nas Makefile
	$(NASK) dsctbl.nas dsctbl.obj dsctbl.lst

bootpack.bim : bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj Makefile
	$(OBJ2BIM) @$(RULEFILE) out:bootpack.bim stack:3136k map:bootpack.map \
		bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj
# 3MB+64KB=3136KB

bootpack.hrb : bootpack.bim Makefile
	$(BIM2HRB) bootpack.bim bootpack.hrb 0

haribote.sys : asmhead.bin bootpack.hrb Makefile
	cat asmhead.bin >  haribote.sys
	cat bootpack.hrb >> haribote.sys

haribote.img : ipl10.bin haribote.sys Makefile
	$(EDIMG)   imgin:..\..\z_tools\fdimg0at.tek \
		wbinimg src:ipl10.bin len:512 from:0 to:0 \
		copy from:haribote.sys to:@: \
		imgout:haribote.img

# command

img :
	$(MAKE) haribote.img

run :
	$(MAKE) img
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: haribote.img

clean :
	-$(DEL) *.bin
	-$(DEL) *.lst
	-$(DEL) *.gas
	-$(DEL) *.obj
	-$(DEL) bootpack.nas
	-$(DEL) bootpack.map
	-$(DEL) bootpack.bim
	-$(DEL) bootpack.hrb
	-$(DEL) haribote.sys

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img
```

graphic.c
```c
/* graphic process */

void io_hlt(void);
void io_cli(void);
void io_out8(int port, int data);
int io_load_eflags(void);
void io_store_eflags(int eflags);

void init_palette(void);
void set_palette(int start, int end, unsigned char *rgd);
void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1);
void init_screen(char *vram, int x, int y);
void putfont8(char *vram, int xsize, int x, int y, char c, char *font);
void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s);
void init_mouse_cursor8(char *mouse, char bc);
void putblock8_8(char *vram, int vxsize, int pxsize,
    int pysize, int px0, int py0, char *buf, int bxsize);

#define COL8_000000  0
#define COL8_FF0000  1
#define COL8_00FF00  2
#define COL8_FFFF00  3
#define COL8_0000FF  4
#define COL8_FF00FF  5
#define COL8_00FFFF  6
#define COL8_FFFFFF  7
#define COL8_C6C6C6  8
#define COL8_840000  9
#define COL8_008400  10
#define COL8_848400  11
#define COL8_000084  12
#define COL8_840084  13
#define COL8_008484  14
#define COL8_848484  15

void init_palette(void)
{
    static unsigned char table_rgb[16 * 3] = {
        0x00, 0x00, 0x00,   /* 0:black */
        0xff, 0x00, 0x00,   /* 1:light red */
        0x00, 0xff, 0x00,   /* 2:light green */
        0xff, 0xff, 0x00,   /* 3:light yellow */
        0x00, 0x00, 0xff,   /* 4:light blue */
        0xff, 0x00, 0xff,   /* 5:light puple */
        0x00, 0xff, 0xff,   /* 6:light sky blue */
        0xff, 0xff, 0xff,   /* 7:white */
        0xc6, 0xc6, 0xc6,   /* 8:light grey */
        0x84, 0x00, 0x00,   /* 9:dark green */
        0x00, 0x84, 0x00,   /* 10:dark green */
        0x84, 0x84, 0x00,   /* 11:dark yellow */
        0x00, 0x00, 0x84,   /* 12:dark blue */
        0x84, 0x00, 0x84,   /* 13:dark puple */
        0x00, 0x84, 0x84,   /* 14:dark sky blue */
        0x84, 0x84, 0x84    /* 15:dark grey */
    };
    set_palette(0, 15, table_rgb);
    return;
}

void set_palette(int start, int end, unsigned char *rgb)
{
    int i, eflags;
    eflags = io_load_eflags();
    io_cli();
    io_out8(0x03c8, start);
    for (i = start; i <= end; i++) {
        io_out8(0x03c9, rgb[0] / 4);
        io_out8(0x03c9, rgb[1] / 4);
        io_out8(0x03c9, rgb[2] / 4);
        rgb += 3;
    }
    io_store_eflags(eflags);
    return;
} 

void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1)
{
    int x, y;
    for (y= y0; y <= y1; y++) {
        for (x = x0; x <= x1; x++)
            vram[y * xsize + x] = c;
    }
    return;
}

void init_screen(char *vram, int x, int y)
{
    boxfill8(vram, x, COL8_008484,  0,          0,       x - 1,    y - 29);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 28,  x - 1,    y - 28);
    boxfill8(vram, x, COL8_FFFFFF,  0,          y - 27,  x - 1,    y - 27);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 26,  x - 1,    y -  1);

    boxfill8(vram, x, COL8_FFFFFF,  3,          y - 24,  59,       y - 24);
    boxfill8(vram, x, COL8_FFFFFF,  2,          y - 24,   2,       y -  4);
    boxfill8(vram, x, COL8_848484,  3,          y -  4,  59,       y -  4);
    boxfill8(vram, x, COL8_848484, 59,          y - 23,  59,       y -  5);
    boxfill8(vram, x, COL8_000000,  2,          y -  3,  59,       y -  3);
    boxfill8(vram, x, COL8_000000, 60,          y - 24,  60,       y -  3);

    boxfill8(vram, x, COL8_848484, x - 47,      y - 24,  x -  4,   y - 24);
    boxfill8(vram, x, COL8_848484, x - 47,      y - 23,  x - 47,   y -  4);
    boxfill8(vram, x, COL8_FFFFFF, x - 47,      y -  3,  x -  4,   y -  3);
    boxfill8(vram, x, COL8_FFFFFF, x -  3,      y - 24,  x -  3,   y -  3);
    return;
}

void putfont8(char *vram, int xsize, int x, int y, char c, char *font)
{
    int i;
    char *p, d;
    for (i = 0; i < 16; i++) {
        p = vram + (y + i) * xsize + x;
        d = font[i];
        if ((d & 0x80) != 0) { p[0] = c; }
        if ((d & 0x40) != 0) { p[1] = c; }
        if ((d & 0x20) != 0) { p[2] = c; }
        if ((d & 0x10) != 0) { p[3] = c; }
        if ((d & 0x08) != 0) { p[4] = c; }
        if ((d & 0x04) != 0) { p[5] = c; }
        if ((d & 0x02) != 0) { p[6] = c; }
        if ((d & 0x01) != 0) { p[7] = c; }
    }
    return ;
}

void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s)
{
    extern char hankaku[4096];
    for (; *s != 0x00; s++) {
        putfont8(vram, xsize, x, y, c, hankaku + *s * 16);
        x += 8;
    }
    return;
}

void init_mouse_cursor8(char *mouse, char bc)
{
    static char cursor[16][16] = {
        "**************..",
		"*OOOOOOOOOOO*...",
		"*OOOOOOOOOO*....",
		"*OOOOOOOOO*.....",
		"*OOOOOOOO*......",
		"*OOOOOOO*.......",
		"*OOOOOOO*.......",
		"*OOOOOOOO*......",
		"*OOOO**OOO*.....",
		"*OOO*..*OOO*....",
		"*OO*....*OOO*...",
		"*O*......*OOO*..",
		"**........*OOO*.",
		"*..........*OOO*",
		"............*OO*",
		".............***"
    };
    int x, y;

    for (y = 0; y < 16; y++) {
        for (x = 0; x < 16; x++) {
            if (cursor[y][x] == '*') {
                mouse[y * 16 + x] = COL8_000000;
            }
            if (cursor[y][x] == 'O') {
                mouse[y + 16 + x] = COL8_FFFFFF;
            }
            if (cursor[y][x] == '.') {
                mouse[y * 16 + x] = bc;
            }
        }
    }
    return;
}

void putblock8_8(char *vram, int vxsize, int pxsize,
    int pysize, int px0, int py0, char *buf, int bxsize)
{
    int x, y;
    for (y = 0; y < pysize; y++) {
        for (x = 0; x < pxsize; x++) {
            vram[(py0 + y) * vxsize + (px0 + x)] = buf[y * bxsize + x];
        }
    }
    return; 
}
```

dsctbl.c
```c
struct SEGMENT_DESCRIPTOR {
    short limit_low, base_low;
    char base_mid, access_right;
    char limit_high, base_high;
};

struct GATE_DESCRIPTOR {
    short offset_low, selector;
    char dw_count, access_right;
    short offset_high;
};

void init_gdtidt(void);
void set_segmdesc(struct SEGMENT_DESCRIPTOR *sd, unsigned int limit, int base, int ar);
void set_gatedesc(struct GATE_DESCRIPTOR *gd, int offset, int selector, int ar);
void load_gdtr(int limit, int addr);
void load_idtr(int limit, int addr);

void init_gdtidt(void) {
    struct SEGMENT_DESCRIPTOR *gdt = (struct SEGMENT_DESCRIPTOR *) 0x00270000;
    struct GATE_DESCRIPTOR *idt = (struct GATE_DESCRIPTOR *) 0x0026f800;
    int i;

    for (i = 0; i < 8192; i++) {
        set_segmdesc(gdt + i, 0, 0, 0);
    }

    set_segmdesc(gdt + 1, 0xffffffff, 0x00000000, 0x4092);
    set_segmdesc(gdt + 2, 0x0007ffff, 0x00280000, 0x409a);
    load_gdtr(0xffff, 0x00270000);

    for (i = 0; i < 256; i++) {
        set_gatedesc(idt + i, 0, 0, 0);
    }

    load_idtr(0x7ff, 0x0026f800);

    return;
}

void set_segmdesc(struct SEGMENT_DESCRIPTOR *sd, unsigned int limit, int base, int ar)
{
    if (limit > 0xffff) {
        ar |= 0x8000;
        limit /= 0x1000;
    }

    sd->limit_low = limit & 0xffff;
    sd->base_low = base & 0xffff;
    sd->base_mid = (base >> 16) & 0xff;
    sd->access_right = ar & 0xff;
    sd->limit_high = ((limit >> 16) & 0x0f) |  ((ar >> 8) & 0xf0);
    sd->base_high = (base >> 24) & 0xff;
    return;
}

void set_gatedesc(struct GATE_DESCRIPTOR *gd, int offset, int selector, int ar)
{
    gd->offset_low = offset & 0xffff;
    gd->selector = selector;
    gd->dw_count = (ar >> 8) & 0xff;
    gd->access_right = ar & 0xff;
    gd->offset_high = (offset >> 16) & 0xffff;
    return;
}
```

Makefile
```Makefile
TOOLPATH = ..\..\z_tools\\

INCPATH  = ..\..\z_tools\haribote\\

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
CC1		 = $(TOOLPATH)cc1.exe -I$(INCPATH) -Os -Wall -quiet
GAS2NASK = $(TOOLPATH)gas2nask.exe -a
OBJ2BIM  = $(TOOLPATH)obj2bim.exe
MAKEFONT = $(TOOLPATH)makefont.exe
BIN2OBJ  = $(TOOLPATH)bin2obj
BIM2HRB  = $(TOOLPATH)bim2hrb.exe
RULEFILE = $(TOOLPATH)haribote\haribote.rul
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f


# default operation
default:
	$(MAKE) img

# file making rule

ipl10.bin : ipl10.nas Makefile
	$(NASK) ipl10.nas ipl10.bin ipl10.lst

asmhead.bin : asmhead.nas Makefile
	$(NASK) asmhead.nas asmhead.bin asmhead.lst

bootpack.gas : bootpack.c Makefile
	$(CC1) -o bootpack.gas bootpack.c

bootpack.nas : bootpack.gas Makefile
	$(GAS2NASK) bootpack.gas  bootpack.nas 

bootpack.obj : bootpack.nas Makefile
	$(NASK) bootpack.nas bootpack.obj bootpack.lst

naskfunc.obj : naskfunc.nas Makefile
	$(NASK) naskfunc.nas naskfunc.obj naskfunc.lst

hankaku.bin : hankaku.txt Makefile
	$(MAKEFONT) hankaku.txt hankaku.bin

hankaku.obj : hankaku.bin Makefile
	$(BIN2OBJ) hankaku.bin hankaku.obj _hankaku

graphic.gas : graphic.c Makefile
	$(CC1) -o graphic.gas graphic.c

graphic.nas : graphic.gas Makefile
	$(GAS2NASK) graphic.gas graphic.nas

graphic.obj : graphic.nas Makefile
	$(NASK) graphic.nas graphic.obj graphic.lst

dsctbl.gas : dsctbl.c Makefile
	$(CC1) -o dsctbl.gas dsctbl.c

dsctbl.nas : dsctbl.gas Makefile
	$(GAS2NASK) dsctbl.gas dsctbl.nas

dsctbl.obj : dsctbl.nas Makefile
	$(NASK) dsctbl.nas dsctbl.obj dsctbl.lst

bootpack.bim : bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj Makefile
	$(OBJ2BIM) @$(RULEFILE) out:bootpack.bim stack:3136k map:bootpack.map \
		bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj
# 3MB+64KB=3136KB

bootpack.hrb : bootpack.bim Makefile
	$(BIM2HRB) bootpack.bim bootpack.hrb 0

haribote.sys : asmhead.bin bootpack.hrb Makefile
	cat asmhead.bin >  haribote.sys
	cat bootpack.hrb >> haribote.sys

haribote.img : ipl10.bin haribote.sys Makefile
	$(EDIMG)   imgin:..\..\z_tools\fdimg0at.tek \
		wbinimg src:ipl10.bin len:512 from:0 to:0 \
		copy from:haribote.sys to:@: \
		imgout:haribote.img

# command

img :
	$(MAKE) haribote.img

run :
	$(MAKE) img
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: haribote.img

clean :
	-$(DEL) *.bin
	-$(DEL) *.lst
	-$(DEL) *.gas
	-$(DEL) *.obj
	-$(DEL) bootpack.nas
	-$(DEL) bootpack.map
	-$(DEL) bootpack.bim
	-$(DEL) bootpack.hrb
	-$(DEL) haribote.sys

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img
```
Makefileが長くなりすぎたので短くします。

Makefile
```Makefile
OBJS_BOOTPACK = bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj

TOOLPATH = ..\..\z_tools\\

INCPATH  = ..\..\z_tools\haribote\\

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
CC1		 = $(TOOLPATH)cc1.exe -I$(INCPATH) -Os -Wall -quiet
GAS2NASK = $(TOOLPATH)gas2nask.exe -a
OBJ2BIM  = $(TOOLPATH)obj2bim.exe
MAKEFONT = $(TOOLPATH)makefont.exe
BIN2OBJ  = $(TOOLPATH)bin2obj
BIM2HRB  = $(TOOLPATH)bim2hrb.exe
RULEFILE = $(TOOLPATH)haribote\haribote.rul
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f


# default operation
default:
	$(MAKE) img

# file making rule

ipl10.bin : ipl10.nas Makefile
	$(NASK) ipl10.nas ipl10.bin ipl10.lst

asmhead.bin : asmhead.nas Makefile
	$(NASK) asmhead.nas asmhead.bin asmhead.lst

hankaku.bin : hankaku.txt Makefile
	$(MAKEFONT) hankaku.txt hankaku.bin

hankaku.obj : hankaku.bin Makefile
	$(BIN2OBJ) hankaku.bin hankaku.obj _hankaku

bootpack.bim : $(OBJS_BOOTPACK) Makefile
	$(OBJ2BIM) @$(RULEFILE) out:bootpack.bim stack:3136k map:bootpack.map \
		$(OBJS_BOOTPACK)
# 3MB+64KB=3136KB

bootpack.hrb : bootpack.bim Makefile
	$(BIM2HRB) bootpack.bim bootpack.hrb 0

haribote.sys : asmhead.bin bootpack.hrb Makefile
	cat asmhead.bin >  haribote.sys
	cat bootpack.hrb >> haribote.sys

haribote.img : ipl10.bin haribote.sys Makefile
	$(EDIMG)   imgin:..\..\z_tools\fdimg0at.tek \
		wbinimg src:ipl10.bin len:512 from:0 to:0 \
		copy from:haribote.sys to:@: \
		imgout:haribote.img

# rules
%.gas : %.c Makefile
	$(CC1) -o $*.gas $*.c

%.nas : %.gas Makefile
	$(GAS2NASK) $*.gas $*.nas

%.obj :%.nas Makefile
	$(NASK) $*.nas $*.obj $*.lst

# command

img :
	$(MAKE) haribote.img

run :
	$(MAKE) img
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) img
	$(IMGTOL) w a: haribote.img

clean :
	-$(DEL) *.bin
	-$(DEL) *.lst
	-$(DEL) *.gas
	-$(DEL) *.obj
	-$(DEL) bootpack.nas
	-$(DEL) bootpack.map
	-$(DEL) bootpack.bim
	-$(DEL) bootpack.hrb
	-$(DEL) haribote.sys

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img
```

これでだいぶ短くなりました。

さらにヘッダーファイルを記載していきます。   
bootpack.h
```h
/* asmhead.nad */
struct BOOTINFO {
    char cyls, leds, vmode, reserve;
    short scrnx, scrny;
    char *vram;
};
#define ADR_BOOTINFO

/* naskfunc.nas */
void io_hlt(void);
void io_cli(void);
void io_out8(int port, int data);
int io_load_eflags(void);
void io_store_eflags(int eflags);
void load_gdtr(int limit, int addr);
void load_idtr(int limit, int addr);

/* graphic.c */
void init_palette(void);
void set_palette(int start, int end, unsigned char *rgd);
void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1);
void init_screen(char *vram, int x, int y);
void putfont8(char *vram, int xsize, int x, int y, char c, char *font);
void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s);
void init_mouse_cursor8(char *mouse, char bc);
void putblock8_8(char *vram, int vxsize, int pxsize,
    int pysize, int px0, int py0, char *buf, int bxsize);

#define COL8_000000  0
#define COL8_FF0000  1
#define COL8_00FF00  2
#define COL8_FFFF00  3
#define COL8_0000FF  4
#define COL8_FF00FF  5
#define COL8_00FFFF  6
#define COL8_FFFFFF  7
#define COL8_C6C6C6  8
#define COL8_840000  9
#define COL8_008400  10
#define COL8_848400  11
#define COL8_000084  12
#define COL8_840084  13
#define COL8_008484  14
#define COL8_848484  15

/* dsctbl.c */
struct SEGMENT_DESCRIPTOR {
    short limit_low, base_low;
    char base_mid, access_right;
    char limit_high, base_high;
};

struct GATE_DESCRIPTOR {
    short offset_low, selector;
    char dw_count, access_right;
    short offset_high;
};

void init_gdtidt(void);
void set_segmdesc(struct SEGMENT_DESCRIPTOR *sd, unsigned int limit, int base, int ar);
void set_gatedesc(struct GATE_DESCRIPTOR *gd, int offset, int selector, int ar);
#define ADR_IDT         0x0026f800
#define LIMIT_IDT       0x000007ff
#define ADR_GDT         0x00270000
#define LIMIT_GDT       0x0000ffff
#define ADR_BOTPAK      0x00280000
#define LIMIT_BOTPAK    0x0007ffff
#define AR_DATA32_RW    0x4092
#define AR_CODE32_ER    0x409a
```

ヘッダーファイルを作ったことでCのソースファイルも短縮することができるようになりました。   
bootpack.c
```c
#include "bootpack.h"
#include <stdio.h>

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) 0x0ff0;
    char s[40], mcursor[256];
    int mx, my;
 
    init_palette();
    init_screen(binfo->vram, binfo->scrnx, binfo->scrny);
    mx = (binfo->scrnx - 16) / 2;
    my = (binfo->scrny - 28 - 16) / 2;
    init_mouse_cursor8(mcursor, COL8_008484);
    putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
    sprintf(s, "(%d, %d)", mx, my);
    putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

    for (;;) {
        io_hlt();
    }
    
}
```

graphic.c
```c
/* graphic process */
#include "bootpack.h"

void init_palette(void)
{
    static unsigned char table_rgb[16 * 3] = {
        0x00, 0x00, 0x00,   /* 0:black */
        0xff, 0x00, 0x00,   /* 1:light red */
        0x00, 0xff, 0x00,   /* 2:light green */
        0xff, 0xff, 0x00,   /* 3:light yellow */
        0x00, 0x00, 0xff,   /* 4:light blue */
        0xff, 0x00, 0xff,   /* 5:light puple */
        0x00, 0xff, 0xff,   /* 6:light sky blue */
        0xff, 0xff, 0xff,   /* 7:white */
        0xc6, 0xc6, 0xc6,   /* 8:light grey */
        0x84, 0x00, 0x00,   /* 9:dark green */
        0x00, 0x84, 0x00,   /* 10:dark green */
        0x84, 0x84, 0x00,   /* 11:dark yellow */
        0x00, 0x00, 0x84,   /* 12:dark blue */
        0x84, 0x00, 0x84,   /* 13:dark puple */
        0x00, 0x84, 0x84,   /* 14:dark sky blue */
        0x84, 0x84, 0x84    /* 15:dark grey */
    };
    set_palette(0, 15, table_rgb);
    return;
}

void set_palette(int start, int end, unsigned char *rgb)
{
    int i, eflags;
    eflags = io_load_eflags();
    io_cli();
    io_out8(0x03c8, start);
    for (i = start; i <= end; i++) {
        io_out8(0x03c9, rgb[0] / 4);
        io_out8(0x03c9, rgb[1] / 4);
        io_out8(0x03c9, rgb[2] / 4);
        rgb += 3;
    }
    io_store_eflags(eflags);
    return;
} 

void boxfill8(unsigned char *vram, int xsize, unsigned char c, int x0, int y0, int x1, int y1)
{
    int x, y;
    for (y= y0; y <= y1; y++) {
        for (x = x0; x <= x1; x++)
            vram[y * xsize + x] = c;
    }
    return;
}

void init_screen(char *vram, int x, int y)
{
    boxfill8(vram, x, COL8_008484,  0,          0,       x - 1,    y - 29);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 28,  x - 1,    y - 28);
    boxfill8(vram, x, COL8_FFFFFF,  0,          y - 27,  x - 1,    y - 27);
    boxfill8(vram, x, COL8_C6C6C6,  0,          y - 26,  x - 1,    y -  1);

    boxfill8(vram, x, COL8_FFFFFF,  3,          y - 24,  59,       y - 24);
    boxfill8(vram, x, COL8_FFFFFF,  2,          y - 24,   2,       y -  4);
    boxfill8(vram, x, COL8_848484,  3,          y -  4,  59,       y -  4);
    boxfill8(vram, x, COL8_848484, 59,          y - 23,  59,       y -  5);
    boxfill8(vram, x, COL8_000000,  2,          y -  3,  59,       y -  3);
    boxfill8(vram, x, COL8_000000, 60,          y - 24,  60,       y -  3);

    boxfill8(vram, x, COL8_848484, x - 47,      y - 24,  x -  4,   y - 24);
    boxfill8(vram, x, COL8_848484, x - 47,      y - 23,  x - 47,   y -  4);
    boxfill8(vram, x, COL8_FFFFFF, x - 47,      y -  3,  x -  4,   y -  3);
    boxfill8(vram, x, COL8_FFFFFF, x -  3,      y - 24,  x -  3,   y -  3);
    return;
}

void putfont8(char *vram, int xsize, int x, int y, char c, char *font)
{
    int i;
    char *p, d;
    for (i = 0; i < 16; i++) {
        p = vram + (y + i) * xsize + x;
        d = font[i];
        if ((d & 0x80) != 0) { p[0] = c; }
        if ((d & 0x40) != 0) { p[1] = c; }
        if ((d & 0x20) != 0) { p[2] = c; }
        if ((d & 0x10) != 0) { p[3] = c; }
        if ((d & 0x08) != 0) { p[4] = c; }
        if ((d & 0x04) != 0) { p[5] = c; }
        if ((d & 0x02) != 0) { p[6] = c; }
        if ((d & 0x01) != 0) { p[7] = c; }
    }
    return ;
}

void putfonts8_asc(char *vram, int xsize, int x, int y, char c, unsigned char *s)
{
    extern char hankaku[4096];
    for (; *s != 0x00; s++) {
        putfont8(vram, xsize, x, y, c, hankaku + *s * 16);
        x += 8;
    }
    return;
}

void init_mouse_cursor8(char *mouse, char bc)
{
    static char cursor[16][16] = {
        "**************..",
		"*OOOOOOOOOOO*...",
		"*OOOOOOOOOO*....",
		"*OOOOOOOOO*.....",
		"*OOOOOOOO*......",
		"*OOOOOOO*.......",
		"*OOOOOOO*.......",
		"*OOOOOOOO*......",
		"*OOOO**OOO*.....",
		"*OOO*..*OOO*....",
		"*OO*....*OOO*...",
		"*O*......*OOO*..",
		"**........*OOO*.",
		"*..........*OOO*",
		"............*OO*",
		".............***"
    };
    int x, y;

    for (y = 0; y < 16; y++) {
        for (x = 0; x < 16; x++) {
            if (cursor[y][x] == '*') {
                mouse[y * 16 + x] = COL8_000000;
            }
            if (cursor[y][x] == 'O') {
                mouse[y + 16 + x] = COL8_FFFFFF;
            }
            if (cursor[y][x] == '.') {
                mouse[y * 16 + x] = bc;
            }
        }
    }
    return;
}

void putblock8_8(char *vram, int vxsize, int pxsize,
    int pysize, int px0, int py0, char *buf, int bxsize)
{
    int x, y;
    for (y = 0; y < pysize; y++) {
        for (x = 0; x < pxsize; x++) {
            vram[(py0 + y) * vxsize + (px0 + x)] = buf[y * bxsize + x];
        }
    }
    return; 
}
```

dsctbl.c
```c
#include "bootpack.h"

void init_gdtidt(void) {
    struct SEGMENT_DESCRIPTOR *gdt = (struct SEGMENT_DESCRIPTOR *) 0x00270000;
    struct GATE_DESCRIPTOR *idt = (struct GATE_DESCRIPTOR *) 0x0026f800;
    int i;

    for (i = 0; i < 8192; i++) {
        set_segmdesc(gdt + i, 0, 0, 0);
    }

    set_segmdesc(gdt + 1, 0xffffffff, 0x00000000, 0x4092);
    set_segmdesc(gdt + 2, 0x0007ffff, 0x00280000, 0x409a);
    load_gdtr(0xffff, 0x00270000);

    for (i = 0; i < 256; i++) {
        set_gatedesc(idt + i, 0, 0, 0);
    }

    load_idtr(0x7ff, 0x0026f800);

    return;
}

void set_segmdesc(struct SEGMENT_DESCRIPTOR *sd, unsigned int limit, int base, int ar)
{
    if (limit > 0xffff) {
        ar |= 0x8000;
        limit /= 0x1000;
    }

    sd->limit_low = limit & 0xffff;
    sd->base_low = base & 0xffff;
    sd->base_mid = (base >> 16) & 0xff;
    sd->access_right = ar & 0xff;
    sd->limit_high = ((limit >> 16) & 0x0f) |  ((ar >> 8) & 0xf0);
    sd->base_high = (base >> 24) & 0xff;
    return;
}

void set_gatedesc(struct GATE_DESCRIPTOR *gd, int offset, int selector, int ar)
{
    gd->offset_low = offset & 0xffff;
    gd->selector = selector;
    gd->dw_count = (ar >> 8) & 0xff;
    gd->access_right = ar & 0xff;
    gd->offset_high = (offset >> 16) & 0xffff;
    return;
}
```

ヘッダーファイルを分割したことでさらに見やすくなりました。   

**実装過程**   
・　[6bf8c9f4729c76f7ab788aef9405847f1c7eb1cd](6bf8c9f4729c76f7ab788aef9405847f1c7eb1cd)

## day7
さらにマウスカーソルを動かす準備をしていきます。   
PICの初期化を実施します。   
int.c
```c
#include "bootpack.h"

void init_pic(void)
/* Initialize pic */
{
    io_out8(PIC0_IMR, 0xff );
    io_out8(PIC1_IMR, 0xff );

    io_out8(PIC0_ICW1, 0x11 );
    io_out8(PIC0_ICW2, 0x20 );
    io_out8(PIC0_ICW3, 1 << 2);
    io_out8(PIC0_ICW4, 0x01 );

    io_out8(PIC1_ICW1, 0x11 );
    io_out8(PIC1_ICW2, 0x28 );
    io_out8(PIC1_ICW3, 2 );
    io_out8(PIC1_ICW4, 0x01 );

    io_out8(PIC0_IMR, 0x0fb );
    io_out8(PIC1_IMR, 0xff );

    return;
}
```

bootpack.h
```h
/* int.c */
void ont_pic(void);
#define PIC0_ICW1       0x0020
#define PIC0_OCW2       0x0020
#define PIC0_IMR        0x0021
#define PIC0_ICW2       0x0021
#define PIC0_ICW3       0x0021
#define PIC0_ICW4       0x0021
#define PIC1_ICW1       0x00a0
#define PIC1_OCW2       0x00a0
#define PIC1_IMR        0x00a1
#define PIC1_ICW2       0x00a1
#define PIC1_ICW3       0x00a1
#define PIC1_ICW4       0x00a1
```

Makefile
```Makefile
OBJS_BOOTPACK = bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj \
				int.obj
```

動作させてみても何も変わりませんがこれでまた一つ下地ができました。   

次は割り込みハンドラを作成しました。   
正直一番苦労したかもしれません^^;   
色々と変更したので載せていきます。

int.c
```c
void inthandler21(int *esp)
/* PS/2 keybord interrupt */
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
    boxfill8(binfo->vram, binfo->scrnx, COL8_000000, 0, 0, 32 * 8 - 1, 15);
    putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, "INT 21 (IRQ-1) : PS/2 keyboard");
    for (;;) {
        io_hlt();
    }
}

void inthandler2c(int *esp)
/* PS/2 mouse interrupt */
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
    boxfill8(binfo->vram, binfo->scrnx, COL8_000000, 0, 0, 32 * 8 - 1, 15);
    putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, "INT 2C (IRQ-12) : PS/2 mouse");
    for (;;) {
        io_hlt();
    }
}

void inthandler27(int *esp)
/* PIC0からの不完全割り込み対策 */
/* Athlon64X2機などではチップセットの都合によりPICの初期化時にこの割り込みが1度だけおこる */
/* この割り込み処理関数は、その割り込みに対して何もしないでやり過ごす */
/* なぜ何もしなくていいの？
	→  この割り込みはPIC初期化時の電気的なノイズによって発生したものなので、
		まじめに何か処理してやる必要がない。									*/
{
    io_out8(PIC0_OCW2, 0x67);
    return;
}
```

dsctbl.c
```c
void init_gdtidt(void) {
    struct SEGMENT_DESCRIPTOR *gdt = (struct SEGMENT_DESCRIPTOR *) ADR_GDT;
    struct GATE_DESCRIPTOR *idt = (struct GATE_DESCRIPTOR *) ADR_IDT;
    int i;

    for (i = 0; i <= LIMIT_GDT / 8; i++) {
        set_segmdesc(gdt + i, 0, 0, 0);
    }

    set_segmdesc(gdt + 1, 0xffffffff, 0x00000000, AR_DATA32_RW);
    set_segmdesc(gdt + 2, LIMIT_BOTPAK, ADR_BOTPAK, AR_CODE32_ER);
    load_gdtr(LIMIT_GDT, ADR_GDT);

    for (i = 0; i <= LIMIT_IDT / 8; i++) {
        set_gatedesc(idt + i, 0, 0, 0);
    }
    load_idtr(LIMIT_IDT, ADR_IDT);

    /* setting IDT */
    set_gatedesc(idt + 0x21, (int) asm_inthandler21, 2 * 8, AR_INTGATE32);
    set_gatedesc(idt + 0x27, (int) asm_inthandler27, 2 * 8, AR_INTGATE32);
    set_gatedesc(idt + 0x2c, (int) asm_inthandler2c, 2 * 8, AR_INTGATE32);

    return;
}
```

naskfunc.nas
```asm
GLOBAL	_asm_inthandler21, _asm_inthandler27, _asm_inthandler2c
EXTERN	_inthandler21, _inthandler27, _inthandler2c
```

```asm
_asm_inthandler21:
		PUSH	ES
		PUSH	DS
		PUSHAD
		MOV		EAX,ESP
		PUSH	EAX
		MOV		AX,SS
		MOV		DS,AX
		MOV		ES,AX
		CALL	_inthandler21
		POP		EAX
		POPAD
		POP		DS
		POP		ES
		IRETD

_asm_inthandler27:
		PUSH	ES
		PUSH	DS
		PUSHAD
		MOV		EAX,ESP
		PUSH	EAX
		MOV		AX,SS
		MOV		DS,AX
		MOV		ES,AX
		CALL	_inthandler27
		POP		EAX
		POPAD
		POP		DS
		POP		ES
		IRETD   

_asm_inthandler2c:
		PUSH	ES
		PUSH	DS
		PUSHAD
		MOV		EAX,ESP
		PUSH	EAX
		MOV		AX,SS
		MOV		DS,AX
		MOV		ES,AX
		CALL	_inthandler2c
		POP		EAX
		POPAD
		POP		DS
		POP		ES
		IRETD
```

bootpack.h
```h
#define ADR_BOOTINFO    0x00000ff0
```

```h
/* naskfunc.nas */
void asm_inthandler21(void);
void asm_inthandler27(void);
void asm_inthandler2c(void);
```

```h
/* graphic.c */
void init_mouse_cursor8(char *mouse, char bc);
```

```h
/* int.c */
void inthandler21(int *esp);
void inthandler27(int *esp);
void inthandler2c(int *esp);
```

bootpack.c

```c
void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
    char s[40], mcursor[256];
    int mx, my;

    init_gdtidt();
    init_pic();
    io_sti();
 
    init_palette();
    init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
    mx = (binfo->scrnx - 16) / 2;
    my = (binfo->scrny - 28 - 16) / 2;
    init_mouse_cursor8(mcursor, COL8_008484);
    putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
    sprintf(s, "(%d, %d)", mx, my);
    putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

    io_out8(PIC0_IMR, 0xf9);
    io_out8(PIC1_IMR, 0xef);

    for (;;) {
        io_hlt();
    }
    
}
```

これでカーソルからの読み込みに対してメッセージを出すことができました。   

本では7日目に突入しました。   
キーコードを取得してみます。   
int.c
```c
#define PORT_KEYDAT  0x0060
```

```c
void inthandler21(int *esp)
/* PS/2 keybord interrupt */
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
    unsigned char data, s[4];
    io_out8(PIC0_OCW2, 0x61);
    data = io_in8(PORT_KEYDAT);
    
    sprintf(s, "%02x", data);
    boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
    putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
    
    return;
}
```

これで押したり話したりすると16進数でキーコードが返ってくるようになりました。   

次は割り込み処理を手早くするためにさらにそーそコードに改良を加えていきます。

bootpack.c
```c
extern struct KEYBUF keybuf;

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
    char s[40], mcursor[256];
    int mx, my, i;

    init_gdtidt();
    init_pic();
    io_sti();

    io_out8(PIC0_IMR, 0xf9);
    io_out8(PIC1_IMR, 0xef);
 
    init_palette();
    init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
    mx = (binfo->scrnx - 16) / 2;
    my = (binfo->scrny - 28 - 16) / 2;
    init_mouse_cursor8(mcursor, COL8_008484);
    putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
    sprintf(s, "(%d, %d)", mx, my);
    putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

    for (;;) {
        io_cli();
        if (keybuf.flag == 0) {
            io_stihlt();
        } else {
            i = keybuf.data;
            keybuf.flag = 0;
            io_sti();
            sprintf(s, "%02X", i);
            boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
            putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
        }
    }
    
}
```

int.c
```c
struct KEYBUF keybuf;
```

```c
void inthandler21(int *esp)
/* PS/2 keybord interrupt */
{
    unsigned char data;
    io_out8(PIC0_OCW2, 0x61);
    data = io_in8(PORT_KEYDAT);
    if (keybuf.flag == 0) {
        keybuf.data = data;
        keybuf.flag = 1;
    }
    return;
}
```

bootpack.h
```h
/* int.c */
struct KEYBUF {
    unsigned char data, flag;
};
```

これで少しは早くなりました。（最近のpcは性能がいいためほとんど体感的にはかわりませんが笑）

FIFOバッファを作成します。   
int.c
```c
void inthandler21(int *esp)
/* PS/2 keybord interrupt */
{
    unsigned char data;
    io_out8(PIC0_OCW2, 0x61);
    data = io_in8(PORT_KEYDAT);
    if (keybuf.next < 32) {
        keybuf.data[keybuf.next] = data;
        keybuf.next++;
    }
    return;
}
```

bootpack.h
```h
/* int.c */
struct KEYBUF {
    unsigned char data[32];
    int next;
};
```

bootpack.c
```c
void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
    char s[40], mcursor[256];
    int mx, my, i, j;

    init_gdtidt();
    init_pic();
    io_sti();

    io_out8(PIC0_IMR, 0xf9);
    io_out8(PIC1_IMR, 0xef);
 
    init_palette();
    init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
    mx = (binfo->scrnx - 16) / 2;
    my = (binfo->scrny - 28 - 16) / 2;
    init_mouse_cursor8(mcursor, COL8_008484);
    putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
    sprintf(s, "(%d, %d)", mx, my);
    putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

    for (;;) {
        io_cli();
        if (keybuf.next == 0) {
            io_stihlt();
        } else {
            i = keybuf.data[0];
            keybuf.next--;
            for (j = 0; j < keybuf.next; j++) {
                keybuf.data[j] = keybuf.data[j + 1];
            }
            io_sti();
            sprintf(s, "%02X", i);
            boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
            putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
        }
    }
    
}
```

挙動は変わりませんがこれでバッファが完成しました。  
バッファを改良していきたいと思います。   
これをやればさらに文字を早く入力できます。   
bootpack.h
```h
/* int.c */
struct KEYBUF {
    unsigned char data[32];
    int next_r, next_w, len;
};
```

int.c
```c
void inthandler21(int *esp)
/* PS/2 keybord interrupt */
{
    unsigned char data;
    io_out8(PIC0_OCW2, 0x61);
    data = io_in8(PORT_KEYDAT);
    if (keybuf.len < 32) {
        keybuf.data[keybuf.next_w] = data;
        keybuf.len++;
        keybuf.next_w++;
        if (keybuf.next_w == 32) {
            keybuf.next_w = 0;
        }
    }
    return;
}
```

bootpack.c
```c
void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
    char s[40], mcursor[256];
    int mx, my, i, j;

    init_gdtidt();
    init_pic();
    io_sti();

    io_out8(PIC0_IMR, 0xf9);
    io_out8(PIC1_IMR, 0xef);
 
    init_palette();
    init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
    mx = (binfo->scrnx - 16) / 2;
    my = (binfo->scrny - 28 - 16) / 2;
    init_mouse_cursor8(mcursor, COL8_008484);
    putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
    sprintf(s, "(%d, %d)", mx, my);
    putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

    for (;;) {
        io_cli();
        if (keybuf.len == 0) {
            io_stihlt();
        } else {
            i = keybuf.data[keybuf.next_r];
            keybuf.len--;
            keybuf.next_r++;
            if (keybuf.next_r == 32) {
                keybuf.next_r = 0;
            }
            io_sti();
            sprintf(s, "%02X", i);
            boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
            putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
        }
    }
    
}
```

さらに改良してfifoを別のソースコードに分けてあげます。   
fifo.c
```c
/* FIFOライブラリ */

#include "bootpack.h"

#define FLAGS_OVERRUN		0x0001

void fifo8_init(struct FIFO8 *fifo, int size, unsigned char *buf)
/* FIFOバッファの初期化 */
{
	fifo->size = size;
	fifo->buf = buf;
	fifo->free = size; /* 空き */
	fifo->flags = 0;
	fifo->p = 0; /* 書き込み位置 */
	fifo->q = 0; /* 読み込み位置 */
	return;
}

int fifo8_put(struct FIFO8 *fifo, unsigned char data)
/* FIFOへデータを送り込んで蓄える */
{
	if (fifo->free == 0) {
		/* 空きがなくてあふれた */
		fifo->flags |= FLAGS_OVERRUN;
		return -1;
	}
	fifo->buf[fifo->p] = data;
	fifo->p++;
	if (fifo->p == fifo->size) {
		fifo->p = 0;
	}
	fifo->free--;
	return 0;
}

int fifo8_get(struct FIFO8 *fifo)
/* FIFOからデータを一つとってくる */
{
	int data;
	if (fifo->free == fifo->size) {
		/* バッファが空っぽのときは、とりあえず-1が返される */
		return -1;
	}
	data = fifo->buf[fifo->q];
	fifo->q++;
	if (fifo->q == fifo->size) {
		fifo->q = 0;
	}
	fifo->free++;
	return data;
}

int fifo8_status(struct FIFO8 *fifo)
/* どのくらいデータが溜まっているかを報告する */
{
	return fifo->size - fifo->free;
}
```

bootpack.c
```h
/* fifo.c */
struct FIFO8 {
	unsigned char *buf;
	int p, q, size, free, flags;
};
void fifo8_init(struct FIFO8 *fifo, int size, unsigned char *buf);
int fifo8_put(struct FIFO8 *fifo, unsigned char data);
int fifo8_get(struct FIFO8 *fifo);
int fifo8_status(struct FIFO8 *fifo);
```

int.c
```c
struct FIFO8 keyfifo;
```

```c
void inthandler21(int *esp)
{
	unsigned char data;
	io_out8(PIC0_OCW2, 0x61);	/* IRQ-01受付完了をPICに通知 */
	data = io_in8(PORT_KEYDAT);
	fifo8_put(&keyfifo, data);
	return;
}
```

bootpack.c
```C
extern struct FIFO8 keyfifo;

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32];
	int mx, my, i;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */

	fifo8_init(&keyfifo, 32, keybuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) == 0) {
			io_stihlt();
		} else {
			i = fifo8_get(&keyfifo);
			io_sti();
			sprintf(s, "%02X", i);
			boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
			putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
		}
	}
}

```

Makefile
```Makefile
OBJS_BOOTPACK = bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj \
				int.obj fifo.obj
```

次にマウスの割り込みを感知させるようにしました。。   
bootpack.c
```c
#include "bootpack.h"
#include <stdio.h>

extern struct FIFO8 keyfifo;
void enable_mouse(void);
void init_keyboard(void);

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32];
	int mx, my, i;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	enable_mouse();

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) == 0) {
			io_stihlt();
		} else {
			i = fifo8_get(&keyfifo);
			io_sti();
			sprintf(s, "%02X", i);
			boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
			putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
		}
	}
}

#define PORT_KEYDAT			0x0060
#define PORT_KEYSTA			0x0064
#define PORAT_KEYCMD		0x0064
#define PORT_KEYCMD			0x0064
#define	KEYSTA_SEND_NOTREDY	0x02
#define KEYCMD_WRITE_MODE	0x60
#define KBC_MODE			0x47

void wait_KBC_sendready(void)
{
	/* キーボードコントローラーがデータ送信可能になるのを待つ */
	for (;;) {
		if ((io_in8(PORT_KEYSTA) & KEYSTA_SEND_NOTREDY) == 0) {
			break;
		}
	}
	return;
}


void init_keyboard(void)
{
	/* キーボードコントローラの初期化 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_WRITE_MODE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, KBC_MODE);
	return;
}

#define KEYCMD_SENDTO_MOUSE	0xd4
#define MOUSECMD_ENABLE		0xf4

void enable_mouse(void)
{
	/* マウス有効 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_SENDTO_MOUSE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, MOUSECMD_ENABLE);
	return; /* うまくいくとACK(0xfa)が送信されてくる */
}
```

マウスを感知したら次はデータを受信させてみます。   
int.c
```c
struct FIFO8 mousefifo;
```

```c
void inthandler2c(int *esp)
/* PS/2 mouse interrupt */
{
    unsigned char data;
    io_out8(PIC1_OCW2, 0x64);
    io_out8(PIC0_OCW2, 0x62);
    data = io_in8(PORT_KEYDAT);
    fifo8_put(&mousefifo, data);
    return;
}
```

bootpack.c
```c
extern struct FIFO8 keyfifo, mousefifo;
void enable_mouse(void);
void init_keyboard(void);

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	enable_mouse();

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 47, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
			}
		}
	}
}
```

これでデータを受信しました。   

ここからは本の8日目に突入です。   
初めはマウスを解読するために入力された3バイトを表示されます。    
bootpack.c
```c
void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned char mouse_dbuf[3], mouse_phase;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	enable_mouse();
	mouse_phase = 0; 

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_phase == 0) {
					/* マウスの0xfaを待っている段階 */
					if (i == 0xfa) {
						mouse_phase = 1;
					}
				} else if (mouse_phase == 1) {
					/* マウスの1バイト目を待っている段階 */
					mouse_dbuf[0] = i;
					mouse_phase = 2;
				} else if (mouse_phase == 2) {
					mouse_dbuf[1] = i;
					mouse_phase = 3;
				} else if (mouse_phase == 3) {
					/* マウスの3バイト目を待っている段階 */
					mouse_dbuf[2] = i;
					mouse_phase = 1;
					/* データが3バイト揃ったので表示 */
					sprintf(s, "%02X %02X %02X", mouse_dbuf[0], mouse_dbuf[1], mouse_dbuf[2]);
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 32 + 8 * 8 - 1, 31);
					putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
				}
			}
		}
	}
}
```

bootpack.cの中身のソースコードが多くなってみにくくなってきたので少し整理をしました。   
bootpack.c
```c
#include "bootpack.h"
#include <stdio.h>

struct MOUSE_DEC {
	unsigned char buf[3], phase;
};

extern struct FIFO8 keyfifo, mousefifo;
void enable_mouse(struct MOUSE_DEC *mdec);
void init_keyboard(void);
int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat);

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;
	struct MOUSE_DEC mdec;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	enable_mouse(&mdec);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "%02X %02X %02X", mdec.buf[0], mdec.buf[1], mdec.buf[2]);
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 32 + 8 * 8 - 1, 31);
					putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
				}
			}
		}
	}
}

#define PORT_KEYDAT			0x0060
#define PORT_KEYSTA			0x0064
#define PORAT_KEYCMD		0x0064
#define PORT_KEYCMD			0x0064
#define	KEYSTA_SEND_NOTREDY	0x02
#define KEYCMD_WRITE_MODE	0x60
#define KBC_MODE			0x47

void wait_KBC_sendready(void)
{
	/* キーボードコントローラーがデータ送信可能になるのを待つ */
	for (;;) {
		if ((io_in8(PORT_KEYSTA) & KEYSTA_SEND_NOTREDY) == 0) {
			break;
		}
	}
	return;
}


void init_keyboard(void)
{
	/* キーボードコントローラの初期化 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_WRITE_MODE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, KBC_MODE);
	return;
}

#define KEYCMD_SENDTO_MOUSE	0xd4
#define MOUSECMD_ENABLE		0xf4

void enable_mouse(struct MOUSE_DEC *mdec)
{
	/* マウス有効 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_SENDTO_MOUSE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, MOUSECMD_ENABLE);
	/* うまくいくとACK(0xfa)が送信されてくる */
	mdec->phase = 0; /* マウスの0xfaを待っている段階 */
	return; 
}

int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat)
{
	 if (mdec->phase == 0) {
		 /* マウスの0xfaを待っている段階 */
		 if (dat == 0xfa) {
			 mdec->phase = 1;
		 }
		 return 0;
	 }
	 if (mdec->phase == 1) {
		 /* マウスの1バイト目を待っている段階 */
		 mdec->buf[0] = dat;
		 mdec->phase = 2;
		 return 0;
	 }
	 if (mdec->phase == 2) {
		 /* マウスの2バイト目を待っている段階 */
		 mdec->buf[1] = dat;
		 mdec->phase = 3;
		 return 0;
	 }
	 if (mdec->phase == 3) {
		 /* マウスの3バイト目を待っている段階 */
		 mdec->buf[2] = dat;
		 mdec->phase = 1;
		 return 1;
	 }
	 return -1; /* ここに来ることはないはず */
}
```

これで少しはコードが見やすくなりました。   
さらにコードを解析するためにbootpack.cのソースコードをいじりました。   
移動情報とボタン状態を表示するようにしています。   
bootpack.c
```c
#include "bootpack.h"
#include <stdio.h>

struct MOUSE_DEC {
	unsigned char buf[3], phase;
	int x, y, btn;
};

extern struct FIFO8 keyfifo, mousefifo;
void enable_mouse(struct MOUSE_DEC *mdec);
void init_keyboard(void);
int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat);

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;
	struct MOUSE_DEC mdec;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	enable_mouse(&mdec);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d%4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2]= 'c';
					}
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
				}
			}
		}
	}
}

#define PORT_KEYDAT			0x0060
#define PORT_KEYSTA			0x0064
#define PORAT_KEYCMD		0x0064
#define PORT_KEYCMD			0x0064
#define	KEYSTA_SEND_NOTREDY	0x02
#define KEYCMD_WRITE_MODE	0x60
#define KBC_MODE			0x47

void wait_KBC_sendready(void)
{
	/* キーボードコントローラーがデータ送信可能になるのを待つ */
	for (;;) {
		if ((io_in8(PORT_KEYSTA) & KEYSTA_SEND_NOTREDY) == 0) {
			break;
		}
	}
	return;
}


void init_keyboard(void)
{
	/* キーボードコントローラの初期化 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_WRITE_MODE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, KBC_MODE);
	return;
}

#define KEYCMD_SENDTO_MOUSE	0xd4
#define MOUSECMD_ENABLE		0xf4

void enable_mouse(struct MOUSE_DEC *mdec)
{
	/* マウス有効 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_SENDTO_MOUSE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, MOUSECMD_ENABLE);
	/* うまくいくとACK(0xfa)が送信されてくる */
	mdec->phase = 0; /* マウスの0xfaを待っている段階 */
	return; 
}

int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat)
{
	 if (mdec->phase == 0) {
		 /* マウスの0xfaを待っている段階 */
		 if (dat == 0xfa) {
			 mdec->phase = 1;
		 }
		 return 0;
	 }
	 if (mdec->phase == 1) {
		 /* マウスの1バイト目を待っている段階 */
		 if ((dat & 0xc8) == 0x08) {
			/* 正しい1バイト目だった */
			mdec->buf[0] = dat;
		 	mdec->phase = 2;
		 }
		 return 0;
	 }
	 if (mdec->phase == 2) {
		 /* マウスの2バイト目を待っている段階 */
		 mdec->buf[1] = dat;
		 mdec->phase = 3;
		 return 0;
	 }
	 if (mdec->phase == 3) {
		 /* マウスの3バイト目を待っている段階 */
		 mdec->buf[2] = dat;
		 mdec->phase = 1;
		 mdec->btn = mdec->buf[0] & 0x07;
		 mdec->x = mdec->buf[1];
		 mdec->y = mdec->buf[2];
		 if ((mdec->buf[0] & 0x10) != 0) {
			 mdec->x |= 0xffffff00;
		 }
		 if ((mdec->buf[0] & 0x20) != 0) {
			 mdec->y |= 0xffffff00;
		 }
		 mdec->y = - mdec->y; /* マウスではy方向の符号が画面と反対 */
		 return 1;
	 }
	 return -1; /* ここに来ることはないはず */
}
```

**実装過程**   
・　[2dc2eed40057daaf4aa84bc00f02a949565165fc](2dc2eed40057daaf4aa84bc00f02a949565165fc)

## day8
開発初めて8日目になりました。   
1週間立ちましたね。   
今日はマウスカーソルを動かすところから作っていきたいと思います。   

bootpack.c
```c
void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;
	struct MOUSE_DEC mdec;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	enable_mouse(&mdec);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d%4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2]= 'c';
					}
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					/* マウスカーソルの移動 */
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, mx, my, mx + 15, my + 15);
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 16) {
						mx = binfo->scrnx - 16;
					}
					if (my > binfo->scrny - 16) {
						my = binfo->scrny - 16;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 0, 79, 15);
					putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
					putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
				}
			}
		}
	}
}
```

これでマウスが動くようになりました。   
かっこいいですね！   

書籍の9日目に突入しました。   
まずはbootpack.cとint.cを整理する所からスタートです。   
整理することで新しくkeybord.cとmouse.cが出来ました。

keybord.c
```c
#include "bootpack.h"

struct FIFO8 keyfifo;

#define PORT_KEYSTA			0x0064
#define	KEYSTA_SEND_NOTREDY	0x02
#define KEYCMD_WRITE_MODE	0x60
#define KBC_MODE			0x47

void wait_KBC_sendready(void)
{
	/* キーボードコントローラーがデータ送信可能になるのを待つ */
	for (;;) {
		if ((io_in8(PORT_KEYSTA) & KEYSTA_SEND_NOTREDY) == 0) {
			break;
		}
	}
	return;
}


void init_keyboard(void)
{
	/* キーボードコントローラの初期化 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_WRITE_MODE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, KBC_MODE);
	return;
}

void inthandler21(int *esp)
{
	unsigned char data;
	io_out8(PIC0_OCW2, 0x61);	/* IRQ-01受付完了をPICに通知 */
	data = io_in8(PORT_KEYDAT);
	fifo8_put(&keyfifo, data);
	return;
}
```

mouse.c
```c
#include "bootpack.h"

struct FIFO8    mousefifo;

#define KEYCMD_SENDTO_MOUSE	0xd4
#define MOUSECMD_ENABLE		0xf4

void enable_mouse(struct MOUSE_DEC *mdec)
{
	/* マウス有効 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_SENDTO_MOUSE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, MOUSECMD_ENABLE);
	/* うまくいくとACK(0xfa)が送信されてくる */
	mdec->phase = 0; /* マウスの0xfaを待っている段階 */
	return; 
}

int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat)
{
	 if (mdec->phase == 0) {
		 /* マウスの0xfaを待っている段階 */
		 if (dat == 0xfa) {
			 mdec->phase = 1;
		 }
		 return 0;
	 }
	 if (mdec->phase == 1) {
		 /* マウスの1バイト目を待っている段階 */
		 if ((dat & 0xc8) == 0x08) {
			/* 正しい1バイト目だった */
			mdec->buf[0] = dat;
		 	mdec->phase = 2;
		 }
		 return 0;
	 }
	 if (mdec->phase == 2) {
		 /* マウスの2バイト目を待っている段階 */
		 mdec->buf[1] = dat;
		 mdec->phase = 3;
		 return 0;
	 }
	 if (mdec->phase == 3) {
		 /* マウスの3バイト目を待っている段階 */
		 mdec->buf[2] = dat;
		 mdec->phase = 1;
		 mdec->btn = mdec->buf[0] & 0x07;
		 mdec->x = mdec->buf[1];
		 mdec->y = mdec->buf[2];
		 if ((mdec->buf[0] & 0x10) != 0) {
			 mdec->x |= 0xffffff00;
		 }
		 if ((mdec->buf[0] & 0x20) != 0) {
			 mdec->y |= 0xffffff00;
		 }
		 mdec->y = - mdec->y; /* マウスではy方向の符号が画面と反対 */
		 return 1;
	 }
	 return -1; /* ここに来ることはないはず */
}

void inthandler2c(int *esp)
/* PS/2 mouse interrupt */
{
    unsigned char data;
    io_out8(PIC1_OCW2, 0x64);
    io_out8(PIC0_OCW2, 0x62);
    data = io_in8(PORT_KEYDAT);
    fifo8_put(&mousefifo, data);
    return;
}
```

bootpack.c
```c
#include "bootpack.h"
#include <stdio.h>

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;
	struct MOUSE_DEC mdec;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	enable_mouse(&mdec);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d%4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2]= 'c';
					}
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					/* マウスカーソルの移動 */
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, mx, my, mx + 15, my + 15);
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 16) {
						mx = binfo->scrnx - 16;
					}
					if (my > binfo->scrny - 16) {
						my = binfo->scrny - 16;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 0, 79, 15);
					putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
					putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
				}
			}
		}
	}
}
```

int.c
```c
#include "bootpack.h"

void init_pic(void)
/* Initialize pic */
{
    io_out8(PIC0_IMR, 0xff );
    io_out8(PIC1_IMR, 0xff );

    io_out8(PIC0_ICW1, 0x11 );
    io_out8(PIC0_ICW2, 0x20 );
    io_out8(PIC0_ICW3, 1 << 2);
    io_out8(PIC0_ICW4, 0x01 );

    io_out8(PIC1_ICW1, 0x11 );
    io_out8(PIC1_ICW2, 0x28 );
    io_out8(PIC1_ICW3, 2 );
    io_out8(PIC1_ICW4, 0x01 );

    io_out8(PIC0_IMR, 0x0fb );
    io_out8(PIC1_IMR, 0xff );

    return;
}

void inthandler27(int *esp)
/* PIC0からの不完全割り込み対策 */
/* Athlon64X2機などではチップセットの都合によりPICの初期化時にこの割り込みが1度だけおこる */
/* この割り込み処理関数は、その割り込みに対して何もしないでやり過ごす */
/* なぜ何もしなくていいの？
	→  この割り込みはPIC初期化時の電気的なノイズによって発生したものなので、
		まじめに何か処理してやる必要がない。									*/
{
    io_out8(PIC0_OCW2, 0x67);
    return;
}
```

bootpack.h
```h
/* keybord.c */
void inthandler21(int *esp);
void wait_KBC_sendready(void);
void init_keyboard(void);
extern struct FIFO8 keyfifo;
#define PORT_KEYDAT			0x0060
#define PORT_KEYCMD			0x0064

/* mouse.c */
struct MOUSE_DEC {
	unsigned char buf[3], phase;
	int x, y, btn;
};
void inthandler2c(int *esp);
void enable_mouse(struct MOUSE_DEC *mdec);
int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat);
extern struct FIFO8 mousefifo;
```

Makefile
```Makefile
OBJS_BOOTPACK = bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj \
				int.obj fifo.obj keybord.obj mouse.obj
```

メモリ容量をチェックするためにソースコードに加筆していきます。   

bootpack.c
```c
#include "bootpack.h"
#include <stdio.h>

unsigned int memtest(unsigned int start, unsigned int end);
unsigned int memtest_sub(unsigned int start, unsigned int end);

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;
	struct MOUSE_DEC mdec;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	i = memtest(0x00400000, 0xbfffffff);
	sprintf(s, "memory %dMB", i);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 32, COL8_FFFFFF, s);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2]= 'c';
					}
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					/* マウスカーソルの移動 */
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, mx, my, mx + 15, my + 15);
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 16) {
						mx = binfo->scrnx - 16;
					}
					if (my > binfo->scrny - 16) {
						my = binfo->scrny - 16;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 0, 79, 15);
					putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
					putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
				}
			}
		}
	}
}

#define EFLAGS_AC_BIT		0x00040000
#define CR0_CACHE_DISABLE	0x60000000

unsigned int memtest(unsigned int start, unsigned int end)
{
	char flag486 = 0;
	unsigned int eflg, cr0, i;

	/* 386か、486以降なのかの確認 */
	eflg = io_load_eflags();
	eflg |= EFLAGS_AC_BIT; /* AC-bit = 1 */
	io_store_eflags(eflg);
	eflg = io_load_eflags();
	if ((eflg & EFLAGS_AC_BIT) != 0) { /* 386ではAC=1にしても自動で0に戻ってしまう */
		flag486 = 1;
	}
	eflg &= ~EFLAGS_AC_BIT; /* AC-bit = 0 */
	io_store_eflags(eflg);

	if (flag486 != 0) {
		cr0 = load_cr0();
		cr0 |= CR0_CACHE_DISABLE; /* キャッシュ禁止 */
		store_cr0(cr0);
	}

	i = memtest_sub(start, end);

	if (flag486 != 0) {
		cr0 = load_cr0();
		cr0 &= ~CR0_CACHE_DISABLE; /* キャッシュ許可 */
		store_cr0(cr0);
	}

	return i;
}

unsigned int memtest_sub(unsigned int start, unsigned int end)
{
	unsigned int i, *p, old, pat0 = 0xaa55aa55, pat1 = 0x55aa55aa;
	for (i = start; i <= end; i += 0x1000) {
		p = (unsigned int *) (i + 0xffc);
		old = *p;			/* いじる前の値を覚えておく */
		*p = pat0;			/* ためしに書いてみる */
		*p ^= 0xffffffff;	/* そしてそれを反転してみる */
		if (*p != pat1) {	/* 反転結果になったか？ */
not_memory:	
			*p = old;
			break;
		}
		*p ^= 0xffffffff;	/* もう一度反転してみる */
		if (*p != pat0) {	/* 元に戻ったか？ */
			goto not_memory;
		}
		*p = old;			/* いじった値を元に戻す */
	}
	return i;
}
```

naskfunc
```
GLOBAL  _load_cr0, _store_cr0
```

```
_load_cr0:		; int load_cr0(void);
		MOV		EAX,CR0
		RET

_store_cr0:		; void store_cr0(int cr0);
		MOV		EAX,[ESP+4]
		MOV		CR0,EAX
		RET
```

bootpack.h
```h
/* naskfunc.nas */
int load_cr0(void);
void store_cr0(int cr0);
```

このままだとメモリの部分がバグったままなのでさらに書き換えることにします。   

naskfunc.nas
```
GLOBAL  _memtest_sub
```

```
_memtest_sub:	; unsigned int memtest_sub(unsigned int start, unsigned int end)
		PUSH	EDI						; （EBX, ESI, EDI も使いたいので）
		PUSH	ESI
		PUSH	EBX
		MOV		ESI,0xaa55aa55			; pat0 = 0xaa55aa55;
		MOV		EDI,0x55aa55aa			; pat1 = 0x55aa55aa;
		MOV		EAX,[ESP+12+4]			; i = start;
mts_loop:
		MOV		EBX,EAX
		ADD		EBX,0xffc				; p = i + 0xffc;
		MOV		EDX,[EBX]				; old = *p;
		MOV		[EBX],ESI				; *p = pat0;
		XOR		DWORD [EBX],0xffffffff	; *p ^= 0xffffffff;
		CMP		EDI,[EBX]				; if (*p != pat1) goto fin;
		JNE		mts_fin
		XOR		DWORD [EBX],0xffffffff	; *p ^= 0xffffffff;
		CMP		ESI,[EBX]				; if (*p != pat0) goto fin;
		JNE		mts_fin
		MOV		[EBX],EDX				; *p = old;
		ADD		EAX,0x1000				; i += 0x1000;
		CMP		EAX,[ESP+12+8]			; if (i <= end) goto mts_loop;
		JBE		mts_loop
		POP		EBX
		POP		ESI
		POP		EDI
		RET
mts_fin:
		MOV		[EBX],EDX				; *p = old;
		POP		EBX
		POP		ESI
		POP		EDI
		RET
```

bootpack.c
```c
void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;
	struct MOUSE_DEC mdec;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	i = memtest(0x00400000, 0xbfffffff) / (1024 * 1024);
	sprintf(s, "memory %dMB", i);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 32, COL8_FFFFFF, s);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2]= 'c';
					}
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					/* マウスカーソルの移動 */
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, mx, my, mx + 15, my + 15);
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 16) {
						mx = binfo->scrnx - 16;
					}
					if (my > binfo->scrny - 16) {
						my = binfo->scrny - 16;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 0, 79, 15);
					putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
					putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
				}
			}
		}
	}
}
```

bootpack.h
```h
/* naskfunc.nas */
unsigned int memtest_sub(unsigned int start, unsigned int end);
```

naskfunc.nasにmemtest_subを記載したので、bootpack.cのmemtest_subは消してください。   
これでメモリの値がバグらなくなりました。

メモリの空き量も知りたいので追加することにします。

bootpack.c
```c
#include "bootpack.h"
#include <stdio.h>

#define MEMMAN_FREES		4090
#define MEMMAN_ADDR			0x003c0000

struct FREEINFO { 	/* あき情報 */
	unsigned int addr, size;
};

struct MEMMAN {		/* メモリ管理 */
	int frees, maxfrees, lostsize, losts;
	struct FREEINFO free[MEMMAN_FREES];
};

unsigned int memtest(unsigned int start, unsigned int end);
void memman_init(struct MEMMAN *man);
unsigned int memman_total(struct MEMMAN *man);
unsigned int memman_alloc(struct MEMMAN *man, unsigned int siza);
int memman_free(struct MEMMAN *man, unsigned int addr, unsigned int size);

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x000010000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	sprintf(s, "memory %dMB free : %dKB", 
			memtotal / (1024 * 1024), memman_total(memman) /1024);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 32, COL8_FFFFFF, s);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2]= 'c';
					}
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					/* マウスカーソルの移動 */
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, mx, my, mx + 15, my + 15);
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 16) {
						mx = binfo->scrnx - 16;
					}
					if (my > binfo->scrny - 16) {
						my = binfo->scrny - 16;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 0, 79, 15);
					putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
					putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
				}
			}
		}
	}
}

#define EFLAGS_AC_BIT		0x00040000
#define CR0_CACHE_DISABLE	0x60000000

unsigned int memtest(unsigned int start, unsigned int end)
{
	char flag486 = 0;
	unsigned int eflg, cr0, i;

	/* 386か、486以降なのかの確認 */
	eflg = io_load_eflags();
	eflg |= EFLAGS_AC_BIT; /* AC-bit = 1 */
	io_store_eflags(eflg);
	eflg = io_load_eflags();
	if ((eflg & EFLAGS_AC_BIT) != 0) { /* 386ではAC=1にしても自動で0に戻ってしまう */
		flag486 = 1;
	}
	eflg &= ~EFLAGS_AC_BIT; /* AC-bit = 0 */
	io_store_eflags(eflg);

	if (flag486 != 0) {
		cr0 = load_cr0();
		cr0 |= CR0_CACHE_DISABLE; /* キャッシュ禁止 */
		store_cr0(cr0);
	}

	i = memtest_sub(start, end);

	if (flag486 != 0) {
		cr0 = load_cr0();
		cr0 &= ~CR0_CACHE_DISABLE; /* キャッシュ許可 */
		store_cr0(cr0);
	}

	return i;
}

void memman_init(struct MEMMAN *man)
{
	man->frees = 0;			/* あき情報の個数 */
	man->maxfrees = 0;		/* 状況観察用：freesの最大値 */
	man->lostsize = 0;		/* 解放に失敗した合計サイズ */
	man->losts = 0;			/* 解放に失敗した回数 */
	return;
}


unsigned int memman_total(struct MEMMAN *man)
/* あきサイズの合計を報告 */
{
	unsigned int i, t = 0;
	for (i = 0; i < man->frees; i++) {
		t += man->free[i].size;
	}
	return t;
}

unsigned int memman_alloc(struct MEMMAN *man, unsigned int size)
/* 確保 */
{
	unsigned int i, a;
	for (i = 0; i< man->frees; i++) {
		if (man->free[i].size >= size) {
			/* 十分な広さのあきを発見 */
			a = man->free[i].addr;
			man->free[i].addr += size;
			man->free[i].size -= size;
			if (man->free[i].size == 0) {
				/* free[i]がなくなったので前へつめる */
				man->frees--;
				for (; i < man->frees; i++) {
					man->free[i] = man->free[i+1]; /* 構造体の代入 */
				}
			}
			return 0;
		}
	}
	return 0; /* あきがない */
}

int memman_free(struct MEMMAN *man, unsigned int addr, unsigned int size) 
/* 解放 */
{
	int i, j;
	/* まとめやすさを考えると、free[]がaddr順に並んでいるほうがいい */
	/* だからまず、どこに入れるべきかを決める */
	for (i = 0; i < man->frees; i++) {
		if (man->free[i].addr > addr) {
			break;
		}
	}
	/* free[i - 1].addr < addr < free[i].addr */
	if (i > 0) {
		/* 前がある */
		if (man->free[i - 1].addr + man->free[i - i].size == addr) {
			/* 前のあき領域にまとめられる */
			man->free[i - 1].size += size;
			if (i < man->frees) {
				/* 後ろもある */
				if (addr + size == man->free[i].size) {
					/* なんと後ろともまとめられる */
					man->free[i - 1].size += man->free[i].size;
					/* man->free[i]の削除 */
					/* free[i]がなくなったので前へつめる */
					man->frees--;
					for (; i < man->frees; i++) {
						man->free[i] = man->free[i + 1]; /* 構造体の代入 */
					}
				}
			}
			return 0; /* 成功終了 */
		}
	}
	/* 前とはまとめられなかった */
	if (i < man->frees) {
		/* 後ろがある */
		if (addr + size == man->free[i].addr) {
			/* 後ろとはまとめられる */
			man->free[i].addr = addr;
			man->free[i].size += size;
			return 0; /* 成功終了 */
		} 
	}
	/* 前にも後ろにもまとめられない */
	if (man->frees < MEMMAN_FREES) {
		/* free[i]より後ろを、後ろへずらして、すきまを作る */
		for (j = man->frees; j > i; j--) {
			man->free[j] = man->free[j - 1];
		}
		man->frees++;
		if (man->maxfrees < man->frees) {
			man->maxfrees = man->frees; /* 最大値を更新 */
		}
		man->free[i].addr = addr;
		man->free[i].size = size;
		return 0; /* 成功終了 */
	}
	/* 後ろにずらせなかった */
	man->losts++;
	man->lostsize += size;
	return -1; /* 失敗終了 */
}
```

これでメモリの空き容量が出せました。   
本では10日目に突入しました。   

今回はメモリ処理の部分をmemory.cというソースコードに分けていきます。   
ついでに新しい処理も追加しています。   
memory.c
```c
#include "bootpack.h"

#define EFLAGS_AC_BIT		0x00040000
#define CR0_CACHE_DISABLE	0x60000000

unsigned int memtest(unsigned int start, unsigned int end)
{
	char flag486 = 0;
	unsigned int eflg, cr0, i;

	/* 386か、486以降なのかの確認 */
	eflg = io_load_eflags();
	eflg |= EFLAGS_AC_BIT; /* AC-bit = 1 */
	io_store_eflags(eflg);
	eflg = io_load_eflags();
	if ((eflg & EFLAGS_AC_BIT) != 0) { /* 386ではAC=1にしても自動で0に戻ってしまう */
		flag486 = 1;
	}
	eflg &= ~EFLAGS_AC_BIT; /* AC-bit = 0 */
	io_store_eflags(eflg);

	if (flag486 != 0) {
		cr0 = load_cr0();
		cr0 |= CR0_CACHE_DISABLE; /* キャッシュ禁止 */
		store_cr0(cr0);
	}

	i = memtest_sub(start, end);

	if (flag486 != 0) {
		cr0 = load_cr0();
		cr0 &= ~CR0_CACHE_DISABLE; /* キャッシュ許可 */
		store_cr0(cr0);
	}

	return i;
}

void memman_init(struct MEMMAN *man)
{
	man->frees = 0;			/* あき情報の個数 */
	man->maxfrees = 0;		/* 状況観察用：freesの最大値 */
	man->lostsize = 0;		/* 解放に失敗した合計サイズ */
	man->losts = 0;			/* 解放に失敗した回数 */
	return;
}


unsigned int memman_total(struct MEMMAN *man)
/* あきサイズの合計を報告 */
{
	unsigned int i, t = 0;
	for (i = 0; i < man->frees; i++) {
		t += man->free[i].size;
	}
	return t;
}

unsigned int memman_alloc(struct MEMMAN *man, unsigned int size)
/* 確保 */
{
	unsigned int i, a;
	for (i = 0; i< man->frees; i++) {
		if (man->free[i].size >= size) {
			/* 十分な広さのあきを発見 */
			a = man->free[i].addr;
			man->free[i].addr += size;
			man->free[i].size -= size;
			if (man->free[i].size == 0) {
				/* free[i]がなくなったので前へつめる */
				man->frees--;
				for (; i < man->frees; i++) {
					man->free[i] = man->free[i+1]; /* 構造体の代入 */
				}
			}
			return 0;
		}
	}
	return 0; /* あきがない */
}

int memman_free(struct MEMMAN *man, unsigned int addr, unsigned int size) 
/* 解放 */
{
	int i, j;
	/* まとめやすさを考えると、free[]がaddr順に並んでいるほうがいい */
	/* だからまず、どこに入れるべきかを決める */
	for (i = 0; i < man->frees; i++) {
		if (man->free[i].addr > addr) {
			break;
		}
	}
	/* free[i - 1].addr < addr < free[i].addr */
	if (i > 0) {
		/* 前がある */
		if (man->free[i - 1].addr + man->free[i - i].size == addr) {
			/* 前のあき領域にまとめられる */
			man->free[i - 1].size += size;
			if (i < man->frees) {
				/* 後ろもある */
				if (addr + size == man->free[i].size) {
					/* なんと後ろともまとめられる */
					man->free[i - 1].size += man->free[i].size;
					/* man->free[i]の削除 */
					/* free[i]がなくなったので前へつめる */
					man->frees--;
					for (; i < man->frees; i++) {
						man->free[i] = man->free[i + 1]; /* 構造体の代入 */
					}
				}
			}
			return 0; /* 成功終了 */
		}
	}
	/* 前とはまとめられなかった */
	if (i < man->frees) {
		/* 後ろがある */
		if (addr + size == man->free[i].addr) {
			/* 後ろとはまとめられる */
			man->free[i].addr = addr;
			man->free[i].size += size;
			return 0; /* 成功終了 */
		} 
	}
	/* 前にも後ろにもまとめられない */
	if (man->frees < MEMMAN_FREES) {
		/* free[i]より後ろを、後ろへずらして、すきまを作る */
		for (j = man->frees; j > i; j--) {
			man->free[j] = man->free[j - 1];
		}
		man->frees++;
		if (man->maxfrees < man->frees) {
			man->maxfrees = man->frees; /* 最大値を更新 */
		}
		man->free[i].addr = addr;
		man->free[i].size = size;
		return 0; /* 成功終了 */
	}
	/* 後ろにずらせなかった */
	man->losts++;
	man->lostsize += size;
	return -1; /* 失敗終了 */
}

unsigned int memman_alloc_4k(struct MEMMAN *man, unsigned int size)
{
    unsigned int a;
    size = (size + 0xfff) & 0xfffff000;
    a = memman_alloc(man, size);
    return a;
}

int memman_free_4k(struct MEMMAN *man, unsigned int addr, unsigned int size)
{
    int i;
    size = (size + 0xfff) & 0xfffff000;
    i = memman_free(man, addr, size);
    return i;
}
```

bootpack.c
```c
#include "bootpack.h"
#include <stdio.h>

void HariMain(void)
{
    struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], mcursor[256], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x000010000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	init_screen8(binfo->vram, binfo->scrnx, binfo->scrny);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	init_mouse_cursor8(mcursor, COL8_008484);
	putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
	sprintf(s, "(%d, %d)", mx, my);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);

	sprintf(s, "memory %dMB free : %dKB", 
			memtotal / (1024 * 1024), memman_total(memman) /1024);
	putfonts8_asc(binfo->vram, binfo->scrnx, 0, 32, COL8_FFFFFF, s);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 16, 15, 31);
				putfonts8_asc(binfo->vram, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2]= 'c';
					}
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(binfo->vram, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					/* マウスカーソルの移動 */
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, mx, my, mx + 15, my + 15);
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 16) {
						mx = binfo->scrnx - 16;
					}
					if (my > binfo->scrny - 16) {
						my = binfo->scrny - 16;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(binfo->vram, binfo->scrnx, COL8_008484, 0, 0, 79, 15);
					putfonts8_asc(binfo->vram, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
					putblock8_8(binfo->vram, binfo->scrnx, 16, 16, mx, my, mcursor, 16);
				}
			}
		}
	}
}
```

bootpack.h
```h
/* memory */
#define MEMMAN_FREES		4090
#define MEMMAN_ADDR			0x003c0000

struct FREEINFO { 	/* あき情報 */
	unsigned int addr, size;
};

struct MEMMAN {		/* メモリ管理 */
	int frees, maxfrees, lostsize, losts;
	struct FREEINFO free[MEMMAN_FREES];
};

unsigned int memtest(unsigned int start, unsigned int end);
void memman_init(struct MEMMAN *man);
unsigned int memman_total(struct MEMMAN *man);
unsigned int memman_alloc(struct MEMMAN *man, unsigned int siza);
int memman_free(struct MEMMAN *man, unsigned int addr, unsigned int size);
unsigned int memman_alloc_4k(struct MEMMAN *man, unsigned int size);
int memman_free_4k(struct MEMMAN *man, unsigned int addr, unsigned int size);
```

Makefile
```Makefile
OBJS_BOOTPACK = bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj \
				int.obj fifo.obj keybord.obj mouse.obj memory.obj
```

これでbootpack.cがきれいになりました。   

さらに進めていってマウスが動いてもグラフィックが消えないように対応します。   
bootpack.c
```c
/* bootpackのメイン */

#include "bootpack.h"
#include <stdio.h>

void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse;
	unsigned char *buf_back, buf_mouse[256];

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	sheet_slide(shtctl, sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(shtctl, sht_mouse, mx, my);
	sheet_updown(shtctl, sht_back,  0);
	sheet_updown(shtctl, sht_mouse, 1);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 32, COL8_FFFFFF, s);
	sheet_refresh(shtctl);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(buf_back, binfo->scrnx, COL8_008484,  0, 16, 15, 31);
				putfonts8_asc(buf_back, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
				sheet_refresh(shtctl);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(buf_back, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 16) {
						mx = binfo->scrnx - 16;
					}
					if (my > binfo->scrny - 16) {
						my = binfo->scrny - 16;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 0, 0, 79, 15); /* 座標消す */
					putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s); /* 座標書く */
					sheet_slide(shtctl, sht_mouse, mx, my); /* sheet_refreshを含む */
				}
			}
		}
	}
}
```

sheet.c
```c
/* マウスやウィンドウの重ね合わせ処理 */

#include "bootpack.h"

#define SHEET_USE		1

struct SHTCTL *shtctl_init(struct MEMMAN *memman, unsigned char *vram, int xsize, int ysize)
{
	struct SHTCTL *ctl;
	int i;
	ctl = (struct SHTCTL *) memman_alloc_4k(memman, sizeof (struct SHTCTL));
	if (ctl == 0) {
		goto err;
	}
	ctl->vram = vram;
	ctl->xsize = xsize;
	ctl->ysize = ysize;
	ctl->top = -1; /* シートは一枚もない */
	for (i = 0; i < MAX_SHEETS; i++) {
		ctl->sheets0[i].flags = 0; /* 未使用マーク */
	}
err:
	return ctl;
}

struct SHEET *sheet_alloc(struct SHTCTL *ctl)
{
	struct SHEET *sht;
	int i;
	for (i = 0; i < MAX_SHEETS; i++) {
		if (ctl->sheets0[i].flags == 0) {
			sht = &ctl->sheets0[i];
			sht->flags = SHEET_USE; /* 使用中マーク */
			sht->height = -1; /* 非表示中 */
			return sht;
		}
	}
	return 0;	/* 全てのシートが使用中だった */
}

void sheet_setbuf(struct SHEET *sht, unsigned char *buf, int xsize, int ysize, int col_inv)
{
	sht->buf = buf;
	sht->bxsize = xsize;
	sht->bysize = ysize;
	sht->col_inv = col_inv;
	return;
}

void sheet_updown(struct SHTCTL *ctl, struct SHEET *sht, int height)
{
	int h, old = sht->height; /* 設定前の高さを記憶する */

	/* 指定が低すぎや高すぎだったら、修正する */
	if (height > ctl->top + 1) {
		height = ctl->top + 1;
	}
	if (height < -1) {
		height = -1;
	}
	sht->height = height; /* 高さを設定 */

	/* 以下は主にsheets[]の並べ替え */
	if (old > height) {	/* 以前よりも低くなる */
		if (height >= 0) {
			/* 間のものを引き上げる */
			for (h = old; h > height; h--) {
				ctl->sheets[h] = ctl->sheets[h - 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
		} else {	/* 非表示化 */
			if (ctl->top > old) {
				/* 上になっているものをおろす */
				for (h = old; h < ctl->top; h++) {
					ctl->sheets[h] = ctl->sheets[h + 1];
					ctl->sheets[h]->height = h;
				}
			}
			ctl->top--; /* 表示中の下じきが一つ減るので、一番上の高さが減る */
		}
		sheet_refresh(ctl); /* 新しい下じきの情報に沿って画面を描き直す */
	} else if (old < height) {	/* 以前よりも高くなる */
		if (old >= 0) {
			/* 間のものを押し下げる */
			for (h = old; h < height; h++) {
				ctl->sheets[h] = ctl->sheets[h + 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
		} else {	/* 非表示状態から表示状態へ */
			/* 上になるものを持ち上げる */
			for (h = ctl->top; h >= height; h--) {
				ctl->sheets[h + 1] = ctl->sheets[h];
				ctl->sheets[h + 1]->height = h + 1;
			}
			ctl->sheets[height] = sht;
			ctl->top++; /* 表示中の下じきが一つ増えるので、一番上の高さが増える */
		}
		sheet_refresh(ctl); /* 新しい下じきの情報に沿って画面を描き直す */
	}
	return;
}

void sheet_refresh(struct SHTCTL *ctl)
{
	int h, bx, by, vx, vy;
	unsigned char *buf, c, *vram = ctl->vram;
	struct SHEET *sht;
	for (h = 0; h <= ctl->top; h++) {
		sht = ctl->sheets[h];
		buf = sht->buf;
		for (by = 0; by < sht->bysize; by++) {
			vy = sht->vy0 + by;
			for (bx = 0; bx < sht->bxsize; bx++) {
				vx = sht->vx0 + bx;
				c = buf[by * sht->bxsize + bx];
				if (c != sht->col_inv) {
					vram[vy * ctl->xsize + vx] = c;
				}
			}
		}
	}
	return;
}

void sheet_slide(struct SHTCTL *ctl, struct SHEET *sht, int vx0, int vy0)
{
	sht->vx0 = vx0;
	sht->vy0 = vy0;
	if (sht->height >= 0) { /* もしも表示中なら */
		sheet_refresh(ctl); /* 新しい下じきの情報に沿って画面を描き直す */
	}
	return;
}

void sheet_free(struct SHTCTL *ctl, struct SHEET *sht)
{
	if (sht->height >= 0) {
		sheet_updown(ctl, sht, -1); /* 表示中ならまず非表示にする */
	}
	sht->flags = 0; /* 未使用マーク */
	return;
}
```

bootpack.h
```h
/* sheet.c */
#define MAX_SHEETS      256
struct SHEET {
    unsigned char *buf;
    int bxsize, bysize, vx0, vy0, col_inv, height, flags;
};
struct SHTCTL {
    unsigned char *vram;
    int xsize, ysize, top;
    struct SHEET *sheets[MAX_SHEETS];
    struct SHEET sheets0[MAX_SHEETS];
};
struct SHTCTL *shtctl_init(struct MEMMAN *memman, unsigned char *vram, int xsize, int ysize);
struct SHEET *sheet_alloc(struct SHTCTL *ctl);
void sheet_setbuf(struct SHEET *sht, unsigned char *buf, int xsize, int ysize, int col_inv);
void sheet_updown(struct SHTCTL *ctl, struct SHEET *sht, int height);
void sheet_refresh(struct SHTCTL *ctl);
void sheet_slide(struct SHTCTL *ctl, struct SHEET *sht, int vx0, int vy0);
void sheet_free(struct SHTCTL *ctl, struct SHEET *sht);
```

これで起動しても画面が真っ暗なまんまです。   
なぜでしょう。

**実装過程**   
・　[2df4bb731190ee7a73d899289655171d175cc624](2df4bb731190ee7a73d899289655171d175cc624)

## day9
9日目です。   
前回最後に画面が真っ暗になってしまった件ですが、解決することができました。   
理由はmemory.cに誤字があったことが理由でした。   
しょうもない理由でした。書き直したmemory.cがこちらになります。   
memory.c
```c
/* メモリ関係 */

#include "bootpack.h"

#define EFLAGS_AC_BIT		0x00040000
#define CR0_CACHE_DISABLE	0x60000000

unsigned int memtest(unsigned int start, unsigned int end)
{
	char flg486 = 0;
	unsigned int eflg, cr0, i;

	/* 386か、486以降なのかの確認 */
	eflg = io_load_eflags();
	eflg |= EFLAGS_AC_BIT; /* AC-bit = 1 */
	io_store_eflags(eflg);
	eflg = io_load_eflags();
	if ((eflg & EFLAGS_AC_BIT) != 0) { /* 386ではAC=1にしても自動で0に戻ってしまう */
		flg486 = 1;
	}
	eflg &= ~EFLAGS_AC_BIT; /* AC-bit = 0 */
	io_store_eflags(eflg);

	if (flg486 != 0) {
		cr0 = load_cr0();
		cr0 |= CR0_CACHE_DISABLE; /* キャッシュ禁止 */
		store_cr0(cr0);
	}

	i = memtest_sub(start, end);

	if (flg486 != 0) {
		cr0 = load_cr0();
		cr0 &= ~CR0_CACHE_DISABLE; /* キャッシュ許可 */
		store_cr0(cr0);
	}

	return i;
}

void memman_init(struct MEMMAN *man)
{
	man->frees = 0;			/* あき情報の個数 */
	man->maxfrees = 0;		/* 状況観察用：freesの最大値 */
	man->lostsize = 0;		/* 解放に失敗した合計サイズ */
	man->losts = 0;			/* 解放に失敗した回数 */
	return;
}

unsigned int memman_total(struct MEMMAN *man)
/* あきサイズの合計を報告 */
{
	unsigned int i, t = 0;
	for (i = 0; i < man->frees; i++) {
		t += man->free[i].size;
	}
	return t;
}

unsigned int memman_alloc(struct MEMMAN *man, unsigned int size)
/* 確保 */
{
	unsigned int i, a;
	for (i = 0; i < man->frees; i++) {
		if (man->free[i].size >= size) {
			/* 十分な広さのあきを発見 */
			a = man->free[i].addr;
			man->free[i].addr += size;
			man->free[i].size -= size;
			if (man->free[i].size == 0) {
				/* free[i]がなくなったので前へつめる */
				man->frees--;
				for (; i < man->frees; i++) {
					man->free[i] = man->free[i + 1]; /* 構造体の代入 */
				}
			}
			return a;
		}
	}
	return 0; /* あきがない */
}

int memman_free(struct MEMMAN *man, unsigned int addr, unsigned int size)
/* 解放 */
{
	int i, j;
	/* まとめやすさを考えると、free[]がaddr順に並んでいるほうがいい */
	/* だからまず、どこに入れるべきかを決める */
	for (i = 0; i < man->frees; i++) {
		if (man->free[i].addr > addr) {
			break;
		}
	}
	/* free[i - 1].addr < addr < free[i].addr */
	if (i > 0) {
		/* 前がある */
		if (man->free[i - 1].addr + man->free[i - 1].size == addr) {
			/* 前のあき領域にまとめられる */
			man->free[i - 1].size += size;
			if (i < man->frees) {
				/* 後ろもある */
				if (addr + size == man->free[i].addr) {
					/* なんと後ろともまとめられる */
					man->free[i - 1].size += man->free[i].size;
					/* man->free[i]の削除 */
					/* free[i]がなくなったので前へつめる */
					man->frees--;
					for (; i < man->frees; i++) {
						man->free[i] = man->free[i + 1]; /* 構造体の代入 */
					}
				}
			}
			return 0; /* 成功終了 */
		}
	}
	/* 前とはまとめられなかった */
	if (i < man->frees) {
		/* 後ろがある */
		if (addr + size == man->free[i].addr) {
			/* 後ろとはまとめられる */
			man->free[i].addr = addr;
			man->free[i].size += size;
			return 0; /* 成功終了 */
		}
	}
	/* 前にも後ろにもまとめられない */
	if (man->frees < MEMMAN_FREES) {
		/* free[i]より後ろを、後ろへずらして、すきまを作る */
		for (j = man->frees; j > i; j--) {
			man->free[j] = man->free[j - 1];
		}
		man->frees++;
		if (man->maxfrees < man->frees) {
			man->maxfrees = man->frees; /* 最大値を更新 */
		}
		man->free[i].addr = addr;
		man->free[i].size = size;
		return 0; /* 成功終了 */
	}
	/* 後ろにずらせなかった */
	man->losts++;
	man->lostsize += size;
	return -1; /* 失敗終了 */
}

unsigned int memman_alloc_4k(struct MEMMAN *man, unsigned int size)
{
	unsigned int a;
	size = (size + 0xfff) & 0xfffff000;
	a = memman_alloc(man, size);
	return a;
}

int memman_free_4k(struct MEMMAN *man, unsigned int addr, unsigned int size)
{
	int i;
	size = (size + 0xfff) & 0xfffff000;
	i = memman_free(man, addr, size);
	return i;
}
```

今回はさらにマウスカーソル早くしていこうという試みを実装していきます。   

sheet.c
```c
void sheet_refreshsub(struct SHTCTL *ctl, int vx0, int vy0, int vx1, int vy1)
{
	int h, bx, by, vx, vy;
	unsigned char *buf, c, *vram = ctl->vram;
	struct SHEET *sht;
	for (h = 0; h <= ctl->top; h++) {
		sht = ctl->sheets[h];
		buf = sht->buf;
		for (by = 0; by < sht->bysize; by++) {
			vy = sht->vy0 + by;
			for (bx = 0; bx < sht->bxsize; bx++) {
				vx = sht->vx0 + bx;
				if (vx0 <= vx && vx < vx1 && vy0 <= vy && vy < vy1) {
					c = buf[by * sht->bxsize + bx];
					if (c != sht->col_inv) {
						vram[vy * ctl->xsize + vx] = c;
					}
				}
			}
		}
	}
	return;
}

void sheet_updown(struct SHTCTL *ctl, struct SHEET *sht, int height)
{
	int h, old = sht->height; /* 設定前の高さを記憶する */

	/* 指定が低すぎや高すぎだったら、修正する */
	if (height > ctl->top + 1) {
		height = ctl->top + 1;
	}
	if (height < -1) {
		height = -1;
	}
	sht->height = height; /* 高さを設定 */

	/* 以下は主にsheets[]の並べ替え */
	if (old > height) {	/* 以前よりも低くなる */
		if (height >= 0) {
			/* 間のものを引き上げる */
			for (h = old; h > height; h--) {
				ctl->sheets[h] = ctl->sheets[h - 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
		} else {	/* 非表示化 */
			if (ctl->top > old) {
				/* 上になっているものをおろす */
				for (h = old; h < ctl->top; h++) {
					ctl->sheets[h] = ctl->sheets[h + 1];
					ctl->sheets[h]->height = h;
				}
			}
			ctl->top--; /* 表示中の下じきが一つ減るので、一番上の高さが減る */
		}
		sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize);
	} else if (old < height) {	/* 以前よりも高くなる */
		if (old >= 0) {
			/* 間のものを押し下げる */
			for (h = old; h < height; h++) {
				ctl->sheets[h] = ctl->sheets[h + 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
		} else {	/* 非表示状態から表示状態へ */
			/* 上になるものを持ち上げる */
			for (h = ctl->top; h >= height; h--) {
				ctl->sheets[h + 1] = ctl->sheets[h];
				ctl->sheets[h + 1]->height = h + 1;
			}
			ctl->sheets[height] = sht;
			ctl->top++; /* 表示中の下じきが一つ増えるので、一番上の高さが増える */
		}
		sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize);
	}
	return;
}

void sheet_refresh(struct SHTCTL *ctl, struct SHEET *sht, int bx0, int by0, int bx1, int by1)
{
	if (sht->height >= 0) { /* もしも表示中なら、新しい下じきの情報に沿って画面を描き直す */
		sheet_refreshsub(ctl, sht->vx0 + bx0, sht->vy0 + by0, sht->vx0 + bx1, sht->vy0 + by1);
	}
	return;
}

void sheet_slide(struct SHTCTL *ctl, struct SHEET *sht, int vx0, int vy0)
{
	int old_vx0 = sht->vx0, old_vy0 = sht->vy0;
	sht->vx0 = vx0;
	sht->vy0 = vy0;
	if (sht->height >= 0) { /* もしも表示中なら、新しい下じきの情報に沿って画面を描き直す */
		sheet_refreshsub(ctl, old_vx0, old_vy0, old_vx0 + sht->bxsize, old_vy0 + sht->bysize);
		sheet_refreshsub(ctl, vx0, vy0, vx0 + sht->bxsize, vy0 + sht->bysize);
	}
	return;
}
```

bootpack.c
```c
void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse;
	unsigned char *buf_back, buf_mouse[256];

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	sheet_slide(shtctl, sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(shtctl, sht_mouse, mx, my);
	sheet_updown(shtctl, sht_back,  0);
	sheet_updown(shtctl, sht_mouse, 1);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 32, COL8_FFFFFF, s);
	sheet_refresh(shtctl, sht_back, 0, 0, binfo->scrnx, 48);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(buf_back, binfo->scrnx, COL8_008484,  0, 16, 15, 31);
				putfonts8_asc(buf_back, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
				sheet_refresh(shtctl, sht_back, 0, 16, 16, 32);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(buf_back, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					sheet_refresh(shtctl, sht_back, 32, 16, 32 + 15 * 8, 32);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 16) {
						mx = binfo->scrnx - 16;
					}
					if (my > binfo->scrny - 16) {
						my = binfo->scrny - 16;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 0, 0, 79, 15); /* 座標消す */
					putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s); /* 座標書く */
					sheet_refresh(shtctl, sht_back, 0, 0, 80, 16);
					sheet_slide(shtctl, sht_mouse, mx, my);
				}
			}
		}
	}
}
```

bootpack.h
```h
/* sheet.c */
#define MAX_SHEETS		256
struct SHEET {
	unsigned char *buf;
	int bxsize, bysize, vx0, vy0, col_inv, height, flags;
};
struct SHTCTL {
	unsigned char *vram;
	int xsize, ysize, top;
	struct SHEET *sheets[MAX_SHEETS];
	struct SHEET sheets0[MAX_SHEETS];
};
struct SHTCTL *shtctl_init(struct MEMMAN *memman, unsigned char *vram, int xsize, int ysize);
struct SHEET *sheet_alloc(struct SHTCTL *ctl);
void sheet_setbuf(struct SHEET *sht, unsigned char *buf, int xsize, int ysize, int col_inv);
void sheet_updown(struct SHTCTL *ctl, struct SHEET *sht, int height);
void sheet_refresh(struct SHTCTL *ctl, struct SHEET *sht, int bx0, int by0, int bx1, int by1);
void sheet_slide(struct SHTCTL *ctl, struct SHEET *sht, int vx0, int vy0);
void sheet_free(struct SHTCTL *ctl, struct SHEET *sht);
```

ようやく動くマウスが見れました笑   
そこそこ早いと思いますがさらにスピードを上げていきます。   

sheet.c
```c
void sheet_refreshsub(struct SHTCTL *ctl, int vx0, int vy0, int vx1, int vy1)
{
	int h, bx, by, vx, vy, bx0, by0, bx1, by1;
	unsigned char *buf, c, *vram = ctl->vram;
	struct SHEET *sht;
	for (h = 0; h <= ctl->top; h++) {
		sht = ctl->sheets[h];
		buf = sht->buf;
		/* vx0～vy1を使って、bx0～by1を逆算する */
		bx0 = vx0 - sht->vx0;
		by0 = vy0 - sht->vy0;
		bx1 = vx1 - sht->vx0;
		by1 = vy1 - sht->vy0;
		if (bx0 < 0) { bx0 = 0; }
		if (by0 < 0) { by0 = 0; }
		if (bx1 > sht->bxsize) { bx1 = sht->bxsize; }
		if (by1 > sht->bysize) { by1 = sht->bysize; }
		for (by = by0; by < by1; by++) {
			vy = sht->vy0 + by;
			for (bx = bx0; bx < bx1; bx++) {
				vx = sht->vx0 + bx;
				c = buf[by * sht->bxsize + bx];
				if (c != sht->col_inv) {
					vram[vy * ctl->xsize + vx] = c;
				}
			}
		}
	}
	return;
}
```

これでさらにすいすい動くようになりました。(体感的には変わらない感じがしますが笑)

本では11日目に入りました。   
マウスカーソルを縦横無人に動かしたいと思います。   
bootpack.cの一部分を16→1に変えるだけですね。   

bootpack.c
```c
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
```

実際動かして画面端までいくとますが増殖するというバッグ発生してしまいます。   
次はこのバグを直していきます。   

sheet.c 
```c
void sheet_refreshsub(struct SHTCTL *ctl, int vx0, int vy0, int vx1, int vy1)
{
	int h, bx, by, vx, vy, bx0, by0, bx1, by1;
	unsigned char *buf, c, *vram = ctl->vram;
	struct SHEET *sht;
	/* refresh範囲が画面買いにはみ出していたら補正 */
	if (vx0 < 0) { vx0 = 0; }
	if (vy0 < 0) { vy0 = 0; }
	if (vx1 > ctl->xsize) { vx1 = ctl->xsize; }
	if (vy1 > ctl->ysize) { vy1 = ctl->ysize; }
	for (h = 0; h <= ctl->top; h++) {
		sht = ctl->sheets[h];
		buf = sht->buf;
		/* vx0～vy1を使って、bx0～by1を逆算する */
		bx0 = vx0 - sht->vx0;
		by0 = vy0 - sht->vy0;
		bx1 = vx1 - sht->vx0;
		by1 = vy1 - sht->vy0;
		if (bx0 < 0) { bx0 = 0; }
		if (by0 < 0) { by0 = 0; }
		if (bx1 > sht->bxsize) { bx1 = sht->bxsize; }
		if (by1 > sht->bysize) { by1 = sht->bysize; }
		for (by = by0; by < by1; by++) {
			vy = sht->vy0 + by;
			for (bx = bx0; bx < bx1; bx++) {
				vx = sht->vx0 + bx;
				c = buf[by * sht->bxsize + bx];
				if (c != sht->col_inv) {
					vram[vy * ctl->xsize + vx] = c;
				}
			}
		}
	}
	return;
}
```

これで縦横無尽にすいすい動くようになりました。

**実装過程**   
・　[e918545150418ef88c8caf8809a5b23998bfd445](e918545150418ef88c8caf8809a5b23998bfd445)

## day10
今日はソースコードを少し簡略化してみたいと思います。  
bootpack.h
```h
/* sheet.c */
#define MAX_SHEETS		256
struct SHEET {
	unsigned char *buf;
	int bxsize, bysize, vx0, vy0, col_inv, height, flags;
	struct SHTCTL *ctl;
};
struct SHTCTL {
	unsigned char *vram;
	int xsize, ysize, top;
	struct SHEET *sheets[MAX_SHEETS];
	struct SHEET sheets0[MAX_SHEETS];
};
struct SHTCTL *shtctl_init(struct MEMMAN *memman, unsigned char *vram, int xsize, int ysize);
struct SHEET *sheet_alloc(struct SHTCTL *ctl);
void sheet_setbuf(struct SHEET *sht, unsigned char *buf, int xsize, int ysize, int col_inv);
void sheet_updown(struct SHEET *sht, int height);
void sheet_refresh(struct SHEET *sht, int bx0, int by0, int bx1, int by1);
void sheet_slide(struct SHEET *sht, int vx0, int vy0);
void sheet_free(struct SHEET *sht);
```

sheet.c
```c
void sheet_updown(struct SHEET *sht, int height)
{
	struct SHTCTL *ctl = sht->ctl;
	int h, old = sht->height; /* 設定前の高さを記憶する */

	/* 指定が低すぎや高すぎだったら、修正する */
	if (height > ctl->top + 1) {
		height = ctl->top + 1;
	}
	if (height < -1) {
		height = -1;
	}
	sht->height = height; /* 高さを設定 */

	/* 以下は主にsheets[]の並べ替え */
	if (old > height) {	/* 以前よりも低くなる */
		if (height >= 0) {
			/* 間のものを引き上げる */
			for (h = old; h > height; h--) {
				ctl->sheets[h] = ctl->sheets[h - 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
		} else {	/* 非表示化 */
			if (ctl->top > old) {
				/* 上になっているものをおろす */
				for (h = old; h < ctl->top; h++) {
					ctl->sheets[h] = ctl->sheets[h + 1];
					ctl->sheets[h]->height = h;
				}
			}
			ctl->top--; /* 表示中の下じきが一つ減るので、一番上の高さが減る */
		}
		sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize);
	} else if (old < height) {	/* 以前よりも高くなる */
		if (old >= 0) {
			/* 間のものを押し下げる */
			for (h = old; h < height; h++) {
				ctl->sheets[h] = ctl->sheets[h + 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
		} else {	/* 非表示状態から表示状態へ */
			/* 上になるものを持ち上げる */
			for (h = ctl->top; h >= height; h--) {
				ctl->sheets[h + 1] = ctl->sheets[h];
				ctl->sheets[h + 1]->height = h + 1;
			}
			ctl->sheets[height] = sht;
			ctl->top++; /* 表示中の下じきが一つ増えるので、一番上の高さが増える */
		}
		sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize);
	}
	return;
}

void sheet_refresh(struct SHEET *sht, int bx0, int by0, int bx1, int by1)
{
	if (sht->height >= 0) { /* もしも表示中なら、新しい下じきの情報に沿って画面を描き直す */
		sheet_refreshsub(sht->ctl, sht->vx0 + bx0, sht->vy0 + by0, sht->vx0 + bx1, sht->vy0 + by1);
	}
	return;
}

void sheet_slide(struct SHEET *sht, int vx0, int vy0)
{
	int old_vx0 = sht->vx0, old_vy0 = sht->vy0;
	sht->vx0 = vx0;
	sht->vy0 = vy0;
	if (sht->height >= 0) { /* もしも表示中なら、新しい下じきの情報に沿って画面を描き直す */
		sheet_refreshsub(sht->ctl, old_vx0, old_vy0, old_vx0 + sht->bxsize, old_vy0 + sht->bysize);
		sheet_refreshsub(sht->ctl, vx0, vy0, vx0 + sht->bxsize, vy0 + sht->bysize);
	}
	return;
}

void sheet_free(struct SHEET *sht)
{
	if (sht->height >= 0) {
		sheet_updown(sht, -1); /* 表示中ならまず非表示にする */
	}
	sht->flags = 0; /* 未使用マーク */
	return;
}
```

bootpack.c
```c
void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse;
	unsigned char *buf_back, buf_mouse[256];

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	sheet_slide(sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(sht_mouse, mx, my);
	sheet_updown(sht_back,  0);
	sheet_updown(sht_mouse, 1);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 32, COL8_FFFFFF, s);
	sheet_refresh(sht_back, 0, 0, binfo->scrnx, 48);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(buf_back, binfo->scrnx, COL8_008484,  0, 16, 15, 31);
				putfonts8_asc(buf_back, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
				sheet_refresh(sht_back, 0, 16, 16, 32);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(buf_back, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					sheet_refresh(sht_back, 32, 16, 32 + 15 * 8, 32);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 0, 0, 79, 15); /* 座標消す */
					putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s); /* 座標書く */
					sheet_refresh(sht_back, 0, 0, 80, 16);
					sheet_slide(sht_mouse, mx, my);
				}
			}
		}
	}
}
```

これで少しはきれいになりました。

**実装過程**   
・　[65be7756b6fab67036c574f2e2292499c3eed180](65be7756b6fab67036c574f2e2292499c3eed180)

## day11
ウィンドウを出してみたいと思います。
bootpack.c
```c
void make_window8(unsigned char *buf, int xsize, int ysize, char *title);

void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse, *sht_win;
	unsigned char *buf_back, buf_mouse[256], *buf_win;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	sht_win = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	buf_win  = (unsigned char *) memman_alloc_4k(memman, 160 * 68);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	sheet_setbuf(sht_win, buf_win, 160, 68, -1); /* 透明色なし */
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	make_window8(buf_win, 160, 68, "window");
	putfonts8_asc(buf_win, 160, 24, 28, COL8_000000, "Welcome to");
	putfonts8_asc(buf_win, 160, 24, 44, COL8_000000, "  Haribote-OS!");
	sheet_slide(sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(sht_mouse, mx, my);
	sheet_slide(sht_win, 80, 72);
	sheet_updown(sht_back,  0);
	sheet_updown(sht_win, 1);
	sheet_updown(sht_mouse, 2);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 32, COL8_FFFFFF, s);
	sheet_refresh(sht_back, 0, 0, binfo->scrnx, 48);

	for (;;) {
		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(buf_back, binfo->scrnx, COL8_008484,  0, 16, 15, 31);
				putfonts8_asc(buf_back, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
				sheet_refresh(sht_back, 0, 16, 16, 32);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(buf_back, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					sheet_refresh(sht_back, 32, 16, 32 + 15 * 8, 32);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 0, 0, 79, 15); /* 座標消す */
					putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s); /* 座標書く */
					sheet_refresh(sht_back, 0, 0, 80, 16);
					sheet_slide(sht_mouse, mx, my);
				}
			}
		}
	}
}

void make_window8(unsigned char *buf, int xsize, int ysize, char *title)
{
	static char closebtn[14][16] = {
		"OOOOOOOOOOOOOOO@",
		"OQQQQQQQQQQQQQ$@",
		"OQQQQQQQQQQQQQ$@",
		"OQQQ@@QQQQ@@QQ$@",
		"OQQQQ@@QQ@@QQQ$@",
		"OQQQQQ@@@@QQQQ$@",
		"OQQQQQQ@@QQQQQ$@",
		"OQQQQQ@@@@QQQQ$@",
		"OQQQQ@@QQ@@QQQ$@",
		"OQQQ@@QQQQ@@QQ$@",
		"OQQQQQQQQQQQQQ$@",
		"OQQQQQQQQQQQQQ$@",
		"O$$$$$$$$$$$$$$@",
		"@@@@@@@@@@@@@@@@"
	};
	int x, y;
	char c;
	boxfill8(buf, xsize, COL8_C6C6C6, 0,         0,         xsize - 1, 0        );
	boxfill8(buf, xsize, COL8_FFFFFF, 1,         1,         xsize - 2, 1        );
	boxfill8(buf, xsize, COL8_C6C6C6, 0,         0,         0,         ysize - 1);
	boxfill8(buf, xsize, COL8_FFFFFF, 1,         1,         1,         ysize - 2);
	boxfill8(buf, xsize, COL8_848484, xsize - 2, 1,         xsize - 2, ysize - 2);
	boxfill8(buf, xsize, COL8_000000, xsize - 1, 0,         xsize - 1, ysize - 1);
	boxfill8(buf, xsize, COL8_C6C6C6, 2,         2,         xsize - 3, ysize - 3);
	boxfill8(buf, xsize, COL8_000084, 3,         3,         xsize - 4, 20       );
	boxfill8(buf, xsize, COL8_848484, 1,         ysize - 2, xsize - 2, ysize - 2);
	boxfill8(buf, xsize, COL8_000000, 0,         ysize - 1, xsize - 1, ysize - 1);
	putfonts8_asc(buf, xsize, 24, 4, COL8_FFFFFF, title);
	for (y = 0; y < 14; y++) {
		for (x = 0; x < 16; x++) {
			c = closebtn[y][x];
			if (c == '@') {
				c = COL8_000000;
			} else if (c == '$') {
				c = COL8_848484;
			} else if (c == 'Q') {
				c = COL8_C6C6C6;
			} else {
				c = COL8_FFFFFF;
			}
			buf[(5 + y) * xsize + (xsize - 21 + x)] = c;
		}
	}
	return;
}
```

シートの順番を変えてみます。
bootpack.c
```c
sheet_updown(sht_back,  0);
sheet_updown(sht_mouse, 1);
sheet_updown(sht_win, 2);
```

カウンターを表示させてみます。
bootpacl.c
```c
void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned int memtotal, count = 0;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse, *sht_win;
	unsigned char *buf_back, buf_mouse[256], *buf_win;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	io_out8(PIC0_IMR, 0xf9); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	sht_win = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	buf_win  = (unsigned char *) memman_alloc_4k(memman, 160 * 52);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	sheet_setbuf(sht_win, buf_win, 160, 52, -1); /* 透明色なし */
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	make_window8(buf_win, 160, 52, "counter");
	sheet_slide(sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(sht_mouse, mx, my);
	sheet_slide(sht_win, 80, 72);
	sheet_updown(sht_back,  0);
	sheet_updown(sht_win, 1);
	sheet_updown(sht_mouse, 2);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 32, COL8_FFFFFF, s);
	sheet_refresh(sht_back, 0, 0, binfo->scrnx, 48);

	for (;;) {
		count++;
		sprintf(s, "%010d", count);
		boxfill8(buf_win, 160, COL8_C6C6C6, 40, 28, 119, 43);
		putfonts8_asc(buf_win, 160, 40, 28, COL8_000000, s);
		sheet_refresh(sht_win, 40, 28, 120, 44);

		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) == 0) {
			io_stihlt();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(buf_back, binfo->scrnx, COL8_008484,  0, 16, 15, 31);
				putfonts8_asc(buf_back, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
				sheet_refresh(sht_back, 0, 16, 16, 32);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(buf_back, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					sheet_refresh(sht_back, 32, 16, 32 + 15 * 8, 32);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 0, 0, 79, 15); /* 座標消す */
					putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s); /* 座標書く */
					sheet_refresh(sht_back, 0, 0, 80, 16);
					sheet_slide(sht_mouse, mx, my);
				}
			}
		}
	}
}
```

次は画面のちらつきを直します（でも最近のpcはスペックがいいので全然ちらつかないのですが^^;）

screen.c
```c
void sheet_refreshsub(struct SHTCTL *ctl, int vx0, int vy0, int vx1, int vy1, int h0)
{
	int h, bx, by, vx, vy, bx0, by0, bx1, by1;
	unsigned char *buf, c, *vram = ctl->vram;
	struct SHEET *sht;
	/* refresh範囲が画面買いにはみ出していたら補正 */
	if (vx0 < 0) { vx0 = 0; }
	if (vy0 < 0) { vy0 = 0; }
	if (vx1 > ctl->xsize) { vx1 = ctl->xsize; }
	if (vy1 > ctl->ysize) { vy1 = ctl->ysize; }
	for (h = h0; h <= ctl->top; h++) {
		sht = ctl->sheets[h];
		buf = sht->buf;
		/* vx0～vy1を使って、bx0～by1を逆算する */
		bx0 = vx0 - sht->vx0;
		by0 = vy0 - sht->vy0;
		bx1 = vx1 - sht->vx0;
		by1 = vy1 - sht->vy0;
		if (bx0 < 0) { bx0 = 0; }
		if (by0 < 0) { by0 = 0; }
		if (bx1 > sht->bxsize) { bx1 = sht->bxsize; }
		if (by1 > sht->bysize) { by1 = sht->bysize; }
		for (by = by0; by < by1; by++) {
			vy = sht->vy0 + by;
			for (bx = bx0; bx < bx1; bx++) {
				vx = sht->vx0 + bx;
				c = buf[by * sht->bxsize + bx];
				if (c != sht->col_inv) {
					vram[vy * ctl->xsize + vx] = c;
				}
			}
		}
	}
	return;
}

void sheet_updown(struct SHEET *sht, int height)
{
	struct SHTCTL *ctl = sht->ctl;
	int h, old = sht->height; /* 設定前の高さを記憶する */

	/* 指定が低すぎや高すぎだったら、修正する */
	if (height > ctl->top + 1) {
		height = ctl->top + 1;
	}
	if (height < -1) {
		height = -1;
	}
	sht->height = height; /* 高さを設定 */

	/* 以下は主にsheets[]の並べ替え */
	if (old > height) {	/* 以前よりも低くなる */
		if (height >= 0) {
			/* 間のものを引き上げる */
			for (h = old; h > height; h--) {
				ctl->sheets[h] = ctl->sheets[h - 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
			sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize, height + 1);
		} else {	/* 非表示化 */
			if (ctl->top > old) {
				/* 上になっているものをおろす */
				for (h = old; h < ctl->top; h++) {
					ctl->sheets[h] = ctl->sheets[h + 1];
					ctl->sheets[h]->height = h;
				}
			}
			ctl->top--; /* 表示中の下じきが一つ減るので、一番上の高さが減る */
			sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize, 0);
		}
	} else if (old < height) {	/* 以前よりも高くなる */
		if (old >= 0) {
			/* 間のものを押し下げる */
			for (h = old; h < height; h++) {
				ctl->sheets[h] = ctl->sheets[h + 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
		} else {	/* 非表示状態から表示状態へ */
			/* 上になるものを持ち上げる */
			for (h = ctl->top; h >= height; h--) {
				ctl->sheets[h + 1] = ctl->sheets[h];
				ctl->sheets[h + 1]->height = h + 1;
			}
			ctl->sheets[height] = sht;
			ctl->top++; /* 表示中の下じきが一つ増えるので、一番上の高さが増える */
		}
		sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize, height);
	}
	return;
}

void sheet_refresh(struct SHEET *sht, int bx0, int by0, int bx1, int by1)
{
	if (sht->height >= 0) { /* もしも表示中なら、新しい下じきの情報に沿って画面を描き直す */
		sheet_refreshsub(sht->ctl, sht->vx0 + bx0, sht->vy0 + by0, sht->vx0 + bx1, sht->vy0 + by1, sht->height);
	}
	return;
}

void sheet_slide(struct SHEET *sht, int vx0, int vy0)
{
	int old_vx0 = sht->vx0, old_vy0 = sht->vy0;
	sht->vx0 = vx0;
	sht->vy0 = vy0;
	if (sht->height >= 0) { /* もしも表示中なら、新しい下じきの情報に沿って画面を描き直す */
		sheet_refreshsub(sht->ctl, old_vx0, old_vy0, old_vx0 + sht->bxsize, old_vy0 + sht->bysize, 0);
		sheet_refreshsub(sht->ctl, vx0, vy0, vx0 + sht->bxsize, vy0 + sht->bysize, sht->height);
	}
	return;
}

void sheet_free(struct SHEET *sht)
{
	if (sht->height >= 0) {
		sheet_updown(sht, -1); /* 表示中ならまず非表示にする */
	}
	sht->flags = 0; /* 未使用マーク */
	return;
}
```

**実装過程**   
・　[0228a237575fa39e2ea91b215f1b0f40ce95980a](0228a237575fa39e2ea91b215f1b0f40ce95980a)

## day12
さらにチラチラしないように改造したいと思います。
sheets.c
```c
/* マウスやウィンドウの重ね合わせ処理 */

#include "bootpack.h"

#define SHEET_USE		1

struct SHTCTL *shtctl_init(struct MEMMAN *memman, unsigned char *vram, int xsize, int ysize)
{
	struct SHTCTL *ctl;
	int i;
	ctl = (struct SHTCTL *) memman_alloc_4k(memman, sizeof (struct SHTCTL));
	if (ctl == 0) {
		goto err;
	}
	ctl->map = (unsigned char *) memman_alloc_4k(memman, xsize * ysize);
	if (ctl->map == 0) {
		memman_free_4k(memman, (int) ctl, sizeof (struct SHTCTL));
		goto err;
	}
	ctl->vram = vram;
	ctl->xsize = xsize;
	ctl->ysize = ysize;
	ctl->top = -1; /* シートは一枚もない */
	for (i = 0; i < MAX_SHEETS; i++) {
		ctl->sheets0[i].flags = 0; /* 未使用マーク */
		ctl->sheets0[i].ctl = ctl; /* 所属を記録 */
	}
err:
	return ctl;
}

struct SHEET *sheet_alloc(struct SHTCTL *ctl)
{
	struct SHEET *sht;
	int i;
	for (i = 0; i < MAX_SHEETS; i++) {
		if (ctl->sheets0[i].flags == 0) {
			sht = &ctl->sheets0[i];
			sht->flags = SHEET_USE; /* 使用中マーク */
			sht->height = -1; /* 非表示中 */
			return sht;
		}
	}
	return 0;	/* 全てのシートが使用中だった */
}

void sheet_setbuf(struct SHEET *sht, unsigned char *buf, int xsize, int ysize, int col_inv)
{
	sht->buf = buf;
	sht->bxsize = xsize;
	sht->bysize = ysize;
	sht->col_inv = col_inv;
	return;
}

void sheet_refreshmap(struct SHTCTL *ctl, int vx0, int vy0, int vx1, int vy1, int h0)
{
	int h, bx, by, vx, vy, bx0, by0, bx1, by1;
	unsigned char *buf, sid, *map = ctl->map;
	struct SHEET *sht;
	if (vx0 < 0) { vx0 = 0; }
	if (vy0 < 0) { vy0 = 0; }
	if (vx1 > ctl->xsize) { vx1 = ctl->xsize; }
	if (vy1 > ctl->ysize) { vy1 = ctl->ysize; }
	for (h = h0; h <= ctl->top; h++) {
		sht = ctl->sheets[h];
		sid = sht - ctl->sheets0; /* 番地を引き算してそれを下じき番号として利用 */
		buf = sht->buf;
		bx0 = vx0 - sht->vx0;
		by0 = vy0 - sht->vy0;
		bx1 = vx1 - sht->vy0;
		by1 = vy1 - sht->vy0;
		if (bx0 < 0) { bx0 = 0; }
		if (by0 < 0) { by0 = 0; }
		if (bx1 > sht->bxsize) { bx1 = sht->bxsize; }
		if (by1 > sht->bysize) { by1 = sht->bysize; }
		for (by = by0; by < by1; by++) {
			vy = sht->vy0 + by;
			for (bx = bx0; bx < bx1; bx++) {
				vx = sht->vx0 + bx;
				if (buf[by * sht->bxsize + bx] != sht->col_inv) {
					map[vy * ctl->xsize + vx] = sid;
				}
			}
		}
	}
	return;
}

void sheet_refreshsub(struct SHTCTL *ctl, int vx0, int vy0, int vx1, int vy1, int h0, int h1)
{
	int h, bx, by, vx, vy, bx0, by0, bx1, by1;
	unsigned char *buf, *vram = ctl->vram, *map = ctl->map, sid;
	struct SHEET *sht;
	/* refresh範囲が画面外にはみ出していたら補正 */
	if (vx0 < 0) { vx0 = 0; }
	if (vy0 < 0) { vy0 = 0; }
	if (vx1 > ctl->xsize) { vx1 = ctl->xsize; }
	if (vy1 > ctl->ysize) { vy1 = ctl->ysize; }
	for (h = h0; h <= h1; h++) {
		sht = ctl->sheets[h];
		buf = sht->buf;
		sid = sht - ctl->sheets0;
		/* vx0～vy1を使って、bx0～by1を逆算する */
		bx0 = vx0 - sht->vx0;
		by0 = vy0 - sht->vy0;
		bx1 = vx1 - sht->vx0;
		by1 = vy1 - sht->vy0;
		if (bx0 < 0) { bx0 = 0; }
		if (by0 < 0) { by0 = 0; }
		if (bx1 > sht->bxsize) { bx1 = sht->bxsize; }
		if (by1 > sht->bysize) { by1 = sht->bysize; }
		for (by = by0; by < by1; by++) {
			vy = sht->vy0 + by;
			for (bx = bx0; bx < bx1; bx++) {
				vx = sht->vx0 + bx;
				if (map[vy * ctl->xsize + vx] == sid) {
					vram[vy * ctl->xsize + vx] = buf[by * sht->bxsize + bx];
				}
			}
		}
	}
	return;
}

void sheet_updown(struct SHEET *sht, int height)
{
	struct SHTCTL *ctl = sht->ctl;
	int h, old = sht->height; /* 設定前の高さを記憶する */

	/* 指定が低すぎや高すぎだったら、修正する */
	if (height > ctl->top + 1) {
		height = ctl->top + 1;
	}
	if (height < -1) {
		height = -1;
	}
	sht->height = height; /* 高さを設定 */

	/* 以下は主にsheets[]の並べ替え */
	if (old > height) {	/* 以前よりも低くなる */
		if (height >= 0) {
			/* 間のものを引き上げる */
			for (h = old; h > height; h--) {
				ctl->sheets[h] = ctl->sheets[h - 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
			sheet_refreshmap(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize, height + 1);
			sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize, height + 1, old);
		} else {	/* 非表示化 */
			if (ctl->top > old) {
				/* 上になっているものをおろす */
				for (h = old; h < ctl->top; h++) {
					ctl->sheets[h] = ctl->sheets[h + 1];
					ctl->sheets[h]->height = h;
				}
			}
			ctl->top--; /* 表示中の下じきが一つ減るので、一番上の高さが減る */
			sheet_refreshmap(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize, 0);
			sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize, 0, old - 1);
		}
	} else if (old < height) {	/* 以前よりも高くなる */
		if (old >= 0) {
			/* 間のものを押し下げる */
			for (h = old; h < height; h++) {
				ctl->sheets[h] = ctl->sheets[h + 1];
				ctl->sheets[h]->height = h;
			}
			ctl->sheets[height] = sht;
		} else {	/* 非表示状態から表示状態へ */
			/* 上になるものを持ち上げる */
			for (h = ctl->top; h >= height; h--) {
				ctl->sheets[h + 1] = ctl->sheets[h];
				ctl->sheets[h + 1]->height = h + 1;
			}
			ctl->sheets[height] = sht;
			ctl->top++; /* 表示中の下じきが一つ増えるので、一番上の高さが増える */
		}
		sheet_refreshmap(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize, height);
		sheet_refreshsub(ctl, sht->vx0, sht->vy0, sht->vx0 + sht->bxsize, sht->vy0 + sht->bysize, height, height);
	}
	return;
}

void sheet_refresh(struct SHEET *sht, int bx0, int by0, int bx1, int by1)
{
	if (sht->height >= 0) { /* もしも表示中なら、新しい下じきの情報に沿って画面を描き直す */
		sheet_refreshsub(sht->ctl, sht->vx0 + bx0, sht->vy0 + by0, sht->vx0 + bx1, sht->vy0 + by1, sht->height, sht->height);
	}
	return;
}

void sheet_slide(struct SHEET *sht, int vx0, int vy0)
{
	struct SHTCTL *ctl = sht->ctl;
	int old_vx0 = sht->vx0, old_vy0 = sht->vy0;
	sht->vx0 = vx0;
	sht->vy0 = vy0;
	if (sht->height >= 0) { /* もしも表示中なら、新しい下じきの情報に沿って画面を描き直す */
		sheet_refreshmap(ctl, old_vx0, old_vy0, old_vx0 + sht->bxsize, old_vy0 + sht->bysize, 0);
		sheet_refreshmap(ctl, vx0, vy0, vx0 + sht->bxsize, vy0 + sht->bysize, sht->height);
		sheet_refreshsub(ctl, old_vx0, old_vy0, old_vx0 + sht->bxsize, old_vy0 + sht->bysize, 0, sht->height - 1);
		sheet_refreshsub(ctl, vx0, vy0, vx0 + sht->bxsize, vy0 + sht->bysize, sht->height, sht->height);
	}
	return;
}

void sheet_free(struct SHEET *sht)
{
	if (sht->height >= 0) {
		sheet_updown(sht, -1); /* 表示中ならまず非表示にする */
	}
	sht->flags = 0; /* 未使用マーク */
	return;
}
```

bootpack.h
```h
struct SHTCTL {
	unsigned char *vram, *map;
	int xsize, ysize, top;
	struct SHEET *sheets[MAX_SHEETS];
	struct SHEET sheets0[MAX_SHEETS];
};
```

本では12日目にやってきました。
今日はタイマをつけてみようと思います。
といっても起動時の処理は変わらないです。

timer.c
```c
/* タイマ関係 */

#include "bootpack.h"

#define PIT_CTRL    0x0043
#define PIT_CNT0    0x0040

void init_pit(void)
{
    io_out8(PIT_CTRL, 0x34);
    io_out8(PIT_CNT0, 0x9c);
    io_out8(PIT_CNT0, 0x2e);
    return;
}

void inthandler20(int *esp)
{
    io_out8(PIC0_OCW2, 0x60);   /* IRQ-00受付完了をPICに通知 */
    /* とりあえずなにもしない */
    return;
}
```

bootpack.c
```c
void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned int memtotal, count = 0;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse, *sht_win;
	unsigned char *buf_back, buf_mouse[256], *buf_win;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	init_pic();
	io_out8(PIC0_IMR, 0xf8); /* PIC1とキーボードを許可(11111001) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */
```

naskfunc.nas
```
GLOBAL	_io_hlt, _io_cli, _io_sti, _io_stihlt
		GLOBAL	_io_in8,  _io_in16,  _io_in32
		GLOBAL	_io_out8, _io_out16, _io_out32
		GLOBAL	_io_load_eflags, _io_store_eflags
		GLOBAL	_load_gdtr, _load_idtr
		GLOBAL	_load_cr0, _store_cr0
		GLOBAL	_asm_inthandler20, _asm_inthandler21
		GLOBAL	_asm_inthandler27, _asm_inthandler2c
		GLOBAL	_memtest_sub
		EXTERN	_inthandler20, _inthandler21
		EXTERN	_inthandler27, _inthandler2c
```

```
_asm_inthandler20:
		PUSH	ES
		PUSH	DS
		PUSHAD
		MOV		EAX,ESP
		PUSH	EAX
		MOV		AX,SS
		MOV		DS,AX
		MOV		ES,AX
		CALL	_inthandler20
		POP		EAX
		POPAD
		POP		DS
		POP		ES
		IRETD
```

dsctbl.c
```c
void init_gdtidt(void)
{
    struct SEGMENT_DESCRIPTOR *gdt = (struct SEGMENT_DESCRIPTOR *) ADR_GDT;
    struct GATE_DESCRIPTOR    *idt = (struct GATE_DESCRIPTOR    *) ADR_IDT;
    int i;

    /* GDTの初期化 */
    for (i = 0; i <= LIMIT_GDT / 8; i++) {
        set_segmdesc(gdt + i, 0, 0, 0);
    }
    set_segmdesc(gdt + 1, 0xffffffff,   0x00000000, AR_DATA32_RW);
    set_segmdesc(gdt + 2, LIMIT_BOTPAK, ADR_BOTPAK, AR_CODE32_ER);
    load_gdtr(LIMIT_GDT, ADR_GDT);

    /* IDTの初期化 */
    for (i = 0; i <= LIMIT_IDT / 8; i++) {
        set_gatedesc(idt + i, 0, 0, 0);
    }
    load_idtr(LIMIT_IDT, ADR_IDT);

    /* IDTの設定 */
    set_gatedesc(idt + 0x20, (int) asm_inthandler20, 2 * 8, AR_INTGATE32);
    set_gatedesc(idt + 0x21, (int) asm_inthandler21, 2 * 8, AR_INTGATE32);
    set_gatedesc(idt + 0x27, (int) asm_inthandler27, 2 * 8, AR_INTGATE32);
    set_gatedesc(idt + 0x2c, (int) asm_inthandler2c, 2 * 8, AR_INTGATE32);

    return;
}
```

Makefile
```Makefile
OBJS_BOOTPACK = bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj \
				int.obj fifo.obj keybord.obj mouse.obj memory.obj sheet.obj timer.obj
```

前回はタイマーを作ったので今回はそれを表示させてみたいと思います。

bootpack.h
```h
/* timer.c */
struct TIMERCTL {
	unsigned int count;
};
```

timer.c
```c
struct TIMERCTL timerctl;

void init_pit(void)
{
    io_out8(PIT_CTRL, 0x34);
    io_out8(PIT_CNT0, 0x9c);
    io_out8(PIT_CNT0, 0x2e);
    timerctl.count = 0;
    return;
}

void inthandler20(int *esp)
{
    io_out8(PIC0_OCW2, 0x60);   /* IRQ-00受付完了をPICに通知 */
    timerctl.count++;
    return;
}
```

bootpack.c
```c
void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	char s[40], keybuf[32], mousebuf[128];
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse, *sht_win;
	unsigned char *buf_back, buf_mouse[256], *buf_win;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	init_pit();
	io_out8(PIC0_IMR, 0xf8); /* PITとPIC1とキーボードを許可(11111000) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	sht_win   = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	buf_win   = (unsigned char *) memman_alloc_4k(memman, 160 * 52);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	sheet_setbuf(sht_win, buf_win, 160, 52, -1); /* 透明色なし */
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	make_window8(buf_win, 160, 52, "counter");
	sheet_slide(sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(sht_mouse, mx, my);
	sheet_slide(sht_win, 80, 72);
	sheet_updown(sht_back,  0);
	sheet_updown(sht_win,   1);
	sheet_updown(sht_mouse, 2);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 32, COL8_FFFFFF, s);
	sheet_refresh(sht_back, 0, 0, binfo->scrnx, 48);

	for (;;) {
		sprintf(s, "%010d", timerctl.count);
		boxfill8(buf_win, 160, COL8_C6C6C6, 40, 28, 119, 43);
		putfonts8_asc(buf_win, 160, 40, 28, COL8_000000, s);
		sheet_refresh(sht_win, 40, 28, 120, 44);
```

これで正確なタイマーができました。   
次は10秒経過したら10[sec]と表示さるようにします。

bootpack.h
```h
/* timer.c */
struct TIMERCTL {
	unsigned int count;
	unsigned int timeout;
	struct FIFO8 *fifo;
	unsigned char data;
};
```

timer.c
```C
void init_pit(void)
{
    io_out8(PIT_CTRL, 0x34);
    io_out8(PIT_CNT0, 0x9c);
    io_out8(PIT_CNT0, 0x2e);
    timerctl.count = 0;
    timerctl.timeout = 0;
    return;
}

void inthandler20(int *esp)
{
    io_out8(PIC0_OCW2, 0x60);   /* IRQ-00受付完了をPICに通知 */
    timerctl.count++;
    if (timerctl.timeout > 0) { /* タイムアウトが設定されている */
        timerctl.timeout--;
        if (timerctl.timeout == 0) {
            fifo8_put(timerctl.fifo, timerctl.data);
        }
    }
    return;
}

void settimer(unsigned int timeout, struct FIFO8 *fifo, unsigned char data)
{
    int eflags;
    eflags = io_load_eflags();
    io_cli();
    timerctl.timeout = timeout;
    timerctl.fifo = fifo;
    timerctl.data = data;
    io_store_eflags(eflags);
    return;
}
```

bootpack.c
```c
struct FIFO8 timerfifo;
char s[40], keybuf[32], mousebuf[128], timerbuf[8];
```

```c
fifo8_init(&timerfifo, 8, timerbuf);
settimer(1000, &timerfifo, 1);
```

```c
io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) + fifo8_status(&timerfifo) == 0) {
			io_sti();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(buf_back, binfo->scrnx, COL8_008484,  0, 16, 15, 31);
				putfonts8_asc(buf_back, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
				sheet_refresh(sht_back, 0, 16, 16, 32);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(buf_back, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					sheet_refresh(sht_back, 32, 16, 32 + 15 * 8, 32);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 0, 0, 79, 15); /* 座標消す */
					putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s); /* 座標書く */
					sheet_refresh(sht_back, 0, 0, 80, 16);
					sheet_slide(sht_mouse, mx, my);
				}
			} else if (fifo8_status(&timerfifo) != 0) {
				i = fifo8_get(&timerfifo); /* とりあえず読み込む（からにするために） */
				io_sti();
				putfonts8_asc(buf_back, binfo->scrnx, 0, 64, COL8_FFFFFF, "10[sec]");
				sheet_refresh(sht_back, 0, 64, 56, 80);
			}
		}
```

タイムアウトを設定できるタイマーをさらに増やしてみます。

bootpack.h
```h
/* timer.c */
#define MAX_TIMER		500
struct TIMER {
	unsigned int timeout, flags;
	struct FIFO8 *fifo;
	unsigned char data;
};
struct TIMERCTL {
	unsigned int count;
	struct TIMER timer[MAX_TIMER];
};
extern struct TIMERCTL timerctl;
void init_pit(void);
struct TIMER *timer_alloc(void);
void timer_free(struct TIMER *timer);
void timer_init(struct TIMER *timer, struct FIFO8 *fifo, unsigned char data);
void timer_settime(struct TIMER *timer, unsigned int timeout);
void inthandler20(int *esp);
```

timer.c
```c
/* タイマ関係 */

#include "bootpack.h"

#define PIT_CTRL    0x0043
#define PIT_CNT0    0x0040

struct TIMERCTL timerctl;

#define TIMER_FLAGS_ALLOC       1   /* 確保した状態 */
#define TIMER_FLAGS_USING       2   /* タイマ作動中 */

void init_pit(void)
{
    int i;
    io_out8(PIT_CTRL, 0x34);
    io_out8(PIT_CNT0, 0x9c);
    io_out8(PIT_CNT0, 0x2e);
    timerctl.count = 0;
    for (i = 0; i < MAX_TIMER; i++) {
        timerctl.timer[i].flags = 0; /* 未使用 */
    }
    return;
}

struct TIMER *timer_alloc(void)
{
    int i;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timer[i].flags == 0) {
            timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
            return &timerctl.timer[i];
        }
    }
    return 0; /* 見つからなかった */
}

void timer_free(struct TIMER *timer)
{
    timer->flags = 0; /* 未使用 */
    return;
}

void timer_init(struct TIMER *timer, struct FIFO8 *fifo, unsigned char data)
{
    timer->fifo = fifo;
    timer->data = data;
    return;
}

void timer_settime(struct TIMER *timer, unsigned int timeout)
{
    timer->timeout = timeout;
    timer->flags = TIMER_FLAGS_USING;
    return;
}

void inthandler20(int *esp)
{
    int i;
    io_out8(PIC0_OCW2, 0x60);   /* IRQ-00受付完了をPICに通知 */
    timerctl.count++;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timer[i].flags == TIMER_FLAGS_USING) {
            timerctl.timer[i].timeout--;
            if (timerctl.timer[i].timeout == 0) {
                timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
                fifo8_put(timerctl.timer[i].fifo, timerctl.timer[i].data);
            }
        }
    }
    return;
}
```

bootpack.c
```c
void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	struct FIFO8 timerfifo, timerfifo2, timerfifo3;
	char s[40], keybuf[32], mousebuf[128], timerbuf[8], timerbuf2[8], timerbuf3[8];
	struct TIMER *timer, *timer2, *timer3;
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse, *sht_win;
	unsigned char *buf_back, buf_mouse[256], *buf_win;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	init_pit();
	io_out8(PIC0_IMR, 0xf8); /* PITとPIC1とキーボードを許可(11111000) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	fifo8_init(&timerfifo, 8, timerbuf);
	timer = timer_alloc();
	timer_init(timer, &timerfifo, 1);
	timer_settime(timer, 1000);
	fifo8_init(&timerfifo2, 8, timerbuf2);
	timer2 = timer_alloc();
	timer_init(timer2, &timerfifo2, 1);
	timer_settime(timer2, 300);
	fifo8_init(&timerfifo3, 8, timerbuf3);
	timer3 = timer_alloc();
	timer_init(timer3, &timerfifo3, 1);
	timer_settime(timer3, 50);

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	sht_win   = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	buf_win   = (unsigned char *) memman_alloc_4k(memman, 160 * 52);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	sheet_setbuf(sht_win, buf_win, 160, 52, -1); /* 透明色なし */
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	make_window8(buf_win, 160, 52, "counter");
	sheet_slide(sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(sht_mouse, mx, my);
	sheet_slide(sht_win, 80, 72);
	sheet_updown(sht_back,  0);
	sheet_updown(sht_win,   1);
	sheet_updown(sht_mouse, 2);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc(buf_back, binfo->scrnx, 0, 32, COL8_FFFFFF, s);
	sheet_refresh(sht_back, 0, 0, binfo->scrnx, 48);

	for (;;) {
		sprintf(s, "%010d", timerctl.count);
		boxfill8(buf_win, 160, COL8_C6C6C6, 40, 28, 119, 43);
		putfonts8_asc(buf_win, 160, 40, 28, COL8_000000, s);
		sheet_refresh(sht_win, 40, 28, 120, 44);

		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) + fifo8_status(&timerfifo)
				+ fifo8_status(&timerfifo2) + fifo8_status(&timerfifo3) == 0) {
			io_sti();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				boxfill8(buf_back, binfo->scrnx, COL8_008484,  0, 16, 15, 31);
				putfonts8_asc(buf_back, binfo->scrnx, 0, 16, COL8_FFFFFF, s);
				sheet_refresh(sht_back, 0, 16, 16, 32);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 32, 16, 32 + 15 * 8 - 1, 31);
					putfonts8_asc(buf_back, binfo->scrnx, 32, 16, COL8_FFFFFF, s);
					sheet_refresh(sht_back, 32, 16, 32 + 15 * 8, 32);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 0, 0, 79, 15); /* 座標消す */
					putfonts8_asc(buf_back, binfo->scrnx, 0, 0, COL8_FFFFFF, s); /* 座標書く */
					sheet_refresh(sht_back, 0, 0, 80, 16);
					sheet_slide(sht_mouse, mx, my);
				}
			} else if (fifo8_status(&timerfifo) != 0) {
				i = fifo8_get(&timerfifo); /* とりあえず読み込む（からにするために） */
				io_sti();
				putfonts8_asc(buf_back, binfo->scrnx, 0, 64, COL8_FFFFFF, "10[sec]");
				sheet_refresh(sht_back, 0, 64, 56, 80);
			} else if (fifo8_status(&timerfifo2) != 0) {
				i = fifo8_get(&timerfifo2); /* とりあえず読み込む（からにするために） */
				io_sti();
				putfonts8_asc(buf_back, binfo->scrnx, 0, 80, COL8_FFFFFF, "3[sec]");
				sheet_refresh(sht_back, 0, 80, 48, 96);
			} else if (fifo8_status(&timerfifo3) != 0) {
				i = fifo8_get(&timerfifo3);
				io_sti();
				if (i != 0) {
					timer_init(timer3, &timerfifo3, 0); /* 次は0を */
					boxfill8(buf_back, binfo->scrnx, COL8_FFFFFF, 8, 96, 15, 111);
				} else {
					timer_init(timer3, &timerfifo3, 1); /* 次は1を */
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 8, 96, 15, 111);
				}
				timer_settime(timer3, 50);
				sheet_refresh(sht_back, 8, 96, 16, 112);
			}
		}
	}
}
```

これでさらにタイマーが増えました。   
割り込み処理のスピードを早くするために改造を加えます。

timer.c
```c
void timer_settime(struct TIMER *timer, unsigned int timeout)
{
	timer->timeout = timeout + timerctl.count;
	timer->flags = TIMER_FLAGS_USING;
	return;
}

void inthandler20(int *esp)
{
	int i;
	io_out8(PIC0_OCW2, 0x60);	/* IRQ-00受付完了をPICに通知 */
	timerctl.count++;
	for (i = 0; i < MAX_TIMER; i++) {
		if (timerctl.timer[i].flags == TIMER_FLAGS_USING) {
			if (timerctl.timer[i].timeout <= timerctl.count) {
				timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
				fifo8_put(timerctl.timer[i].fifo, timerctl.timer[i].data);
			}
		}
	}
	return;
}
```

割り込み処理をさらに早くしたいと思います。   
bootpack.h
```h
struct TIMERCTL {
	unsigned int count, next;
	struct TIMER timer[MAX_TIMER];
};
```

timer.c
```c
/* タイマ関係 */

#include "bootpack.h"

#define PIT_CTRL    0x0043
#define PIT_CNT0    0x0040

struct TIMERCTL timerctl;

#define TIMER_FLAGS_ALLOC       1   /* 確保した状態 */
#define TIMER_FLAGS_USING       2   /* タイマ作動中 */

void init_pit(void)
{
    int i;
    io_out8(PIT_CTRL, 0x34);
    io_out8(PIT_CNT0, 0x9c);
    io_out8(PIT_CNT0, 0x2e);
    timerctl.count = 0;
    timerctl.next = 0xffffffff; /* 最初は作動中のタイマがないので */
    for (i = 0; i < MAX_TIMER; i++) {
        timerctl.timer[i].flags = 0; /* 未使用 */
    }
    return;
}

struct TIMER *timer_alloc(void)
{
    int i;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timer[i].flags == 0) {
            timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
            return &timerctl.timer[i];
        }
    }
    return 0; /* 見つからなかった */
}

void timer_free(struct TIMER *timer)
{
    timer->flags = 0; /* 未使用 */
    return;
}

void timer_init(struct TIMER *timer, struct FIFO8 *fifo, unsigned char data)
{
    timer->fifo = fifo;
    timer->data = data;
    return;
}

void timer_settime(struct TIMER *timer, unsigned int timeout)
{
    timer->timeout = timeout + timerctl.count;
    timer->flags = TIMER_FLAGS_USING;
    if (timerctl.next > timer->timeout) {
        /* 次回の時刻を更新 */
        timerctl.next = timer->timeout;
    }
    return;
}

void inthandler20(int *esp)
{
    int i;
    io_out8(PIC0_OCW2, 0x60);	/* IRQ-00受付完了をPICに通知 */
    timerctl.count++;
    if (timerctl.next > timerctl.count) {
        return; /* まだ次の時刻になってないので、もうおしまい */
    }
    timerctl.next = 0xffffffff;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timer[i].flags == TIMER_FLAGS_USING) {
            if (timerctl.timer[i].timeout <= timerctl.count) {
                /* タイムアウト */
                timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
                fifo8_put(timerctl.timer[i].fifo, timerctl.timer[i].data);
            } else {
                /* まだタイムアウトではない */
                if (timerctl.next > timerctl.timer[i].timeout) {
                    timerctl.next = timerctl.timer[i].timeout;
                }
            }
        }
    }
    return;
}
```

これで少しは早くなりました。   
今度はもっと早くしてみようと思います。

bootpack.h
```h
struct TIMERCTL {
	unsigned int count, next, using;
	struct TIMER *timers[MAX_TIMER];
	struct TIMER timers0[MAX_TIMER];
};
```

timer.c
```c
/* タイマ関係 */

#include "bootpack.h"

#define PIT_CTRL    0x0043
#define PIT_CNT0    0x0040

struct TIMERCTL timerctl;

#define TIMER_FLAGS_ALLOC       1   /* 確保した状態 */
#define TIMER_FLAGS_USING       2   /* タイマ作動中 */

void init_pit(void)
{
    int i;
    io_out8(PIT_CTRL, 0x34);
    io_out8(PIT_CNT0, 0x9c);
    io_out8(PIT_CNT0, 0x2e);
    timerctl.count = 0;
    timerctl.next = 0xffffffff; /* 最初は作動中のタイマがないので */
    timerctl.using = 0;
    for (i = 0; i < MAX_TIMER; i++) {
        timerctl.timers0[i].flags = 0; /* 未使用 */
    }
    return;
}

struct TIMER *timer_alloc(void)
{
    int i;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timers0[i].flags == 0) {
            timerctl.timers0[i].flags = TIMER_FLAGS_ALLOC;
            return &timerctl.timers0[i];
        }
    }
    return 0; /* 見つからなかった */
}

void timer_free(struct TIMER *timer)
{
    timer->flags = 0; /* 未使用 */
    return;
}

void timer_init(struct TIMER *timer, struct FIFO8 *fifo, unsigned char data)
{
    timer->fifo = fifo;
    timer->data = data;
    return;
}

void timer_settime(struct TIMER *timer, unsigned int timeout)
{
    int e, i, j;
    timer->timeout = timeout + timerctl.count;
    timer->flags = TIMER_FLAGS_USING;
    e = io_load_eflags();
    io_cli();
    /* どこに入れればいいかを探す */
    for (i = 0; i < timerctl.using; i++) {
        if (timerctl.timers[i]->timeout >= timer->timeout) {
            break;
        }
    }
    /* うしろをずらす */
    for (j = timerctl.using; j > i; j--) {
        timerctl.timers[j] = timerctl.timers[j - 1];
    }
    timerctl.using++;
    /* あいたすきまに入れる */
    timerctl.timers[i] = timer;
    timerctl.next = timerctl.timers[0]->timeout;
    io_store_eflags(e);
    return;
}

void inthandler20(int *esp)
{
    int i, j;
    io_out8(PIC0_OCW2, 0x60);	/* IRQ-00受付完了をPICに通知 */
    timerctl.count++;
    if (timerctl.next > timerctl.count) {
        return;
    }
    for (i = 0; i < timerctl.using; i++) {
        /* timersのタイマは全て動作中のものなので、flagsを確認しない */
        if (timerctl.timers[i]->timeout > timerctl.count) {
            break;
        }
        /* タイムアウト */
        timerctl.timers[i]->flags = TIMER_FLAGS_ALLOC;
        fifo8_put(timerctl.timers[i]->fifo, timerctl.timers[i]->data);
    }
    /* ちょうどi個のタイマがタイムアウトした。残りをずらす。 */
    timerctl.using -= i;
    for (j = 0; j < timerctl.using; j++) {
        timerctl.timers[j] = timerctl.timers[i + j];
    }
    if (timerctl.using > 0) {
        timerctl.next = timerctl.timers[0]->timeout;
    } else {
        timerctl.next = 0xffffffff;
    }
    return;
}
```

これでさらにはやくなりました。   

**実装過程**   
・　[e0e9aa0b10946ca9abda1418215153ff4ad9299d](e0e9aa0b10946ca9abda1418215153ff4ad9299d)

## day13
本でも13日目に突入しました。   
今回は文字列表示を簡単にしてbootpack.cのソースコードを簡単にします。   

bootpack.c
```c
void putfonts8_asc_sht(struct SHEET *sht, int x, int y, int c, int b, char *s, int l);

void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	struct FIFO8 timerfifo, timerfifo2, timerfifo3;
	char s[40], keybuf[32], mousebuf[128], timerbuf[8], timerbuf2[8], timerbuf3[8];
	struct TIMER *timer, *timer2, *timer3;
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse, *sht_win;
	unsigned char *buf_back, buf_mouse[256], *buf_win;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	init_pit();
	io_out8(PIC0_IMR, 0xf8); /* PITとPIC1とキーボードを許可(11111000) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	fifo8_init(&timerfifo, 8, timerbuf);
	timer = timer_alloc();
	timer_init(timer, &timerfifo, 1);
	timer_settime(timer, 1000);
	fifo8_init(&timerfifo2, 8, timerbuf2);
	timer2 = timer_alloc();
	timer_init(timer2, &timerfifo2, 1);
	timer_settime(timer2, 300);
	fifo8_init(&timerfifo3, 8, timerbuf3);
	timer3 = timer_alloc();
	timer_init(timer3, &timerfifo3, 1);
	timer_settime(timer3, 50);

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	sht_win   = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	buf_win   = (unsigned char *) memman_alloc_4k(memman, 160 * 52);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	sheet_setbuf(sht_win, buf_win, 160, 52, -1); /* 透明色なし */
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	make_window8(buf_win, 160, 52, "counter");
	sheet_slide(sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(sht_mouse, mx, my);
	sheet_slide(sht_win, 80, 72);
	sheet_updown(sht_back,  0);
	sheet_updown(sht_win,   1);
	sheet_updown(sht_mouse, 2);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc_sht(sht_back, 0, 0, COL8_FFFFFF, COL8_008484, s, 10);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc_sht(sht_back, 0, 32, COL8_FFFFFF, COL8_008484, s, 40);

	for (;;) {
		sprintf(s, "%010d", timerctl.count);
		putfonts8_asc_sht(sht_win, 40, 28, COL8_000000, COL8_C6C6C6, s, 10);

		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) + fifo8_status(&timerfifo)
				+ fifo8_status(&timerfifo2) + fifo8_status(&timerfifo3) == 0) {
			io_sti();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				putfonts8_asc_sht(sht_back, 0, 16, COL8_FFFFFF, COL8_008484, s, 2);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					putfonts8_asc_sht(sht_back, 32, 16, COL8_FFFFFF, COL8_008484, s, 15);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					putfonts8_asc_sht(sht_back, 0, 0, COL8_FFFFFF, COL8_008484, s, 10);
					sheet_slide(sht_mouse, mx, my);
				}
			} else if (fifo8_status(&timerfifo) != 0) {
				i = fifo8_get(&timerfifo); /* とりあえず読み込む（からにするために） */
				io_sti();
				putfonts8_asc_sht(sht_back, 0, 64, COL8_FFFFFF, COL8_008484, "10[sec]", 7);
			} else if (fifo8_status(&timerfifo2) != 0) {
				i = fifo8_get(&timerfifo2); /* とりあえず読み込む（からにするために） */
				io_sti();
				putfonts8_asc_sht(sht_back, 0, 80, COL8_FFFFFF, COL8_008484, "3[sec]", 6);
			} else if (fifo8_status(&timerfifo3) != 0) {
				i = fifo8_get(&timerfifo3);
				io_sti();
				if (i != 0) {
					timer_init(timer3, &timerfifo3, 0); /* 次は0を */
					boxfill8(buf_back, binfo->scrnx, COL8_FFFFFF, 8, 96, 15, 111);
				} else {
					timer_init(timer3, &timerfifo3, 1); /* 次は1を */
					boxfill8(buf_back, binfo->scrnx, COL8_008484, 8, 96, 15, 111);
				}
				timer_settime(timer3, 50);
				sheet_refresh(sht_back, 8, 96, 16, 112);
			}
		}
	}
}
```

```c
void putfonts8_asc_sht(struct SHEET *sht, int x, int y, int c, int b, char *s, int l)
{
	boxfill8(sht->buf, sht->bxsize, b, x, y, x + l * 8 - 1, y + 15);
	putfonts8_asc(sht->buf, sht->bxsize, x, y, c, s);
	sheet_refresh(sht, x, y, x + l * 8, y + 16);
	return;
}
```

今回もHariMainのコード数を削減していきます。   
FIFOバッファを再構成してみます。   
bootpack.c
```c
void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	struct FIFO8 timerfifo;
	char s[40], keybuf[32], mousebuf[128], timerbuf[8];
	struct TIMER *timer, *timer2, *timer3;
	int mx, my, i;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse, *sht_win;
	unsigned char *buf_back, buf_mouse[256], *buf_win;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	init_pit();
	io_out8(PIC0_IMR, 0xf8); /* PITとPIC1とキーボードを許可(11111000) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	fifo8_init(&timerfifo, 8, timerbuf);
	timer = timer_alloc();
	timer_init(timer, &timerfifo, 10);
	timer_settime(timer, 1000);
	timer2 = timer_alloc();
	timer_init(timer2, &timerfifo, 3);
	timer_settime(timer2, 300);
	timer3 = timer_alloc();
	timer_init(timer3, &timerfifo, 1);
	timer_settime(timer3, 50);

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	sht_win   = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	buf_win   = (unsigned char *) memman_alloc_4k(memman, 160 * 52);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	sheet_setbuf(sht_win, buf_win, 160, 52, -1); /* 透明色なし */
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	make_window8(buf_win, 160, 52, "counter");
	sheet_slide(sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(sht_mouse, mx, my);
	sheet_slide(sht_win, 80, 72);
	sheet_updown(sht_back,  0);
	sheet_updown(sht_win,   1);
	sheet_updown(sht_mouse, 2);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc_sht(sht_back, 0, 0, COL8_FFFFFF, COL8_008484, s, 10);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc_sht(sht_back, 0, 32, COL8_FFFFFF, COL8_008484, s, 40);

	for (;;) {
		sprintf(s, "%010d", timerctl.count);
		putfonts8_asc_sht(sht_win, 40, 28, COL8_000000, COL8_C6C6C6, s, 10);

		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) + fifo8_status(&timerfifo) == 0) {
			io_sti();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				putfonts8_asc_sht(sht_back, 0, 16, COL8_FFFFFF, COL8_008484, s, 2);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					putfonts8_asc_sht(sht_back, 32, 16, COL8_FFFFFF, COL8_008484, s, 15);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					putfonts8_asc_sht(sht_back, 0, 0, COL8_FFFFFF, COL8_008484, s, 10);
					sheet_slide(sht_mouse, mx, my);
				}
			} else if (fifo8_status(&timerfifo) != 0) {
				i = fifo8_get(&timerfifo); /* タイムアウトしたのはどれかな？ */
				io_sti();
				if (i == 10) {
					putfonts8_asc_sht(sht_back, 0, 64, COL8_FFFFFF, COL8_008484, "10[sec]", 7);	
				} else if (i == 3) {
					putfonts8_asc_sht(sht_back, 0, 80, COL8_FFFFFF, COL8_008484, "3[sec]", 6);
				} else {
					/* 0か1 */
					if (i != 0) {
						timer_init(timer3, &timerfifo, 0); /* 次は0を */
						boxfill8(buf_back, binfo->scrnx, COL8_FFFFFF, 8, 96, 15, 111);
					} else {
						timer_init(timer3, &timerfifo, 1); /* 次は1を */
						boxfill8(buf_back, binfo->scrnx, COL8_008484, 8, 96, 15, 111);
					}
					timer_settime(timer3, 50);
					sheet_refresh(sht_back, 8, 96, 16, 112);
				}
			}
		}
	}
}
```

タイマーを改良して少しずつ速度を上げていきます。   
(ここからしばらくはタイマーの改良に尽力します)   
bootpack.c
```c
void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	struct FIFO8 timerfifo;
	char s[40], keybuf[32], mousebuf[128], timerbuf[8];
	struct TIMER *timer, *timer2, *timer3;
	int mx, my, i, count = 0;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse, *sht_win;
	unsigned char *buf_back, buf_mouse[256], *buf_win;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo8_init(&keyfifo, 32, keybuf);
	fifo8_init(&mousefifo, 128, mousebuf);
	init_pit();
	io_out8(PIC0_IMR, 0xf8); /* PITとPIC1とキーボードを許可(11111000) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	fifo8_init(&timerfifo, 8, timerbuf);
	timer = timer_alloc();
	timer_init(timer, &timerfifo, 10);
	timer_settime(timer, 1000);
	timer2 = timer_alloc();
	timer_init(timer2, &timerfifo, 3);
	timer_settime(timer2, 300);
	timer3 = timer_alloc();
	timer_init(timer3, &timerfifo, 1);
	timer_settime(timer3, 50);

	init_keyboard();
	enable_mouse(&mdec);
	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	sht_win   = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	buf_win   = (unsigned char *) memman_alloc_4k(memman, 160 * 52);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	sheet_setbuf(sht_win, buf_win, 160, 52, -1); /* 透明色なし */
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	make_window8(buf_win, 160, 52, "counter");
	sheet_slide(sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(sht_mouse, mx, my);
	sheet_slide(sht_win, 80, 72);
	sheet_updown(sht_back,  0);
	sheet_updown(sht_win,   1);
	sheet_updown(sht_mouse, 2);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc_sht(sht_back, 0, 0, COL8_FFFFFF, COL8_008484, s, 10);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc_sht(sht_back, 0, 32, COL8_FFFFFF, COL8_008484, s, 40);

	for (;;) {
		count++;

		io_cli();
		if (fifo8_status(&keyfifo) + fifo8_status(&mousefifo) + fifo8_status(&timerfifo) == 0) {
			io_sti();
		} else {
			if (fifo8_status(&keyfifo) != 0) {
				i = fifo8_get(&keyfifo);
				io_sti();
				sprintf(s, "%02X", i);
				putfonts8_asc_sht(sht_back, 0, 16, COL8_FFFFFF, COL8_008484, s, 2);
			} else if (fifo8_status(&mousefifo) != 0) {
				i = fifo8_get(&mousefifo);
				io_sti();
				if (mouse_decode(&mdec, i) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					putfonts8_asc_sht(sht_back, 32, 16, COL8_FFFFFF, COL8_008484, s, 15);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					putfonts8_asc_sht(sht_back, 0, 0, COL8_FFFFFF, COL8_008484, s, 10);
					sheet_slide(sht_mouse, mx, my);
				}
			} else if (fifo8_status(&timerfifo) != 0) {
				i = fifo8_get(&timerfifo); /* タイムアウトしたのはどれかな？ */
				io_sti();
				if (i == 10) {
					putfonts8_asc_sht(sht_back, 0, 64, COL8_FFFFFF, COL8_008484, "10[sec]", 7);	
					sprintf(s, "%010d", count);
					putfonts8_asc_sht(sht_win, 40, 28, COL8_000000, COL8_C6C6C6, s, 10);
				} else if (i == 3) {
					putfonts8_asc_sht(sht_back, 0, 80, COL8_FFFFFF, COL8_008484, "3[sec]", 6);
					count = 0; /* 測定開始 */
				} else {
					/* 0か1 */
					if (i != 0) {
						timer_init(timer3, &timerfifo, 0); /* 次は0を */
						boxfill8(buf_back, binfo->scrnx, COL8_FFFFFF, 8, 96, 15, 111);
					} else {
						timer_init(timer3, &timerfifo, 1); /* 次は1を */
						boxfill8(buf_back, binfo->scrnx, COL8_008484, 8, 96, 15, 111);
					}
					timer_settime(timer3, 50);
					sheet_refresh(sht_back, 8, 96, 16, 112);
				}
			}
		}
	}
}

hari09dの頃のtimer.cとbootpack.hを使って時間を測定してみましょう。   
(ここからtimer.cとbootpack.hは過去のソースコード使用しています。)   
timer.c
```c
/* タイマ関係 */

#include "bootpack.h"

#define PIT_CTRL    0x0043
#define PIT_CNT0    0x0040

struct TIMERCTL timerctl;

#define TIMER_FLAGS_ALLOC       1   /* 確保した状態 */
#define TIMER_FLAGS_USING       2   /* タイマ作動中 */

void init_pit(void)
{
    int i;
    io_out8(PIT_CTRL, 0x34);
    io_out8(PIT_CNT0, 0x9c);
    io_out8(PIT_CNT0, 0x2e);
    timerctl.count = 0;
    for (i = 0; i < MAX_TIMER; i++) {
        timerctl.timer[i].flags = 0; /* 未使用 */
    }
    return;
}

struct TIMER *timer_alloc(void)
{
    int i;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timer[i].flags == 0) {
            timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
            return &timerctl.timer[i];
        }
    }
    return 0; /* 見つからなかった */
}

void timer_free(struct TIMER *timer)
{
    timer->flags = 0; /* 未使用 */
    return;
}

void timer_init(struct TIMER *timer, struct FIFO8 *fifo, unsigned char data)
{
    timer->fifo = fifo;
    timer->data = data;
    return;
}

void timer_settime(struct TIMER *timer, unsigned int timeout)
{
    timer->timeout = timeout;
    timer->flags = TIMER_FLAGS_USING;
    return;
}

void inthandler20(int *esp)
{
    int i;
    io_out8(PIC0_OCW2, 0x60);	/* IRQ-00受付完了をPICに通知 */
    timerctl.count++;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timer[i].flags == TIMER_FLAGS_USING) {
            timerctl.timer[i].timeout--;
            if (timerctl.timer[i].timeout == 0) {
                timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
                fifo8_put(timerctl.timer[i].fifo, timerctl.timer[i].data);
            }
        }
    }
    return;
}
```

bootpack.h
```h
struct TIMERCTL {
	unsigned int count;
	struct TIMER timer[MAX_TIMER];
};
```

残り時間をやめて、タイムアウトを時刻を導入して時間を測定しています。   
timer.c
```c
void inthandler20(int *esp)
{
    int i;
    io_out8(PIC0_OCW2, 0x60);	/* IRQ-00受付完了をPICに通知 */
    timerctl.count++;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timer[i].flags == TIMER_FLAGS_USING) {
            if (timerctl.timer[i].timeout <= timerctl.count) {
                timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
                fifo8_put(timerctl.timer[i].fifo, timerctl.timer[i].data);
            }
        }
    }
    return;
}
```

nextを追加したバージョンで時間を測定したいと思います。   
timer.c
```c
/* タイマ関係 */

#include "bootpack.h"

#define PIT_CTRL	0x0043
#define PIT_CNT0	0x0040

struct TIMERCTL timerctl;

#define TIMER_FLAGS_ALLOC		1	/* 確保した状態 */
#define TIMER_FLAGS_USING		2	/* タイマ作動中 */

void init_pit(void)
{
    int i;
    io_out8(PIT_CTRL, 0x34);
    io_out8(PIT_CNT0, 0x9c);
    io_out8(PIT_CNT0, 0x2e);
    timerctl.count = 0;
    timerctl.next = 0xffffffff; /* 最初は作動中のタイマがないので */
    for (i = 0; i < MAX_TIMER; i++) {
        timerctl.timer[i].flags = 0; /* 未使用 */
    }
    return;
}

struct TIMER *timer_alloc(void)
{
    int i;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timer[i].flags == 0) {
            timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
            return &timerctl.timer[i];
        }
    }
    return 0; /* 見つからなかった */
}

void timer_free(struct TIMER *timer)
{
    timer->flags = 0; /* 未使用 */
    return;
}

void timer_init(struct TIMER *timer, struct FIFO8 *fifo, unsigned char data)
{
    timer->fifo = fifo;
    timer->data = data;
    return;
}

void timer_settime(struct TIMER *timer, unsigned int timeout)
{
    timer->timeout = timeout + timerctl.count;
    timer->flags = TIMER_FLAGS_USING;
    if (timerctl.next > timer->timeout) {
        /* 次回の時刻を更新 */
        timerctl.next = timer->timeout;
    }
    return;
}

void inthandler20(int *esp)
{
    int i;
    io_out8(PIC0_OCW2, 0x60);	/* IRQ-00受付完了をPICに通知 */
    timerctl.count++;
    if (timerctl.next > timerctl.count) {
        return; /* まだ次の時刻になってないので、もうおしまい */
    }
    timerctl.next = 0xffffffff;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timer[i].flags == TIMER_FLAGS_USING) {
            if (timerctl.timer[i].timeout <= timerctl.count) {
                /* タイムアウト */
                timerctl.timer[i].flags = TIMER_FLAGS_ALLOC;
                fifo8_put(timerctl.timer[i].fifo, timerctl.timer[i].data);
            } else {
                /* まだタイムアウトではない */
                if (timerctl.next > timerctl.timer[i].timeout) {
                    timerctl.next = timerctl.timer[i].timeout;
                }
            }
        }
    }
    return;
}
```

bootpack.h
```h
struct TIMERCTL {
	unsigned int count, next;
	struct TIMER timer[MAX_TIMER];
};
```

性能がどんどんよくなってることがわかりました。   
harib10cに戻して続きを書いていきます。   

キーボードとマウスも一つのバッファに纏めて管理したいと思います。   
bootpack.c
```c
void HariMain(void)
{
	struct BOOTINFO *binfo = (struct BOOTINFO *) ADR_BOOTINFO;
	struct FIFO32 fifo;
	char s[40];
	int fifobuf[128];
	struct TIMER *timer, *timer2, *timer3;
	int mx, my, i, count = 0;
	unsigned int memtotal;
	struct MOUSE_DEC mdec;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct SHTCTL *shtctl;
	struct SHEET *sht_back, *sht_mouse, *sht_win;
	unsigned char *buf_back, buf_mouse[256], *buf_win;

	init_gdtidt();
	init_pic();
	io_sti(); /* IDT/PICの初期化が終わったのでCPUの割り込み禁止を解除 */
	fifo32_init(&fifo, 128, fifobuf);
	init_pit();
	init_keyboard(&fifo, 256);
	enable_mouse(&fifo, 512, &mdec);
	io_out8(PIC0_IMR, 0xf8); /* PITとPIC1とキーボードを許可(11111000) */
	io_out8(PIC1_IMR, 0xef); /* マウスを許可(11101111) */

	timer = timer_alloc();
	timer_init(timer, &fifo, 10);
	timer_settime(timer, 1000);
	timer2 = timer_alloc();
	timer_init(timer2, &fifo, 3);
	timer_settime(timer2, 300);
	timer3 = timer_alloc();
	timer_init(timer3, &fifo, 1);
	timer_settime(timer3, 50);

	memtotal = memtest(0x00400000, 0xbfffffff);
	memman_init(memman);
	memman_free(memman, 0x00001000, 0x0009e000); /* 0x00001000 - 0x0009efff */
	memman_free(memman, 0x00400000, memtotal - 0x00400000);

	init_palette();
	shtctl = shtctl_init(memman, binfo->vram, binfo->scrnx, binfo->scrny);
	sht_back  = sheet_alloc(shtctl);
	sht_mouse = sheet_alloc(shtctl);
	sht_win   = sheet_alloc(shtctl);
	buf_back  = (unsigned char *) memman_alloc_4k(memman, binfo->scrnx * binfo->scrny);
	buf_win   = (unsigned char *) memman_alloc_4k(memman, 160 * 52);
	sheet_setbuf(sht_back, buf_back, binfo->scrnx, binfo->scrny, -1); /* 透明色なし */
	sheet_setbuf(sht_mouse, buf_mouse, 16, 16, 99);
	sheet_setbuf(sht_win, buf_win, 160, 52, -1); /* 透明色なし */
	init_screen8(buf_back, binfo->scrnx, binfo->scrny);
	init_mouse_cursor8(buf_mouse, 99);
	make_window8(buf_win, 160, 52, "counter");
	sheet_slide(sht_back, 0, 0);
	mx = (binfo->scrnx - 16) / 2; /* 画面中央になるように座標計算 */
	my = (binfo->scrny - 28 - 16) / 2;
	sheet_slide(sht_mouse, mx, my);
	sheet_slide(sht_win, 80, 72);
	sheet_updown(sht_back,  0);
	sheet_updown(sht_win,   1);
	sheet_updown(sht_mouse, 2);
	sprintf(s, "(%3d, %3d)", mx, my);
	putfonts8_asc_sht(sht_back, 0, 0, COL8_FFFFFF, COL8_008484, s, 10);
	sprintf(s, "memory %dMB   free : %dKB",
			memtotal / (1024 * 1024), memman_total(memman) / 1024);
	putfonts8_asc_sht(sht_back, 0, 32, COL8_FFFFFF, COL8_008484, s, 40);

	for (;;) {
		count++;

		io_cli();
		if (fifo32_status(&fifo) == 0) {
			io_sti();
		} else {
			i = fifo32_get(&fifo);
				io_sti();
			if (256 <= i && i <= 511) { /* キーボードデータ */
			sprintf(s, "%02X", i - 256);
				putfonts8_asc_sht(sht_back, 0, 16, COL8_FFFFFF, COL8_008484, s, 2);
			} else if (512 <= i && i <= 767) { /* マウスデータ */
				if (mouse_decode(&mdec, i - 512) != 0) {
					/* データが3バイト揃ったので表示 */
					sprintf(s, "[lcr %4d %4d]", mdec.x, mdec.y);
					if ((mdec.btn & 0x01) != 0) {
						s[1] = 'L';
					}
					if ((mdec.btn & 0x02) != 0) {
						s[3] = 'R';
					}
					if ((mdec.btn & 0x04) != 0) {
						s[2] = 'C';
					}
					putfonts8_asc_sht(sht_back, 32, 16, COL8_FFFFFF, COL8_008484, s, 15);
					/* マウスカーソルの移動 */
					mx += mdec.x;
					my += mdec.y;
					if (mx < 0) {
						mx = 0;
					}
					if (my < 0) {
						my = 0;
					}
					if (mx > binfo->scrnx - 1) {
						mx = binfo->scrnx - 1;
					}
					if (my > binfo->scrny - 1) {
						my = binfo->scrny - 1;
					}
					sprintf(s, "(%3d, %3d)", mx, my);
					putfonts8_asc_sht(sht_back, 0, 0, COL8_FFFFFF, COL8_008484, s, 10);
					sheet_slide(sht_mouse, mx, my);
				}
			} else if (i == 10) { /* 10秒タイマ */
				putfonts8_asc_sht(sht_back, 0, 64, COL8_FFFFFF, COL8_008484, "10[sec]", 7);	
				sprintf(s, "%010d", count);
				putfonts8_asc_sht(sht_win, 40, 28, COL8_000000, COL8_C6C6C6, s, 10);
			} else if (i == 3) { /* 3秒タイマ */
				putfonts8_asc_sht(sht_back, 0, 80, COL8_FFFFFF, COL8_008484, "3[sec]", 6);
				count = 0; /* 測定開始 */
			} else if (i == 1) { /* カーソル用タイマ */
				timer_init(timer3, &fifo, 0); /* 次は0を */
				boxfill8(buf_back, binfo->scrnx, COL8_FFFFFF, 8, 96, 15, 111);
				timer_settime(timer3, 50);
				sheet_refresh(sht_back, 8, 96, 16, 112);
			} else if (i == 0) { /* カーソル用タイマ */
				timer_init(timer3, &fifo, 1); /* 次は1を */
				boxfill8(buf_back, binfo->scrnx, COL8_008484, 8, 96, 15, 111);
				timer_settime(timer3, 50);
				sheet_refresh(sht_back, 8, 96, 16, 112);
			}
		}
	}
}
```

timer.c
```c
/* タイマ関係 */

#include "bootpack.h"

#define PIT_CTRL	0x0043
#define PIT_CNT0	0x0040

struct TIMERCTL timerctl;

#define TIMER_FLAGS_ALLOC		1	/* 確保した状態 */
#define TIMER_FLAGS_USING		2	/* タイマ作動中 */

void init_pit(void)
{
    int i;
    io_out8(PIT_CTRL, 0x34);
    io_out8(PIT_CNT0, 0x9c);
    io_out8(PIT_CNT0, 0x2e);
    timerctl.count = 0;
    timerctl.next = 0xffffffff; /* 最初は作動中のタイマがないので */
    timerctl.using = 0;
    for (i = 0; i < MAX_TIMER; i++) {
        timerctl.timers0[i].flags = 0; /* 未使用 */
    }
    return;
}

struct TIMER *timer_alloc(void)
{
    int i;
    for (i = 0; i < MAX_TIMER; i++) {
        if (timerctl.timers0[i].flags == 0) {
            timerctl.timers0[i].flags = TIMER_FLAGS_ALLOC;
            return &timerctl.timers0[i];
        }
    }
    return 0; /* 見つからなかった */
}

void timer_free(struct TIMER *timer)
{
    timer->flags = 0; /* 未使用 */
    return;
}

void timer_init(struct TIMER *timer, struct FIFO32 *fifo, int data)
{
    timer->fifo = fifo;
    timer->data = data;
    return;
}

void timer_settime(struct TIMER *timer, unsigned int timeout)
{
    int e, i, j;
    timer->timeout = timeout + timerctl.count;
    timer->flags = TIMER_FLAGS_USING;
    e = io_load_eflags();
    io_cli();
    /* どこに入れればいいかを探す */
    for (i = 0; i < timerctl.using; i++) {
        if (timerctl.timers[i]->timeout >= timer->timeout) {
            break;
        }
    }
    /* うしろをずらす */
    for (j = timerctl.using; j > i; j--) {
        timerctl.timers[j] = timerctl.timers[j - 1];
    }
    timerctl.using++;
    /* あいたすきまに入れる */
    timerctl.timers[i] = timer;
    timerctl.next = timerctl.timers[0]->timeout;
    io_store_eflags(e);
    return;
}

void inthandler20(int *esp)
{
    int i, j;
    io_out8(PIC0_OCW2, 0x60);	/* IRQ-00受付完了をPICに通知 */
    timerctl.count++;
    if (timerctl.next > timerctl.count) {
        return;
    }
    for (i = 0; i < timerctl.using; i++) {
        /* timersのタイマは全て動作中のものなので、flagsを確認しない */
        if (timerctl.timers[i]->timeout > timerctl.count) {
            break;
        }
        /* タイムアウト */
        timerctl.timers[i]->flags = TIMER_FLAGS_ALLOC;
        fifo32_put(timerctl.timers[i]->fifo, timerctl.timers[i]->data);
    }
    /* ちょうどi個のタイマがタイムアウトした。残りをずらす。 */
    timerctl.using -= i;
    for (j = 0; j < timerctl.using; j++) {
        timerctl.timers[j] = timerctl.timers[i + j];
    }
    if (timerctl.using > 0) {
        timerctl.next = timerctl.timers[0]->timeout;
    } else {
        timerctl.next = 0xffffffff;
    }
    return;
}
```

mouse.c
```c
/* マウス関係 */

#include "bootpack.h"

struct FIFO32 *mousefifo;
int mousedata0;

void inthandler2c(int *esp)
/* PS/2マウスからの割り込み */
{
	int data;
	io_out8(PIC1_OCW2, 0x64);	/* IRQ-12受付完了をPIC1に通知 */
	io_out8(PIC0_OCW2, 0x62);	/* IRQ-02受付完了をPIC0に通知 */
	data = io_in8(PORT_KEYDAT);
	fifo32_put(mousefifo, data + mousedata0);
	return;
}

#define KEYCMD_SENDTO_MOUSE		0xd4
#define MOUSECMD_ENABLE			0xf4

void enable_mouse(struct FIFO32 *fifo, int data0, struct MOUSE_DEC *mdec)
{
	/* 書き込み先のFIFOバッファを記憶 */
	mousefifo = fifo;
	mousedata0 = data0;
	/* マウス有効 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_SENDTO_MOUSE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, MOUSECMD_ENABLE);
	/* うまくいくとACK(0xfa)が送信されてくる */
	mdec->phase = 0; /* マウスの0xfaを待っている段階 */
	return;
}

int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat)
{
	if (mdec->phase == 0) {
		/* マウスの0xfaを待っている段階 */
		if (dat == 0xfa) {
			mdec->phase = 1;
		}
		return 0;
	}
	if (mdec->phase == 1) {
		/* マウスの1バイト目を待っている段階 */
		if ((dat & 0xc8) == 0x08) {
			/* 正しい1バイト目だった */
			mdec->buf[0] = dat;
			mdec->phase = 2;
		}
		return 0;
	}
	if (mdec->phase == 2) {
		/* マウスの2バイト目を待っている段階 */
		mdec->buf[1] = dat;
		mdec->phase = 3;
		return 0;
	}
	if (mdec->phase == 3) {
		/* マウスの3バイト目を待っている段階 */
		mdec->buf[2] = dat;
		mdec->phase = 1;
		mdec->btn = mdec->buf[0] & 0x07;
		mdec->x = mdec->buf[1];
		mdec->y = mdec->buf[2];
		if ((mdec->buf[0] & 0x10) != 0) {
			mdec->x |= 0xffffff00;
		}
		if ((mdec->buf[0] & 0x20) != 0) {
			mdec->y |= 0xffffff00;
		}
		mdec->y = - mdec->y; /* マウスではy方向の符号が画面と反対 */
		return 1;
	}
	return -1; /* ここに来ることはないはず */
}
```

keybord.c
```c
/* キーボード関係 */

#include "bootpack.h"

struct FIFO32 *keyfifo;
int keydata0;

void inthandler21(int *esp)
{
	int data;
	io_out8(PIC0_OCW2, 0x61);	/* IRQ-01受付完了をPICに通知 */
	data = io_in8(PORT_KEYDAT);
	fifo32_put(keyfifo, data + keydata0);
	return;
}

#define PORT_KEYSTA				0x0064
#define KEYSTA_SEND_NOTREADY	0x02
#define KEYCMD_WRITE_MODE		0x60
#define KBC_MODE				0x47

void wait_KBC_sendready(void)
{
	/* キーボードコントローラがデータ送信可能になるのを待つ */
	for (;;) {
		if ((io_in8(PORT_KEYSTA) & KEYSTA_SEND_NOTREADY) == 0) {
			break;
		}
	}
	return;
}

void init_keyboard(struct FIFO32 *fifo, int data0)
{
	/* 書き込み先のFIFOバッファを記憶 */
	keyfifo = fifo;
	keydata0 = data0;
	/* キーボードコントローラの初期化 */
	wait_KBC_sendready();
	io_out8(PORT_KEYCMD, KEYCMD_WRITE_MODE);
	wait_KBC_sendready();
	io_out8(PORT_KEYDAT, KBC_MODE);
	return;
}
```

fifo.c
```c
/* FIFOライブラリ */

#include "bootpack.h"

#define FLAGS_OVERRUN		0x0001

void fifo32_init(struct FIFO32 *fifo, int size, int *buf)
/* FIFOバッファの初期化 */
{
	fifo->size = size;
	fifo->buf = buf;
	fifo->free = size; /* 空き */
	fifo->flags = 0;
	fifo->p = 0; /* 書き込み位置 */
	fifo->q = 0; /* 読み込み位置 */
	return;
}

int fifo32_put(struct FIFO32 *fifo, int data)
/* FIFOへデータを送り込んで蓄える */
{
	if (fifo->free == 0) {
		/* 空きがなくてあふれた */
		fifo->flags |= FLAGS_OVERRUN;
		return -1;
	}
	fifo->buf[fifo->p] = data;
	fifo->p++;
	if (fifo->p == fifo->size) {
		fifo->p = 0;
	}
	fifo->free--;
	return 0;
}

int fifo32_get(struct FIFO32 *fifo)
/* FIFOからデータを一つとってくる */
{
	int data;
	if (fifo->free == fifo->size) {
		/* バッファが空っぽのときは、とりあえず-1が返される */
		return -1;
	}
	data = fifo->buf[fifo->q];
	fifo->q++;
	if (fifo->q == fifo->size) {
		fifo->q = 0;
	}
	fifo->free++;
	return data;
}

int fifo32_status(struct FIFO32 *fifo)
/* どのくらいデータが溜まっているかを報告する */
{
	return fifo->size - fifo->free;
}
```

bootpack.h
```h
/* fifo.c */
struct FIFO32 {
	int *buf;
	int p, q, size, free, flags;
};
void fifo32_init(struct FIFO32 *fifo, int size, int *buf);
int fifo32_put(struct FIFO32 *fifo, int data);
int fifo32_get(struct FIFO32 *fifo);
int fifo32_status(struct FIFO32 *fifo);

(中略)
/* keyboard.c */
void inthandler21(int *esp);
void wait_KBC_sendready(void);
void init_keyboard(struct FIFO32 *fifo, int data0);
#define PORT_KEYDAT		0x0060
#define PORT_KEYCMD		0x0064

(中略)
/* mouse.c */
struct MOUSE_DEC {
	unsigned char buf[3], phase;
	int x, y, btn;
};
void inthandler2c(int *esp);
void enable_mouse(struct FIFO32 *fifo, int data0, struct MOUSE_DEC *mdec);
int mouse_decode(struct MOUSE_DEC *mdec, unsigned char dat);

(中略)
/* timer.c */
#define MAX_TIMER		500
struct TIMER {
	unsigned int timeout, flags;
	struct FIFO32 *fifo;
	int data;
};
struct TIMERCTL {
	unsigned int count, next, using;
	struct TIMER *timers[MAX_TIMER];
	struct TIMER timers0[MAX_TIMER];
};
extern struct TIMERCTL timerctl;
void init_pit(void);
struct TIMER *timer_alloc(void);
void timer_free(struct TIMER *timer);
void timer_init(struct TIMER *timer, struct FIFO32 *fifo, int data);
void timer_settime(struct TIMER *timer, unsigned int timeout);
void inthandler20(int *esp);
```

この改造をおこなうことで割り込みの速度も早くなりました。

**実装過程**   
・　[0fd5c03332195a404214e9953cc1471a5320e933](0fd5c03332195a404214e9953cc1471a5320e933)