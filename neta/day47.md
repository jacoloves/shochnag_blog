## day47
MMLプレイヤーを作成していきます。   
mmlplay/Makefile
```Makefile
APP      = mmlplay
STACK    = 132k
MALLOC   = 0k

include ../app_make.txt
```

mmlplay.c
```c
#include "apilib.h"

#include <string.h>	/* strlen */

int strtol(char *s, char **endp, int base);	/* 標準関数（stdlib.h） */

void waittimer(int timer, int time);
void end(char *s);

void HariMain(void)
{
    char winbuf[256 * 112], txtbuf[100 * 1024];
    char s[32], *p, *r;
    int win, timer, i, j, t = 120, l = 192 / 4, o = 4, q = 7, note_old = 0;

    /* 音番号と周波数(mHz)の対応表 */
	/* たとえば、O4Aは440Hzなので、440000 */
	/* オクターブ16のAは1802240Hzなので、1802240000 */
	/* 以下はオクターブ16のリスト(C～B) */
    static int tonetable[12] = {
        1071618315, 1135340056, 1202850889, 1274376125, 1350154473, 1430438836,
        1515497155, 1605613306, 1701088041, 1802240000, 1909406767, 2022946002
    };
    static int notetable[7] = { +9, +11, +0 /* C */, +2, +4, +5, +7 };

    /* コマンドライン解析 */
    api_cmdline(s, 30);
    for (p = s; *p > ' '; p++) { }	/* スペースが来るまで読み飛ばす */
    for (; *p == ' '; p++) { }	/* スペースを読み飛ばす */
    i = strlen(p);
    if (i > 12) {
file_error:
        end("file open error.\n");
    }
    if (i == 0) {
        end(0);
    }

    /* ウィンドウの準備 */
    win = api_openwin(winbuf, 256, 112, -1, "mmlplay");
    api_putstrwin(win, 128, 32, 0, i, p);
    api_boxfilwin(win, 8, 60, 247,  76, 7);
    api_boxfilwin(win, 6, 86, 249, 105, 7);

    /* ファイル読み込み */
    i = api_fopen(p);
    if (i == 0) {
        goto file_error;
    }
    j = api_fsize(i, 0);
    if (j >= 100 * 1024) {
        j = 100 * 1024 - 1;
    }
    api_fread(txtbuf, j, i);
    api_fclose(i);
    txtbuf[j] = 0;
    r = txtbuf;
    i = 0; /* 通常モード */
    for (p = txtbuf; *p != 0; p++) {	/* 処理を簡単にするためにコメントや空白を消す */
        if (i == 0 && *p > ' ') {	/* スペースや改行コードなどではない */
            if (*p == '/') {
                if (p[1] == '*') {
                    i = 1;
                } else if (p[1] == '/') {
                    i = 2;
                } else {
                    *r = *p;
                    if ('a' <= *p && *p <= 'z') {
                        *r += 'A' - 'a';	/* 小文字は大文字に変換 */
                    }
                    r++;
                }
            } else if (*p == 0x22) {
                *r = *p;
                r++;
                i = 3;
            } else {
                *r = *p;
                r++;
            }
        } else if (i == 1 && *p == '*' && p[1] == '/') {	/* ブロックコメント */
            p++;
            i = 0;
        } else if (i == 2 && *p == 0x0a) {	/* 行コメント */
            i = 0;
        } else if (i == 3) {	/* 文字列 */
            *r = *p;
            r++;
            if (*p == 0x22) {
                i = 0;
            } else if (*p == '%') {
                p++;
                *r = *p;
                r++;
            }
        }
    }
    *r = 0;

    /* タイマの準備 */
    timer = api_alloctimer();
    api_inittimer(timer, 128);

    /* メイン */
    p = txtbuf;
    for (;;) {
        if (('A' <= *p && *p <= 'G') || *p == 'R') {	/* 音符・休符 */
            /* 周波数計算 */
            if (*p == 'R') {
                i = 0;
                s[0] = 0;
            } else {
                i = o * 12 + notetable[*p - 'A'] + 12;
                s[0] = 'O';
                s[1] = '0' + o;
                s[2] = *p;
                s[3] = ' ';
                s[4] = 0;
            }
            p++;
            if (*p == '+' || *p == '-' || *p == '#') {
                s[3] = *p;
                if (*p == '-') {
                    i--;
                } else {
                    i++;
                }
                p++;
            }
            if (i != note_old) {
                api_boxfilwin(win + 1, 32, 36, 63, 51, 8);
                if (s[0] != 0) {
                    api_putstrwin(win + 1, 32, 36, 10, 4, s);
                }
                api_refreshwin(win, 32, 36, 64, 52);
                if (28 <= note_old && note_old <= 107) {
                    api_boxfilwin(win, (note_old - 28) * 3 + 8, 60, (note_old - 28) * 3 + 10,  76, 7);
                }
                if (28 <= i && i <= 107) {
                    api_boxfilwin(win, (i - 28) * 3 + 8, 60, (i - 28) * 3 + 10,  76, 4);
                }
                if (s[0] != 0) {
                    api_beep(tonetable[i % 12] >> (17 - i / 12));
                } else {
                    api_beep(0);
                }
                note_old = i;
            }
            /* 音長計算 */
            if ('0' <= *p && *p <= '9') {
                i = 192 / strtol(p, &p, 10);
            } else {
                i = l;
            }
            for (; *p == '.'; ) {
                p++;
                i += i / 2;
            }
            i *= (60 * 100 / 48);
            i /= t;
            if (s[0] != 0 && q < 8 && *p != '&') {
                j = i * q / 8;
                waittimer(timer, j);
                api_boxfilwin(win, 32, 36, 63, 51, 8);
                if (28 <= note_old && note_old <= 107) {
                    api_boxfilwin(win, (note_old - 28) * 3 + 8, 60, (note_old - 28) * 3 + 10,  76, 7);
                }
                note_old = 0;
                api_beep(0);
            } else {
                j = 0;
                if (*p == '&') {
                    p++;
                }
            }
            waittimer(timer, i - j);
        } else if (*p == '<') {	/* オクターブ-- */
            p++;
            o--;
        } else if (*p == '>') {	/* オクターブ++ */
            p++;
            o++;
        } else if (*p == 'O') {	/* オクターブ指定 */
            o = strtol(p + 1, &p, 10);
        } else if (*p == 'Q') {	/* Qパラメータ指定 */
            q = strtol(p + 1, &p, 10);
        } else if (*p == 'L') { /* デフォルト音長指定 */ 
            l = strtol(p + 1, &p, 10);
            if (l == 0) {
                goto syntax_error;
            }
            l = 192 / l;
            for (; *p == '.'; ) {
                p++;
                l += l / 2;
            }
        } else if (*p == 'T') {	/* テンポ指定 */
            t = strtol(p + 1, &p, 10);
        } else if (*p == '$') {	/* 拡張コマンド */
            if (p[1] == 'K') {	/* カラオケコマンド */
                p += 2;
                for (; *p != 0x22; p++) {
                    if (*p == 0) {
                        goto syntax_error;
                    }
                }
                p++;
                for (i = 0; i < 32; i++) {
                    if (*p == 0) {
                        goto syntax_error;
                    }
                    if (*p == 0x22) {
                        break;
                    }
                    if (*p == '%') {
                        s[i] = p[1];
                        p += 2;
                    } else {
                        s[i] = *p;
                        p++;
                    }
                }
                if (i > 30) {
                    end("karaoke too long.\n");
                }
                api_boxfilwin(win + 1, 8, 88, 247, 103, 7);
                s[i] = 0;
                if (i != 0) {
                    api_putstrwin(win + 1, 128 - i * 4, 88, 0, i, s);
                }
                api_refreshwin(win, 8, 88, 248, 104);
            }
            for (; *p != ';'; p++) {
                if (*p == 0) {
                    goto syntax_error;
                }
            }
            p++;
        } else if (*p == 0) {
            p = txtbuf;
        } else {
syntax_error:
            end("mml syntax error.\n");
        }
    }
}

void waittimer(int timer, int time)
{
    int i;
    api_settimer(timer, time);
    for (;;) {
        i = api_getkey(1);
        if (i == 'Q' || i == 'q') {
            api_beep(0);
            api_end();
        }
        if (i == 128) {
            return;
        }
    }
}

void end(char *s)
{
    if (s != 0) {
        api_putstr0(s);
    }
    api_beep(0);
    api_end();
}
```

