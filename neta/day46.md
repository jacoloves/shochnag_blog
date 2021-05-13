## day46
今日はテキストビューアを作っていきます。    

Makefile
```Makefile
APP      = tview
STACK    = 1024k
MALLOC   = 0k

include ../app_make.txt
```

tview.c
```c
#include "apilib.h"

#include <stdio.h>

int strtol(char *s, char **endp, int base);	/* 標準関数（stdlib.h） */

char *skipspace(char *p);
void textview(int win, int w, int h, int xskip, char *p, int tab, int lang);
char *lineview(int win, int w, int y, int xskip, unsigned char *p, int tab, int lang);
int puttab(int x, int w, int xskip, char *s, int tab);

void HariMain(void)
{
    char winbuf[1024 * 757], txtbuf[240 * 1024];
    int w = 30, h = 10, t = 4, spd_x = 1, spd_y = 1;
    int win, i, j, lang = api_getlang(), xskip = 0;
    char s[30], *p, *q = 0, *r = 0;

    /* コマンドライン解析 */
    api_cmdline(s, 30);
    for (p = s; *p > ' '; p++) { }	/* スペースが来るまで読み飛ばす */
    for (; *p != 0; ) {
        p = skipspace(p);
        if (*p == '-') {
            if (p[1] == 'w') {
                w = strtol(p + 2, &p, 0);
                if (w < 20) {
                    w = 20;
                }
                if (w > 126) {
                    w = 126;
                }
            } else if (p[1] == 'h') {
                h = strtol(p + 2, &p, 0);
                if (h < 1) {
                    h = 1;
                }
                if (h > 45) {
                    h = 45;
                }
            } else if (p[1] == 't') {
                t = strtol(p + 2, &p, 0);
                if (t < 1) {
                    t = 4;
                }
            } else {
err:
                api_putstr0(" >tview file [-w30 -h10 -t4]\n");
                api_end();
            }
        } else {	/* ファイル名発見 */
            if (q != 0) {
                goto err;
            }
            q = p;
            for (; *p > ' '; p++) { }	/* スペースが来るまで読み飛ばす */
            r = p;
        }
    }
    if (q == 0) {
        goto err;
    }

    /* ウィンドウの準備 */
    win = api_openwin(winbuf, w * 8 + 16, h * 16 + 37, -1, "tview");
    api_boxfilwin(win, 6, 27, w * 8 + 9, h * 16 + 30, 7);

    /* ファイル読み込み */
    *r = 0;
    i = api_fopen(q);
    if (i == 0) {
        api_putstr0("file open error.\n");
        api_end();
    }
    j = api_fsize(i, 0);
    if (j >= 240 * 1024 - 1) {
        j = 240 * 1024 - 2;
    }
    txtbuf[0] = 0x0a; /* 番兵用の改行コード */
    api_fread(txtbuf + 1, j, i);
    api_fclose(i);
    txtbuf[j + 1] = 0;
    q = txtbuf + 1;
    for (p = txtbuf + 1; *p != 0; p++) {	/* 処理を簡単にするために0x0dのコードを消す */
        if (*p != 0x0d) {
            *q = *p;
            q++;
        }
    }
    *q = 0;

    /* メイン */
    p = txtbuf + 1;
    for (;;) {
        textview(win, w, h, xskip, p, t, lang);
        i = api_getkey(1);
        if (i == 'Q' || i == 'q') {
            api_end();
        }
        if ('A' <= i && i <= 'F') {
            spd_x = 1 << (i - 'A');	/* 1, 2, 4, 8, 16, 32 */
        }
        if ('a' <= i && i <= 'f') {
            spd_y = 1 << (i - 'a');	/* 1, 2, 4, 8, 16, 32 */
        }
        if (i == '<' && t > 1) {
            t /= 2;
        }
        if (i == '>' && t < 256) {
            t *= 2;
        }
        if (i == '4') {
            for (;;) {
                xskip -= spd_x;
                if (xskip < 0) {
                    xskip = 0;
                }
                if (api_getkey(0) != '4') { /* もう'4'を押していなければ、処理終わり */
                    break;
                }
            }
        }
        if (i == '6') {
            for (;;) {
                xskip += spd_x;
                if (api_getkey(0) != '6') {
                    break;
                }
            }
        }
        if (i == '8') {
            for (;;) {
                for (j = 0; j < spd_y; j++) {
                    if (p == txtbuf + 1) {
                        break;
                    }
                    for (p--; p[-1] != 0x0a; p--) { } /* 一文字前に0x0aがでるまでさかのぼる */
                }
                if (api_getkey(0) != '8') {
                    break;
                }
            }
        }
        if (i == '2') {
            for (;;) {
                for (j = 0; j < spd_y; j++) {
                    for (q = p; *q != 0 && *q != 0x0a; q++) { }
                    if (*q == 0) {
                        break;
                    }
                    p = q + 1;
                }
                if (api_getkey(0) != '2') {
                    break;
                }
            }
        }
    }
}

char *skipspace(char *p)
{
    for (; *p == ' '; p++) { }	/* スペースを読み飛ばす */
    return p;
}

void textview(int win, int w, int h, int xskip, char *p, int tab, int lang)
{
    int i;
    api_boxfilwin(win + 1, 8, 29, w * 8 + 7, h * 16 + 28, 7);
    for (i = 0; i < h; i++) {
        p = lineview(win, w, i * 16 + 29, xskip, p, tab, lang);
    }
    api_refreshwin(win, 8, 29, w * 8 + 8, h * 16 + 29);
    return;
}

char *lineview(int win, int w, int y, int xskip, unsigned char *p, int tab, int lang)
{
    int x = - xskip;
    char s[130];
    for (;;) {
        if (*p == 0) {
            break;
        }
        if (*p == 0x0a) {
            p++;
            break;
        }
        if (lang == 0) {	/* ASCII */
            if (*p == 0x09) {
                x = puttab(x, w, xskip, s, tab);
            } else {
                if (0 <= x && x < w) {
                    s[x] = *p;
                }
                x++;
            }
            p++;
        }
        if (lang == 1) {	/* SJIS */
            if (*p == 0x09) {
                x = puttab(x, w, xskip, s, tab);
                p++;
            } else if ((0x81 <= *p && *p <= 0x9f) || (0xe0 <= *p && *p <= 0xfc)) {
                /* 全角文字 */
                if (x == -1) {
                    s[0] = ' ';
                }
                if (0 <= x && x < w - 1) {
                    s[x]     = *p;
                    s[x + 1] = p[1];
                }
                if (x == w - 1) {
                    s[x] = ' ';
                }
                x += 2;
                p += 2;
            } else {
                if (0 <= x && x < w) {
                    s[x] = *p;
                }
                x++;
                p++;
            }
        }
        if (lang == 2) {	/* EUC */
            if (*p == 0x09) {
                x = puttab(x, w, xskip, s, tab);
                p++;
            } else if (0xa1 <= *p && *p <= 0xfe) {
                /* 全角文字 */
                if (x == -1) {
                    s[0] = ' ';
                }
                if (0 <= x && x < w - 1) {
                    s[x]     = *p;
                    s[x + 1] = p[1];
                }
                if (x == w - 1) {
                    s[x] = ' ';
                }
                x += 2;
                p += 2;
            } else {
                if (0 <= x && x < w) {
                    s[x] = *p;
                }
                x++;
                p++;
            }
        }
    }
    if (x > w) {
        x = w;
    }
    if (x > 0) {
        s[x] = 0;
        api_putstrwin(win + 1, 8, y, 0, x, s);
    }
    return p;
}

int puttab(int x, int w, int xskip, char *s, int tab)
{
    for (;;) {
        if (0 <= x && x < w) {
            s[x] = ' ';
        }
        x++;
        if ((x + xskip) % tab == 0) {
            break;
        }
    }
    return x;
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
		calc/calc.hrb tview/tview.hrb
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
	-$(DEL) haribote.img

refresh :
	$(MAKE) full
	$(MAKE) clean_full
	-$(DEL) haribote.img
```

**実装過程**   
・　[b3bd0b881255f40036f6e19dcf3c55fbcd5baa8b](b3bd0b881255f40036f6e19dcf3c55fbcd5baa8b)