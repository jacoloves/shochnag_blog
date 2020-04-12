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