Makefile
```Makefile
TOOLPATH = ..\..\z_tools\\

INCPATH  = ..\..\z_tools\haribote\\

MAKE     = $(TOOLPATH)make.exe -r
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
COPY     = cmd.exe /C copy
DEL      = rm -f

# default operation

default :
	$(MAKE) haribote.img

# file making rule

haribote.img : haribote/ipl20.bin haribote/haribote.sys Makefile \
		a/a.hrb hello3/hello3.hrb hello4/hello4.hrb hello5/hello5.hrb \
		winhelo/winhelo.hrb winhelo2/winhelo2.hrb winhelo3/winhelo3.hrb \
		star1/star1.hrb stars/stars.hrb stars2/stars2.hrb \
		lines/lines.hrb walk/walk.hrb noodle/noodle.hrb \
		beepdown/beepdown.hrb color/color.hrb color2/color2.hrb \
		sosu/sosu.hrb sosu2/sosu2.hrb sosu3/sosu3.hrb \
		type/type.hrb iroha/iroha.hrb chklang/chklang.hrb \
		notrec/notrec.hrb bball/bball.hrb invader/invader.hrb \
		calc/calc.hrb tview/tview.hrb mmlplay/mmlplay.hrb
	$(EDIMG)   imgin:../../z_tools/fdimg0at.tek \
		wbinimg src:haribote/ipl20.bin len:512 from:0 to:0 \
		copy from:haribote/haribote.sys to:@: \
		copy from:haribote/ipl20.nas to:@: \
		copy from:make.bat to:@: \
		copy from:a/a.hrb to:@: \
		copy from:hello3/hello3.hrb to:@: \
		copy from:hello4/hello4.hrb to:@: \
		copy from:hello5/hello5.hrb to:@: \
		copy from:winhelo/winhelo.hrb to:@: \
		copy from:winhelo2/winhelo2.hrb to:@: \
		copy from:winhelo3/winhelo3.hrb to:@: \
		copy from:star1/star1.hrb to:@: \
		copy from:stars/stars.hrb to:@: \
		copy from:stars2/stars2.hrb to:@: \
		copy from:lines/lines.hrb to:@: \
		copy from:walk/walk.hrb to:@: \
		copy from:noodle/noodle.hrb to:@: \
		copy from:beepdown/beepdown.hrb to:@: \
		copy from:color/color.hrb to:@: \
		copy from:color2/color2.hrb to:@: \
		copy from:sosu/sosu.hrb to:@: \
		copy from:sosu2/sosu2.hrb to:@: \
		copy from:sosu3/sosu3.hrb to:@: \
		copy from:type/type.hrb to:@: \
		copy from:iroha/iroha.hrb to:@: \
		copy from:chklang/chklang.hrb to:@: \
		copy from:euc.txt to:@: \
		copy from:notrec/notrec.hrb to:@: \
		copy from:bball/bball.hrb to:@: \
		copy from:invader/invader.hrb to:@: \
		copy from:calc/calc.hrb to:@: \
		copy from:tview/tview.hrb to:@: \
		copy from:mmlplay/mmlplay.hrb to:@: \
		copy from:mmldata/kirakira.mml to:@: \
		copy from:mmldata/fujisan.mml to:@: \
		copy from:mmldata/daigo.mml to:@: \
		copy from:mmldata/daiku.mml to:@: \
		copy from:nihongo/nihongo.fnt to:@: \
		imgout:haribote.img

# command

run :
	$(MAKE) haribote.img
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install :
	$(MAKE) haribote.img
	$(IMGTOL) w a: haribote.img

full :
	$(MAKE) -C haribote
	$(MAKE) -C apilib
	$(MAKE) -C a
	$(MAKE) -C hello3
	$(MAKE) -C hello4
	$(MAKE) -C hello5
	$(MAKE) -C winhelo
	$(MAKE) -C winhelo2
	$(MAKE) -C winhelo3
	$(MAKE) -C star1
	$(MAKE) -C stars
	$(MAKE) -C stars2
	$(MAKE) -C lines
	$(MAKE) -C walk
	$(MAKE) -C noodle
	$(MAKE) -C beepdown
	$(MAKE) -C color
	$(MAKE) -C color2
	$(MAKE) -C sosu
	$(MAKE) -C sosu2
	$(MAKE) -C sosu3
	$(MAKE) -C type
	$(MAKE) -C iroha
	$(MAKE) -C chklang
	$(MAKE) -C notrec
	$(MAKE) -C bball
	$(MAKE) -C invader
	$(MAKE) -C calc
	$(MAKE) -C tview
	$(MAKE) -C mmlplay
	$(MAKE) haribote.img

run_full :
	$(MAKE) full
	$(COPY) haribote.img ..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\z_tools\qemu

install_full :
	$(MAKE) full
	$(IMGTOL) w a: haribote.img

run_os :
	$(MAKE) -C haribote
	$(MAKE) run

clean :
# 何もしない

src_only :
	$(MAKE) clean
	-$(DEL) haribote.img
	
clean_full :
	$(MAKE) -C haribote		clean
	$(MAKE) -C apilib		clean
	$(MAKE) -C a			clean
	$(MAKE) -C hello3		clean
	$(MAKE) -C hello4		clean
	$(MAKE) -C hello5		clean
	$(MAKE) -C winhelo		clean
	$(MAKE) -C winhelo2		clean
	$(MAKE) -C winhelo3		clean
	$(MAKE) -C star1		clean
	$(MAKE) -C stars		clean
	$(MAKE) -C stars2		clean
	$(MAKE) -C lines		clean
	$(MAKE) -C walk			clean	
	$(MAKE) -C noodle		clean
	$(MAKE) -C beepdown		clean
	$(MAKE) -C color		clean
	$(MAKE) -C color2		clean
	$(MAKE) -C sosu			clean
	$(MAKE) -C sosu2		clean
	$(MAKE) -C sosu3		clean
	$(MAKE) -C type			clean
	$(MAKE) -C iroha		clean
	$(MAKE) -C chklang		clean
	$(MAKE) -C notrec		clean
	$(MAKE) -C bball		clean
	$(MAKE) -C invader		clean
	$(MAKE) -C calc			clean
	$(MAKE) -C tview		clean
	$(MAKE) -C mmlplay		clean

src_only_full :
	$(MAKE) -C haribote		src_only
	$(MAKE) -C apilib		src_only
	$(MAKE) -C a			src_only
	$(MAKE) -C hello3		src_only
	$(MAKE) -C hello4		src_only
	$(MAKE) -C hello5		src_only
	$(MAKE) -C winhelo		src_only
	$(MAKE) -C winhelo2		src_only
	$(MAKE) -C winhelo3		src_only
	$(MAKE) -C star1		src_only
	$(MAKE) -C stars		src_only
	$(MAKE) -C stars2		src_only
	$(MAKE) -C lines		src_only
	$(MAKE) -C walk			src_only	
	$(MAKE) -C noodle		src_only
	$(MAKE) -C beepdown		src_only
	$(MAKE) -C color		src_only
	$(MAKE) -C color2		src_only
	$(MAKE) -C sosu			src_only
	$(MAKE) -C sosu2		src_only
	$(MAKE) -C sosu3		src_only
	$(MAKE) -C type			src_only
	$(MAKE) -C iroha		src_only
	$(MAKE) -C chklang		src_only
	$(MAKE) -C notrec		src_only
	$(MAKE) -C bball		src_only
	$(MAKE) -C invader		src_only
	$(MAKE) -C calc			src_only
	$(MAKE) -C tview		src_only
	$(MAKE) -C mmlplay		src_only
	-$(DEL) haribote.img

refresh :
	$(MAKE) full
	$(MAKE) clean_full
	-$(DEL) haribote.img
```

**実装過程**   
・　[29bbc10d4d38ce0da456c9eb27b91212cb408b49](29bbc10d4d38ce0da456c9eb27b91212cb408b49)