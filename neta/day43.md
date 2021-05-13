## day43
本では29日目に突入しました。
まずは全角文字のバグを修正する所から始めていきます。   
window.c
```c
void putfonts8_asc_sht(struct SHEET *sht, int x, int y, int c, int b, char *s, int l)
{
	struct TASK *task = task_now();
	boxfill8(sht->buf, sht->bxsize, b, x, y, x + l * 8 - 1, y + 15);
	if (task->langmode != 0 && task->langbyte1 != 0) {
		putfonts8_asc(sht->buf, sht->bxsize, x, y, c, s);
		sheet_refresh(sht, x - 8, y, x + l * 8, y + 16);
	} else {
		putfonts8_asc(sht->buf, sht->bxsize, x, y, c, s);
		sheet_refresh(sht, x, y, x + l * 8, y + 16);
	}
	return;
}
```

次は暗号化ツールを作っていきます。    
tek.c
```c
#include "bootpack.h"
#include <setjmp.h>
#include <string.h>
#define NULL		0

typedef unsigned char UCHAR;
typedef unsigned int UINT32;
typedef UINT32 tek_TPRB;

static int tek_decode1(int siz, UCHAR *p, UCHAR *q);
static int tek_decode2(int siz, UCHAR *p, UCHAR *q);
static int tek_decode5(int siz, UCHAR *p, UCHAR *q);

static unsigned int tek_getnum_s7s(UCHAR **pp)
/* これは必ずbig-endian */
/* 下駄がないので中身をいじりやすい */
{
	unsigned int s = 0;
	UCHAR *p = *pp;
	do {
		s = s << 7 | *p++;
	} while ((s & 1) == 0);
	s >>= 1;
	*pp = p;
	return s;
}

int tek_getsize(unsigned char *p)
{
	static char header[15] = {
		0xff, 0xff, 0xff, 0x01, 0x00, 0x00, 0x00, 0x4f, 0x53, 0x41, 0x53, 0x4b, 0x43, 0x4d, 0x50
	};
	int size = -1;
	if (memcmp(p + 1, header, 15) == 0 && (*p == 0x83 || *p == 0x85 || *p == 0x89)) {
		p += 16;
		size = tek_getnum_s7s(&p);
	}
	return size;
}	  /* （註）memcmpはstrncmpの仲間で、文字列中に0があっても指定された15文字まで比較する関数 */

int tek_decomp(unsigned char *p, char *q, int size)
{
	int err = -1;
	if (*p == 0x83) {
		err = tek_decode1(size, p, q);
	} else if (*p == 0x85) {
		err = tek_decode2(size, p, q);
	} else if (*p == 0x89) {
		err = tek_decode5(size, p, q);
	}
	if (err != 0) {
		return -1;	/* 失敗 */
	}
	return 0;	/* 成功 */
}

static int tek_lzrestore_stk1(int srcsiz, UCHAR *src, int outsiz, UCHAR *q)
{
	int by, lz, cp, ds;
	UCHAR *q1 = q + outsiz, *s7ptr = src, *q0 = q;
	do {
		if ((by = (lz = *s7ptr++) & 0x0f) == 0)
			by = tek_getnum_s7s(&s7ptr);
		if ((lz >>= 4) == 0)
			lz = tek_getnum_s7s(&s7ptr);
		do {
			*q++ = *s7ptr++;
		} while (--by);
		if (q >= q1)
			break;
		do {
			ds = (cp = *s7ptr++) & 0x0f;
			if ((ds & 1) == 0) {
				do {
					ds = ds << 7 | *s7ptr++;
				} while ((ds & 1) == 0);
			}
			ds = ~(ds >> 1);
			if ((cp >>= 4) == 0) {
				do {
					cp = cp << 7 | *s7ptr++;
				} while ((cp & 1) == 0);
				cp >>= 1;
			} /* 0がこないことをあてにする */
			cp++;
			if (q + ds < q0)
				goto err;
			if (q + cp > q1)
				cp = q1 - q;
			do {
				*q = *(q + ds);
				q++;
			} while (--cp);
		} while (--lz);
	} while (q < q1);
	return 0;
err:
	return 1;
}

static int tek_decode1(int siz, UCHAR *p, UCHAR *q)
{
	int dsiz, hed, bsiz;
	UCHAR *p1 = p + siz;
	p += 16;
	if ((dsiz = tek_getnum_s7s(&p)) > 0) {
		hed = tek_getnum_s7s(&p);
		bsiz = 1 << (((hed >> 1) & 0x0f) + 8);
		if (dsiz > bsiz || (hed & 0x21) != 0x01)
			return 1;
		if (hed & 0x40)
			tek_getnum_s7s(&p); /* オプション情報へのポインタを読み飛ばす */
		if (tek_getnum_s7s(&p) != 0)
			return 1; /* 補助バッファ使用 */
		return tek_lzrestore_stk1(p1 - p, p, dsiz, q);
	}
	return 0;
}

static unsigned int tek_getnum_s7(UCHAR **pp)
/* これは必ずbig-endian */
{
	unsigned int s = 0, b = 0, a = 1;
	UCHAR *p = *pp;
	for (;;) {
		s = s << 7 | *p++;
		if (s & 1)
			break;
		a <<= 7;
		b += a;
	}
	s >>= 1;
	*pp = p;
	return s + b;
}

static int tek_lzrestore_stk2(int srcsiz, UCHAR *src, int outsiz, UCHAR *q)
{
	int cp, ds, repdis[4], i, j;
	UCHAR *q1 = q + outsiz, *s7ptr = src, *q0 = q, bylz, cbylz;
	for (j = 0; j < 4; j++)
		repdis[j] = -1 - j;
	bylz = cbylz = 0;
	if (outsiz) {
		if (tek_getnum_s7s(&s7ptr))
			return 1;
		do {
			/* byフェーズ */
			j = 0;
			do {
				j++;
				if (j >= 17) {
					j += tek_getnum_s7s(&s7ptr);
					break;
				}
				if (cbylz == 0) {
					cbylz = 8;
					bylz = *s7ptr++;
				}
				cbylz--;
				i = bylz & 1;
				bylz >>= 1;
			} while (i == 0);
			do {
				*q++ = *s7ptr++;
			} while (--j);
			if (q >= q1)
				break;

			/* lzフェーズ */
			j = 0;
			do {
				j++;
				if (j >= 17) {
					j += tek_getnum_s7s(&s7ptr);
					break;
				}
				if (cbylz == 0) {
					cbylz = 8;
					bylz = *s7ptr++;
				}
				cbylz--;
				i = bylz & 1;
				bylz >>= 1;
			} while (i == 0);
			do {
				i = *s7ptr++;
				cp = i >> 4;
				i &= 0x0f;
				if ((i & 1) == 0)
					i |= (tek_getnum_s7(&s7ptr) + 1) << 4;
				i >>= 1;
				ds = ~(i - 6);
				if (i < 4)
					ds = repdis[i];
				if (i == 4)
					ds = repdis[0] - tek_getnum_s7(&s7ptr) - 1;
				if (i == 5)
					ds = repdis[0] + tek_getnum_s7(&s7ptr) + 1;
				if (cp == 0)
					cp = tek_getnum_s7(&s7ptr) + 16;
				cp++;
				if (i > 0) {
					if (i > 1) {
						if (i > 2)
							repdis[3] = repdis[2];
						repdis[2] = repdis[1];
					}
					repdis[1] = repdis[0];
					repdis[0] = ds;
				}
				if (q + ds < q0)
					goto err;
				if (q + cp > q1)
					cp = q1 - q;
				do {
					*q = *(q + ds);
					q++;
				} while (--cp);
			} while (--j);
		} while (q < q1);
	}
	return 0;
err:
	return 1;
}

static int tek_decode2(int siz, UCHAR *p, UCHAR *q)
{
	UCHAR *p1 = p + siz;
	int dsiz, hed, bsiz, st = 0;
	p += 16;
	if ((dsiz = tek_getnum_s7s(&p)) > 0) {
		hed = tek_getnum_s7s(&p);
		bsiz = 1 << (((hed >> 1) & 0x0f) + 8);
		if (dsiz > bsiz || (hed & 0x21) != 0x01)
			return 1;
		if (hed & 0x40)
			tek_getnum_s7s(&p); /* オプション情報へのポインタを読み飛ばす */
		st = tek_lzrestore_stk2(p1 - p, p, dsiz, q);
	}
	return st;
}

static int tek_decmain5(int *work, UCHAR *src, int osiz, UCHAR *q, int lc, int pb, int lp, int flags);

static int tek_lzrestore_tek5(int srcsiz, UCHAR *src, int outsiz, UCHAR *outbuf)
{
	int wrksiz, lc, lp, pb, flags, *work, prop0, fl;

	if ((fl = (prop0 = *src) & 0x0f) == 0x01) /* 0001 */
		flags |= -1;
	else if (fl == 0x05)
		flags = -2;
	else if (fl == 0x09)
		flags &= 0;
	else
		return 1;
	src++;
	prop0 >>= 4;
	if (prop0 == 0)
		prop0 = *src++;
	else {
		static UCHAR prop0_table[] = { 0x5d, 0x00 }, prop1_table[] = { 0x00 };
		if (flags == -1) {
			if (prop0 >= 3)
				return 1;
			prop0 = prop0_table[prop0 - 1];
		} else {
			if (prop0 >= 2)
				return 1;
			prop0 = prop1_table[prop0 - 1];
		}
	}
	lp = prop0 / (9 * 5);
	prop0 %= 9 * 5;
	pb = prop0 / 9;
	lc = prop0 % 9;
	if (flags == 0) /* tek5:z2 */
		flags = *src++;
	if (flags == -1) { /* stk5 */
		wrksiz = lp;
		lp = pb;
		pb = wrksiz;
	}
	wrksiz = 0x180 * sizeof (UINT32) + (0x840 + (0x300 << (lc + lp))) * sizeof (tek_TPRB); /* 最低15KB, lc+lp=3なら、36KB */
	work = (int *) memman_alloc_4k((struct MEMMAN *) MEMMAN_ADDR, wrksiz);
	if (work == NULL)
		return -1;
	flags = tek_decmain5(work, src, outsiz, outbuf, lc, pb, lp, flags);
	memman_free_4k((struct MEMMAN *) MEMMAN_ADDR, (int) work, wrksiz);
	return flags;
}

struct tek_STR_BITMODEL {
	UCHAR t, m, s, dmy;
	UINT32 prb0, prb1, tmsk, ntm, lt, lt0, dmy4;
};

struct tek_STR_PRB {
	struct tek_STR_PRB_PB {
		struct tek_STR_PRB_PBST {
			tek_TPRB mch, rep0l1;
		} st[12];
		tek_TPRB lenlow[2][8], lenmid[2][8];
	} pb[16];
	struct tek_STR_PRB_ST {
		tek_TPRB rep, repg0, repg1, repg2;
	} st[12];
	tek_TPRB lensel[2][2], lenhigh[2][256], pslot[4][64], algn[64];
	tek_TPRB spdis[2][2+4+8+16+32], lenext[2+4+8+16+32];
	tek_TPRB repg3, fchgprm[2 * 32], tbmt[16], tbmm[16], fchglt;
	tek_TPRB lit[1];
};

struct tek_STR_RNGDEC {
	UCHAR *p;
	UINT32 range, code, rmsk;
	jmp_buf errjmp;
	struct tek_STR_BITMODEL bm[32], *ptbm[16];
	struct tek_STR_PRB probs;
};

static void tek_setbm5(struct tek_STR_BITMODEL *bm, int t, int m)
{
	bm->t = t;
	bm->m = m;
	bm->prb1 = -1 << (m + t);
	bm->prb0 = ~bm->prb1;
	bm->prb1 |= 1 << t;
	bm->tmsk = (-1 << t) & 0xffff;
	bm->prb0 &= bm->tmsk;
	bm->prb1 &= bm->tmsk;
	bm->ntm = ~bm->tmsk;
	return;
}

static int tek_rdget0(struct tek_STR_RNGDEC *rd, int n, int i)
{
	do {
		while (rd->range < (UINT32) (1 << 24)) {
			rd->range <<= 8;
			rd->code = rd->code << 8 | *rd->p++;
		}
		rd->range >>= 1;
		i += i;
		if (rd->code >= rd->range) {
			rd->code -= rd->range;
			i |= 1;
		}
	} while (--n);
	return ~i;
}

static int tek_rdget1(struct tek_STR_RNGDEC *rd, tek_TPRB *prob0, int n, int j, struct tek_STR_BITMODEL *bm)
{
	UINT32 p, i, *prob, nm = n >> 4;
	n &= 0x0f;
	prob0 -= j;
	do {
		p = *(prob = prob0 + j);
		if (bm->lt > 0) {
			if (--bm->lt == 0) {
				/* 寿命切れ */
				if (tek_rdget1(rd, &rd->probs.fchglt, 0x71, 0, &rd->bm[3]) == 0) {
					/* 寿命変更はまだサポートしてない */
err:
					longjmp(rd->errjmp, 1);
				}
				i = bm - rd->bm;
				if ((bm->s = tek_rdget1(rd, &rd->probs.fchgprm[i * 2 + bm->s], 0x71, 0, &rd->bm[1])) == 0) {
					i = tek_rdget1(rd, rd->probs.tbmt, 0x74, 1, &rd->bm[2]) & 15;
					if (i == 15)
						goto err;
					tek_setbm5(bm, i, ((tek_rdget1(rd, rd->probs.tbmm, 0x74, 1, &rd->bm[2]) - 1) & 15) + 1);
				}
				bm->lt = bm->lt0;
			}
			if (p < bm->prb0) {
				p = bm->prb0;
				goto fixprob;
			}
			if (p > bm->prb1) {
				p = bm->prb1;
				goto fixprob;
			}
			if (p & bm->ntm) {
				p &= bm->tmsk;
	fixprob:
				*prob = p;
			}
		}

		while (rd->range < (UINT32) (1 << 24)) {
			rd->range <<= 8;
			rd->code = rd->code << 8 | *rd->p++;
		}
		j += j;
		i = ((unsigned long long) (rd->range & rd->rmsk) * p) >> 16;
		if (rd->code < i) {
			j |= 1;
			rd->range = i;
			*prob += ((0x10000 - p) >> bm->m) & bm->tmsk;
		} else {
			rd->range -= i;
			rd->code -= i;
			*prob -= (p >> bm->m) & bm->tmsk;
		}
		--n;
		if ((n & nm) == 0)
			bm++;
	} while (n);
	return j;
}

static UINT32 tek_revbit(UINT32 data, int len)
{
	UINT32 rev = 0;
	do {
		rev += rev + (data & 1);
		data >>= 1;
	} while (--len);
	return rev;
}

static int tek_getlen5(struct tek_STR_RNGDEC *rd, int m, int s_pos, int stk)
{
	int i;
	if (tek_rdget1(rd, &rd->probs.lensel[m][0], 0x71, 0, rd->ptbm[3]) ^ stk) /* low */
		i = tek_rdget1(rd, rd->probs.pb[s_pos].lenlow[m], 0x73, 1, rd->ptbm[4]) & 7;
	else if (tek_rdget1(rd, &rd->probs.lensel[m][1], 0x71, 0, rd->ptbm[3]) ^ stk) /* mid */
		i = tek_rdget1(rd, rd->probs.pb[s_pos].lenmid[m], 0x73, 1, rd->ptbm[5]);
	else {
		/* high */
		i = tek_rdget1(rd, rd->probs.lenhigh[m], 0x78, 1, rd->ptbm[6]) - (256 + 256 - 8);
		if (i > 0) {
			if (i < 6 && stk == 0)
				i = tek_rdget1(rd, &rd->probs.lenext[(1 << i) - 2], i | 0x70, 1, rd->ptbm[7]) - 1;
			else
				i = tek_rdget0(rd, i, ~1) - 1;
			i = tek_rdget0(rd, i, ~1) - 1;
		}
		i += 256 - 8 + 16;
	}
	return i;
}

static int tek_decmain5(int *work, UCHAR *src, int osiz, UCHAR *q, int lc, int pb, int lp, int flags)
{
	static int state_table[] = { 0, 0, 0, 0, 1, 2, 3, 4, 5, 6, 4, 5 };
	int i, j, k, pmch, rep[4], s, pos, m_pos = (1 << pb) - 1, m_lp = (1 << lp) - 1;
	int stk = (flags == -1), lcr = 8 - lc, s_pos, lit0cntmsk = 0x78;
	UINT32 *lit1;
	struct tek_STR_RNGDEC *rd = (struct tek_STR_RNGDEC *) work;
	struct tek_STR_PRB *prb = &rd->probs;

	rd->p = &src[4];
	rd->range |= -1;
	rd->code = src[0] << 24 | src[1] << 16 | src[2] << 8 | src[3];
	for (i = 0; i < 4; i++)
		rep[i] = ~i;
	if (setjmp(rd->errjmp))
		goto err;
	for (i = sizeof (struct tek_STR_PRB) / sizeof (tek_TPRB) + (0x300 << (lc + lp)) - 2; i >= 0; i--)
		((tek_TPRB *) prb)[i] = 1 << 15;
	for (i = 0; i < 32; i++) {
		rd->bm[i].lt = (i >= 4); /* 0..3は寿命なし */
		rd->bm[i].lt0 = (i < 24) ? 16 * 1024 : 8 * 1024;
		rd->bm[i].s &= 0;
		rd->bm[i].t = rd->bm[i].m = 5;
	}
	lit1 = prb->lit + ((256 << (lc + lp)) - 2);
	if (stk) {
		rd->rmsk = -1 << 11;
		for (i = 0; i < 32; i++)
			rd->bm[i].lt = 0; /* 全て寿命なし */
		for (i = 0; i < 14; i++)
			rd->ptbm[i] = &rd->bm[0];
	} else {
		UCHAR pt[14];
		static UCHAR pt1[14] = {
			 8,  8,  8,  8,  8,  8,  8,  8,
			 8,  8, 18, 18, 18,  8
		};
		static UCHAR pt2[14] = {
			 8,  8, 10, 11, 12, 12, 14, 15,
			16, 16, 18, 18, 20, 21
		};
		/*
			 0- 7:mch, mch, lit1, lensel, lenlow, lenmid, lenhigh, lenext
			 8-15:pslot, pslot, sdis, sdis, align, rep-repg2
		*/
		rd->rmsk |= -1;
		rd->bm[1].t = 5; rd->bm[1].m = 3; /* for fchgprm */
		rd->bm[2].t = 9; rd->bm[2].m = 2; /* for tbmt, tbmm */
		if (flags & 0x40) { /* lt-flag */
			rd->bm[3].t = 0; rd->bm[3].m = 1;
			prb->fchglt = 0xffff;
		}
		rd->bm[22].t = 0; rd->bm[22].m = 1;
		prb->repg3 = 0xffff;
		if (flags == -2) { /* z1 */
			rd->bm[22].lt = 0; /* repg3のltを0に */
			for (i = 0; i < 14; i++)
				pt[i] = pt1[i];
		} else {
			for (i = 0; i < 14; i++)
				pt[i] = pt2[i];
			lit0cntmsk = (7 >> (flags & 3)) << 4 | 8;
			pt[ 1] =  8 + ((flags & 0x04) != 0); /* mch */
			pt[ 5] = 12 + ((flags & 0x08) != 0); /* llm */
			pt[ 9] = 16 + ((flags & 0x10) != 0); /* pst */
			pt[11] = 18 + ((flags & 0x20) != 0); /* sds */
		}
		for (i = 0; i < 14; i++)
			rd->ptbm[i] = &rd->bm[pt[i]];
	}
	for (i = 0; i < 32; i++)
		tek_setbm5(&rd->bm[i], rd->bm[i].t, rd->bm[i].m);

	if ((tek_rdget1(rd, &prb->pb[0].st[0].mch, 0x71, 0, rd->ptbm[0]) ^ stk) == 0)
		goto err;
	*q++ = tek_rdget1(rd, prb->lit, lit0cntmsk, 1, &rd->bm[24]) & 0xff;
	pmch &= 0; s &= 0; pos = 1;
	while (pos < osiz) {
		s_pos = pos & m_pos;
		if (tek_rdget1(rd, &prb->pb[s_pos].st[s].mch, 0x71, 0, rd->ptbm[s > 0]) ^ stk) { /* 非lz */
			i = (q[-1] >> lcr | (pos & m_lp) << lc) << 8;
			s = state_table[s];
			if (pmch == 0)
				*q = tek_rdget1(rd, &prb->lit[i], lit0cntmsk, 1, &rd->bm[24]) & 0xff;
			else {
				struct tek_STR_BITMODEL *bm = &rd->bm[24];
				j = 1; /* lit1は最初から2を減じてある */
				k = 8;
				pmch = q[rep[0]];
				do {
					j += j + tek_rdget1(rd, &lit1[(i + j) << 1 | pmch >> 7], 0x71, 0, rd->ptbm[2]);
					k--;
					if ((k & (lit0cntmsk >> 4)) == 0)
						bm++;
					if ((((pmch >> 7) ^ j) & 1) != 0 && k != 0) {
						j = tek_rdget1(rd, &prb->lit[i + j - 1], k | (lit0cntmsk & 0x70), j, bm);
						break;
					}
					pmch <<= 1;
				} while (k);
				*q = j & 0xff;
				pmch &= 0;
			}
			pos++;
			q++;
		} else { /* lz */
			pmch |= 1;
			if (tek_rdget1(rd, &prb->st[s].rep, 0x71, 0, rd->ptbm[13]) ^ stk) { /* len/dis */
				rep[3] = rep[2];
				rep[2] = rep[1];
				rep[1] = rep[0];
				j = i = tek_getlen5(rd, 0, s_pos, stk);
				s = s < 7 ? 7 : 10;
				if (j >= 4)
					j = 3;
				rep[0] = j = tek_rdget1(rd, prb->pslot[j], 0x76, 1, rd->ptbm[8 + (j == 3)]) & 0x3f;
				if (j >= 4) {
					k = (j >> 1) - 1; /* k = [1, 30] */
					rep[0] = (2 | (j & 1)) << k;
					if (j < 14) /* k < 6 */
						rep[0] |= tek_revbit(tek_rdget1(rd, &prb->spdis[j & 1][(1 << k) - 2], k | 0x70, 1, rd->ptbm[10 + (k >= 4)]), k);
					else {
						if (stk == 0) {
							if (k -= 6)
								rep[0] |= tek_rdget0(rd, k, ~0) << 6;
							rep[0] |= tek_revbit(tek_rdget1(rd, prb->algn, 0x76, 1, rd->ptbm[12]), 6);
						} else {
							rep[0] |= tek_rdget0(rd, k - 4, ~0) << 4;
							rep[0] |= tek_revbit(tek_rdget1(rd, prb->algn, 0x74, 1, rd->ptbm[12]), 4);
						}
					}
				}
				rep[0] = ~rep[0];
			} else { /* repeat-dis */
				if (tek_rdget1(rd, &prb->st[s].repg0, 0x71, 0, rd->ptbm[13]) ^ stk) { /* rep0 */
					i |= -1;
					if (tek_rdget1(rd, &prb->pb[s_pos].st[s].rep0l1, 0x71, 0, rd->ptbm[13]) == 0) {
						s = s < 7 ? 9 : 11;
						goto skip;
					}
				} else {
					if (tek_rdget1(rd, &prb->st[s].repg1, 0x71, 0, rd->ptbm[13]) ^ stk) /* rep1 */
						i = rep[1];
					else {
						if (tek_rdget1(rd, &prb->st[s].repg2, 0x71, 0, rd->ptbm[13]) ^ stk) /* rep2 */
							i = rep[2];
						else {
							if (stk == 0) {
								if  (tek_rdget1(rd, &prb->repg3, 0x71, 0, &rd->bm[22]) == 0)
									goto err;
							}
							i = rep[3]; /* rep3 */
							rep[3] = rep[2];
						}
						rep[2] = rep[1];
					}
					rep[1] = rep[0];
					rep[0] = i;
				}
				i = tek_getlen5(rd, 1, s_pos, stk);
				s = s < 7 ? 8 : 11;
			}
skip:
			i += 2;
			if (pos + rep[0] < 0)
				goto err;
			if (pos + i > osiz)
				i = osiz - pos;
			pos += i;
			do {
				*q = q[rep[0]];
				q++;
			} while (--i);
		}
	}
	return 0;
err:
	return 1;
}

static int tek_decode5(int siz, UCHAR *p, UCHAR *q)
{
	UCHAR *p1 = p + siz;
	int dsiz, hed, bsiz, st = 0;
	p += 16;
	if ((dsiz = tek_getnum_s7s(&p)) > 0) {
		hed = tek_getnum_s7s(&p);
		if ((hed & 1) == 0)
			st = tek_lzrestore_tek5(p1 - p + 1, p - 1, dsiz, q);
		else {
			bsiz = 1 << (((hed >> 1) & 0x0f) + 8);
			if (hed & 0x20)
				return 1;
			if (bsiz == 256)
				st = tek_lzrestore_tek5(p1 - p, p, dsiz, q);
			else {
				if (dsiz > bsiz)
					return 1;
				if (hed & 0x40)
					tek_getnum_s7s(&p); /* オプション情報へのポインタを読み飛ばす */
				st = tek_lzrestore_tek5(p1 - p, p, dsiz, q);
			}
		}
	}
	return st;
}
```

autodec_.c
```c
#include <stdio.h>		/* NULL */
#include <stdlib.h>		/* malloc, free */
#include <setjmp.h>

typedef unsigned char UCHAR;
typedef unsigned int UINT32;
typedef UINT32 tek_TPRB;

static int tek_decode1(int siz, UCHAR *p, UCHAR *q);
static int tek_decode2(int siz, UCHAR *p, UCHAR *q);
static int tek_decode5(int siz, UCHAR *p, UCHAR *q);

static unsigned int tek_getnum_s7s(UCHAR **pp)
/* これは必ずbig-endian */
/* 下駄がないので中身をいじりやすい */
{
	unsigned int s = 0;
	UCHAR *p = *pp;
	do {
		s = s << 7 | *p++;
	} while ((s & 1) == 0);
	s >>= 1;
	*pp = p;
	return s;
}

int autodecomp(int siz0, UCHAR *p0, int siz)
{
	unsigned char *b = p0, *c, *c0;
	int s, i, e = 0;
	if ((*(int *) &b[0x08] == 0x5341534f) && (*(int *) &b[0x0c] == 0x504d434b)) {
		if (*(int *) &b[0x04] == 0x00000001) {
			unsigned int t = *(int *) &b[0x00];
			e |= 1;
			if (0xffffff83 <= t && t <= 0xffffff89) {
				c = &b[0x10];
				s = tek_getnum_s7s(&c);
				if (s + siz - 0x10 <= siz0) {
					c0 = c = b + siz0 - siz;
					for (i = siz - 1; i >= 0x10; i--)
						c[i] = b[i];
					c += 0x10;
					tek_getnum_s7s(&c);
					if (t == 0xffffff83)
						e = tek_decode1(siz, c0, b);
					if (t == 0xffffff85)
						e = tek_decode2(siz, c0, b);
					if (t == 0xffffff89)
						e = tek_decode5(siz, c0, b);
					siz = s;
				}
			}
		}
	}
	if (e)
		siz |= -1;
	return siz;
}

static int tek_lzrestore_stk1(int srcsiz, UCHAR *src, int outsiz, UCHAR *q)
{
	int by, lz, cp, ds;
	UCHAR *q1 = q + outsiz, *s7ptr = src, *q0 = q;
	do {
		if ((by = (lz = *s7ptr++) & 0x0f) == 0)
			by = tek_getnum_s7s(&s7ptr);
		if ((lz >>= 4) == 0)
			lz = tek_getnum_s7s(&s7ptr);
		do {
			*q++ = *s7ptr++;
		} while (--by);
		if (q >= q1)
			break;
		do {
			ds = (cp = *s7ptr++) & 0x0f;
			if ((ds & 1) == 0) {
				do {
					ds = ds << 7 | *s7ptr++;
				} while ((ds & 1) == 0);
			}
			ds = ~(ds >> 1);
			if ((cp >>= 4) == 0) {
				do {
					cp = cp << 7 | *s7ptr++;
				} while ((cp & 1) == 0);
				cp >>= 1;
			} /* 0がこないことをあてにする */
			cp++;
			if (q + ds < q0)
				goto err;
			if (q + cp > q1)
				cp = q1 - q;
			do {
				*q = *(q + ds);
				q++;
			} while (--cp);
		} while (--lz);
	} while (q < q1);
	return 0;
err:
	return 1;
}

static int tek_decode1(int siz, UCHAR *p, UCHAR *q)
{
	int dsiz, hed, bsiz;
	UCHAR *p1 = p + siz;
	p += 16;
	if ((dsiz = tek_getnum_s7s(&p)) > 0) {
		hed = tek_getnum_s7s(&p);
		bsiz = 1 << (((hed >> 1) & 0x0f) + 8);
		if (dsiz > bsiz || (hed & 0x21) != 0x01)
			return 1;
		if (hed & 0x40)
			tek_getnum_s7s(&p); /* オプション情報へのポインタを読み飛ばす */
		if (tek_getnum_s7s(&p) != 0)
			return 1; /* 補助バッファ使用 */
		return tek_lzrestore_stk1(p1 - p, p, dsiz, q);
	}
	return 0;
}

static unsigned int tek_getnum_s7(UCHAR **pp)
/* これは必ずbig-endian */
{
	unsigned int s = 0, b = 0, a = 1;
	UCHAR *p = *pp;
	for (;;) {
		s = s << 7 | *p++;
		if (s & 1)
			break;
		a <<= 7;
		b += a;
	}
	s >>= 1;
	*pp = p;
	return s + b;
}

static int tek_lzrestore_stk2(int srcsiz, UCHAR *src, int outsiz, UCHAR *q)
{
	int cp, ds, repdis[4], i, j;
	UCHAR *q1 = q + outsiz, *s7ptr = src, *q0 = q, bylz, cbylz;
	for (j = 0; j < 4; j++)
		repdis[j] = -1 - j;
	bylz = cbylz = 0;
	if (outsiz) {
		if (tek_getnum_s7s(&s7ptr))
			return 1;
		do {
			/* byフェーズ */
			j = 0;
			do {
				j++;
				if (j >= 17) {
					j += tek_getnum_s7s(&s7ptr);
					break;
				}
				if (cbylz == 0) {
					cbylz = 8;
					bylz = *s7ptr++;
				}
				cbylz--;
				i = bylz & 1;
				bylz >>= 1;
			} while (i == 0);
			do {
				*q++ = *s7ptr++;
			} while (--j);
			if (q >= q1)
				break;

			/* lzフェーズ */
			j = 0;
			do {
				j++;
				if (j >= 17) {
					j += tek_getnum_s7s(&s7ptr);
					break;
				}
				if (cbylz == 0) {
					cbylz = 8;
					bylz = *s7ptr++;
				}
				cbylz--;
				i = bylz & 1;
				bylz >>= 1;
			} while (i == 0);
			do {
				i = *s7ptr++;
				cp = i >> 4;
				i &= 0x0f;
				if ((i & 1) == 0)
					i |= (tek_getnum_s7(&s7ptr) + 1) << 4;
				i >>= 1;
				ds = ~(i - 6);
				if (i < 4)
					ds = repdis[i];
				if (i == 4)
					ds = repdis[0] - tek_getnum_s7(&s7ptr) - 1;
				if (i == 5)
					ds = repdis[0] + tek_getnum_s7(&s7ptr) + 1;
				if (cp == 0)
					cp = tek_getnum_s7(&s7ptr) + 16;
				cp++;
				if (i > 0) {
					if (i > 1) {
						if (i > 2)
							repdis[3] = repdis[2];
						repdis[2] = repdis[1];
					}
					repdis[1] = repdis[0];
					repdis[0] = ds;
				}
				if (q + ds < q0)
					goto err;
				if (q + cp > q1)
					cp = q1 - q;
				do {
					*q = *(q + ds);
					q++;
				} while (--cp);
			} while (--j);
		} while (q < q1);
	}
	return 0;
err:
	return 1;
}

static int tek_decode2(int siz, UCHAR *p, UCHAR *q)
{
	UCHAR *p1 = p + siz;
	int dsiz, hed, bsiz, st = 0;
	p += 16;
	if ((dsiz = tek_getnum_s7s(&p)) > 0) {
		hed = tek_getnum_s7s(&p);
		bsiz = 1 << (((hed >> 1) & 0x0f) + 8);
		if (dsiz > bsiz || (hed & 0x21) != 0x01)
			return 1;
		if (hed & 0x40)
			tek_getnum_s7s(&p); /* オプション情報へのポインタを読み飛ばす */
		st = tek_lzrestore_stk2(p1 - p, p, dsiz, q);
	}
	return st;
}

static int tek_decmain5(int *work, UCHAR *src, int osiz, UCHAR *q, int lc, int pb, int lp, int flags);

static int tek_lzrestore_tek5(int srcsiz, UCHAR *src, int outsiz, UCHAR *outbuf)
{
	int wrksiz, lc, lp, pb, flags, *work, prop0, fl;

	if ((fl = (prop0 = *src) & 0x0f) == 0x01) /* 0001 */
		flags |= -1;
	else if (fl == 0x05)
		flags = -2;
	else if (fl == 0x09)
		flags &= 0;
	else
		return 1;
	src++;
	prop0 >>= 4;
	if (prop0 == 0)
		prop0 = *src++;
	else {
		static UCHAR prop0_table[] = { 0x5d, 0x00 }, prop1_table[] = { 0x00 };
		if (flags == -1) {
			if (prop0 >= 3)
				return 1;
			prop0 = prop0_table[prop0 - 1];
		} else {
			if (prop0 >= 2)
				return 1;
			prop0 = prop1_table[prop0 - 1];
		}
	}
	lp = prop0 / (9 * 5);
	prop0 %= 9 * 5;
	pb = prop0 / 9;
	lc = prop0 % 9;
	if (flags == 0) /* tek5:z2 */
		flags = *src++;
	if (flags == -1) { /* stk5 */
		wrksiz = lp;
		lp = pb;
		pb = wrksiz;
	}
	wrksiz = 0x180 * sizeof (UINT32) + (0x840 + (0x300 << (lc + lp))) * sizeof (tek_TPRB); /* 最低15KB, lc+lp=3なら、36KB */
	work = malloc(wrksiz);
	if (work == NULL)
		return -1;
	flags = tek_decmain5(work, src, outsiz, outbuf, lc, pb, lp, flags);
	free(work);
	return flags;
}

struct tek_STR_BITMODEL {
	UCHAR t, m, s, dmy;
	UINT32 prb0, prb1, tmsk, ntm, lt, lt0, dmy4;
};

struct tek_STR_PRB {
	struct tek_STR_PRB_PB {
		struct tek_STR_PRB_PBST {
			tek_TPRB mch, rep0l1;
		} st[12];
		tek_TPRB lenlow[2][8], lenmid[2][8];
	} pb[16];
	struct tek_STR_PRB_ST {
		tek_TPRB rep, repg0, repg1, repg2;
	} st[12];
	tek_TPRB lensel[2][2], lenhigh[2][256], pslot[4][64], algn[64];
	tek_TPRB spdis[2][2+4+8+16+32], lenext[2+4+8+16+32];
	tek_TPRB repg3, fchgprm[2 * 32], tbmt[16], tbmm[16], fchglt;
	tek_TPRB lit[1];
};

struct tek_STR_RNGDEC {
	UCHAR *p;
	UINT32 range, code, rmsk;
	jmp_buf errjmp;
	struct tek_STR_BITMODEL bm[32], *ptbm[16];
	struct tek_STR_PRB probs;
};

static void tek_setbm5(struct tek_STR_BITMODEL *bm, int t, int m)
{
	bm->t = t;
	bm->m = m;
	bm->prb1 = -1 << (m + t);
	bm->prb0 = ~bm->prb1;
	bm->prb1 |= 1 << t;
	bm->tmsk = (-1 << t) & 0xffff;
	bm->prb0 &= bm->tmsk;
	bm->prb1 &= bm->tmsk;
	bm->ntm = ~bm->tmsk;
	return;
}

static int tek_rdget0(struct tek_STR_RNGDEC *rd, int n, int i)
{
	do {
		while (rd->range < (UINT32) (1 << 24)) {
			rd->range <<= 8;
			rd->code = rd->code << 8 | *rd->p++;
		}
		rd->range >>= 1;
		i += i;
		if (rd->code >= rd->range) {
			rd->code -= rd->range;
			i |= 1;
		}
	} while (--n);
	return ~i;
}

static int tek_rdget1(struct tek_STR_RNGDEC *rd, tek_TPRB *prob0, int n, int j, struct tek_STR_BITMODEL *bm)
{
	UINT32 p, i, *prob, nm = n >> 4;
	n &= 0x0f;
	prob0 -= j;
	do {
		p = *(prob = prob0 + j);
		if (bm->lt > 0) {
			if (--bm->lt == 0) {
				/* 寿命切れ */
				if (tek_rdget1(rd, &rd->probs.fchglt, 0x71, 0, &rd->bm[3]) == 0) {
					/* 寿命変更はまだサポートしてない */
err:
					longjmp(rd->errjmp, 1);
				}
				i = bm - rd->bm;
				if ((bm->s = tek_rdget1(rd, &rd->probs.fchgprm[i * 2 + bm->s], 0x71, 0, &rd->bm[1])) == 0) {
					i = tek_rdget1(rd, rd->probs.tbmt, 0x74, 1, &rd->bm[2]) & 15;
					if (i == 15)
						goto err;
					tek_setbm5(bm, i, ((tek_rdget1(rd, rd->probs.tbmm, 0x74, 1, &rd->bm[2]) - 1) & 15) + 1);
				}
				bm->lt = bm->lt0;
			}
			if (p < bm->prb0) {
				p = bm->prb0;
				goto fixprob;
			}
			if (p > bm->prb1) {
				p = bm->prb1;
				goto fixprob;
			}
			if (p & bm->ntm) {
				p &= bm->tmsk;
	fixprob:
				*prob = p;
			}
		}

		while (rd->range < (UINT32) (1 << 24)) {
			rd->range <<= 8;
			rd->code = rd->code << 8 | *rd->p++;
		}
		j += j;
		i = ((unsigned long long) (rd->range & rd->rmsk) * p) >> 16;
		if (rd->code < i) {
			j |= 1;
			rd->range = i;
			*prob += ((0x10000 - p) >> bm->m) & bm->tmsk;
		} else {
			rd->range -= i;
			rd->code -= i;
			*prob -= (p >> bm->m) & bm->tmsk;
		}
		--n;
		if ((n & nm) == 0)
			bm++;
	} while (n);
	return j;
}

static UINT32 tek_revbit(UINT32 data, int len)
{
	UINT32 rev = 0;
	do {
		rev += rev + (data & 1);
		data >>= 1;
	} while (--len);
	return rev;
}

static int tek_getlen5(struct tek_STR_RNGDEC *rd, int m, int s_pos, int stk)
{
	int i;
	if (tek_rdget1(rd, &rd->probs.lensel[m][0], 0x71, 0, rd->ptbm[3]) ^ stk) /* low */
		i = tek_rdget1(rd, rd->probs.pb[s_pos].lenlow[m], 0x73, 1, rd->ptbm[4]) & 7;
	else if (tek_rdget1(rd, &rd->probs.lensel[m][1], 0x71, 0, rd->ptbm[3]) ^ stk) /* mid */
		i = tek_rdget1(rd, rd->probs.pb[s_pos].lenmid[m], 0x73, 1, rd->ptbm[5]);
	else {
		/* high */
		i = tek_rdget1(rd, rd->probs.lenhigh[m], 0x78, 1, rd->ptbm[6]) - (256 + 256 - 8);
		if (i > 0) {
			if (i < 6 && stk == 0)
				i = tek_rdget1(rd, &rd->probs.lenext[(1 << i) - 2], i | 0x70, 1, rd->ptbm[7]) - 1;
			else
				i = tek_rdget0(rd, i, ~1) - 1;
			i = tek_rdget0(rd, i, ~1) - 1;
		}
		i += 256 - 8 + 16;
	}
	return i;
}

static int tek_decmain5(int *work, UCHAR *src, int osiz, UCHAR *q, int lc, int pb, int lp, int flags)
{
	static int state_table[] = { 0, 0, 0, 0, 1, 2, 3, 4, 5, 6, 4, 5 };
	int i, j, k, pmch, rep[4], s, pos, m_pos = (1 << pb) - 1, m_lp = (1 << lp) - 1;
	int stk = (flags == -1), lcr = 8 - lc, s_pos, lit0cntmsk = 0x78;
	UINT32 *lit1;
	struct tek_STR_RNGDEC *rd = (struct tek_STR_RNGDEC *) work;
	struct tek_STR_PRB *prb = &rd->probs;

	rd->p = &src[4];
	rd->range |= -1;
	rd->code = src[0] << 24 | src[1] << 16 | src[2] << 8 | src[3];
	for (i = 0; i < 4; i++)
		rep[i] = ~i;
	if (setjmp(rd->errjmp))
		goto err;
	for (i = sizeof (struct tek_STR_PRB) / sizeof (tek_TPRB) + (0x300 << (lc + lp)) - 2; i >= 0; i--)
		((tek_TPRB *) prb)[i] = 1 << 15;
	for (i = 0; i < 32; i++) {
		rd->bm[i].lt = (i >= 4); /* 0..3は寿命なし */
		rd->bm[i].lt0 = (i < 24) ? 16 * 1024 : 8 * 1024;
		rd->bm[i].s &= 0;
		rd->bm[i].t = rd->bm[i].m = 5;
	}
	lit1 = prb->lit + ((256 << (lc + lp)) - 2);
	if (stk) {
		rd->rmsk = -1 << 11;
		for (i = 0; i < 32; i++)
			rd->bm[i].lt = 0; /* 全て寿命なし */
		for (i = 0; i < 14; i++)
			rd->ptbm[i] = &rd->bm[0];
	} else {
		UCHAR pt[14];
		static UCHAR pt1[14] = {
			 8,  8,  8,  8,  8,  8,  8,  8,
			 8,  8, 18, 18, 18,  8
		};
		static UCHAR pt2[14] = {
			 8,  8, 10, 11, 12, 12, 14, 15,
			16, 16, 18, 18, 20, 21
		};
		/*
			 0- 7:mch, mch, lit1, lensel, lenlow, lenmid, lenhigh, lenext
			 8-15:pslot, pslot, sdis, sdis, align, rep-repg2
		*/
		rd->rmsk |= -1;
		rd->bm[1].t = 5; rd->bm[1].m = 3; /* for fchgprm */
		rd->bm[2].t = 9; rd->bm[2].m = 2; /* for tbmt, tbmm */
		if (flags & 0x40) { /* lt-flag */
			rd->bm[3].t = 0; rd->bm[3].m = 1;
			prb->fchglt = 0xffff;
		}
		rd->bm[22].t = 0; rd->bm[22].m = 1;
		prb->repg3 = 0xffff;
		if (flags == -2) { /* z1 */
			rd->bm[22].lt = 0; /* repg3のltを0に */
			for (i = 0; i < 14; i++)
				pt[i] = pt1[i];
		} else {
			for (i = 0; i < 14; i++)
				pt[i] = pt2[i];
			lit0cntmsk = (7 >> (flags & 3)) << 4 | 8;
			pt[ 1] =  8 + ((flags & 0x04) != 0); /* mch */
			pt[ 5] = 12 + ((flags & 0x08) != 0); /* llm */
			pt[ 9] = 16 + ((flags & 0x10) != 0); /* pst */
			pt[11] = 18 + ((flags & 0x20) != 0); /* sds */
		}
		for (i = 0; i < 14; i++)
			rd->ptbm[i] = &rd->bm[pt[i]];
	}
	for (i = 0; i < 32; i++)
		tek_setbm5(&rd->bm[i], rd->bm[i].t, rd->bm[i].m);

	if ((tek_rdget1(rd, &prb->pb[0].st[0].mch, 0x71, 0, rd->ptbm[0]) ^ stk) == 0)
		goto err;
	*q++ = tek_rdget1(rd, prb->lit, lit0cntmsk, 1, &rd->bm[24]) & 0xff;
	pmch &= 0; s &= 0; pos = 1;
	while (pos < osiz) {
		s_pos = pos & m_pos;
		if (tek_rdget1(rd, &prb->pb[s_pos].st[s].mch, 0x71, 0, rd->ptbm[s > 0]) ^ stk) { /* 非lz */
			i = (q[-1] >> lcr | (pos & m_lp) << lc) << 8;
			s = state_table[s];
			if (pmch == 0)
				*q = tek_rdget1(rd, &prb->lit[i], lit0cntmsk, 1, &rd->bm[24]) & 0xff;
			else {
				struct tek_STR_BITMODEL *bm = &rd->bm[24];
				j = 1; /* lit1は最初から2を減じてある */
				k = 8;
				pmch = q[rep[0]];
				do {
					j += j + tek_rdget1(rd, &lit1[(i + j) << 1 | pmch >> 7], 0x71, 0, rd->ptbm[2]);
					k--;
					if ((k & (lit0cntmsk >> 4)) == 0)
						bm++;
					if ((((pmch >> 7) ^ j) & 1) != 0 && k != 0) {
						j = tek_rdget1(rd, &prb->lit[i + j - 1], k | (lit0cntmsk & 0x70), j, bm);
						break;
					}
					pmch <<= 1;
				} while (k);
				*q = j & 0xff;
				pmch &= 0;
			}
			pos++;
			q++;
		} else { /* lz */
			pmch |= 1;
			if (tek_rdget1(rd, &prb->st[s].rep, 0x71, 0, rd->ptbm[13]) ^ stk) { /* len/dis */
				rep[3] = rep[2];
				rep[2] = rep[1];
				rep[1] = rep[0];
				j = i = tek_getlen5(rd, 0, s_pos, stk);
				s = s < 7 ? 7 : 10;
				if (j >= 4)
					j = 3;
				rep[0] = j = tek_rdget1(rd, prb->pslot[j], 0x76, 1, rd->ptbm[8 + (j == 3)]) & 0x3f;
				if (j >= 4) {
					k = (j >> 1) - 1; /* k = [1, 30] */
					rep[0] = (2 | (j & 1)) << k;
					if (j < 14) /* k < 6 */
						rep[0] |= tek_revbit(tek_rdget1(rd, &prb->spdis[j & 1][(1 << k) - 2], k | 0x70, 1, rd->ptbm[10 + (k >= 4)]), k);
					else {
						if (stk == 0) {
							if (k -= 6)
								rep[0] |= tek_rdget0(rd, k, ~0) << 6;
							rep[0] |= tek_revbit(tek_rdget1(rd, prb->algn, 0x76, 1, rd->ptbm[12]), 6);
						} else {
							rep[0] |= tek_rdget0(rd, k - 4, ~0) << 4;
							rep[0] |= tek_revbit(tek_rdget1(rd, prb->algn, 0x74, 1, rd->ptbm[12]), 4);
						}
					}
				}
				rep[0] = ~rep[0];
			} else { /* repeat-dis */
				if (tek_rdget1(rd, &prb->st[s].repg0, 0x71, 0, rd->ptbm[13]) ^ stk) { /* rep0 */
					i |= -1;
					if (tek_rdget1(rd, &prb->pb[s_pos].st[s].rep0l1, 0x71, 0, rd->ptbm[13]) == 0) {
						s = s < 7 ? 9 : 11;
						goto skip;
					}
				} else {
					if (tek_rdget1(rd, &prb->st[s].repg1, 0x71, 0, rd->ptbm[13]) ^ stk) /* rep1 */
						i = rep[1];
					else {
						if (tek_rdget1(rd, &prb->st[s].repg2, 0x71, 0, rd->ptbm[13]) ^ stk) /* rep2 */
							i = rep[2];
						else {
							if (stk == 0) {
								if  (tek_rdget1(rd, &prb->repg3, 0x71, 0, &rd->bm[22]) == 0)
									goto err;
							}
							i = rep[3]; /* rep3 */
							rep[3] = rep[2];
						}
						rep[2] = rep[1];
					}
					rep[1] = rep[0];
					rep[0] = i;
				}
				i = tek_getlen5(rd, 1, s_pos, stk);
				s = s < 7 ? 8 : 11;
			}
skip:
			i += 2;
			if (pos + rep[0] < 0)
				goto err;
			if (pos + i > osiz)
				i = osiz - pos;
			pos += i;
			do {
				*q = q[rep[0]];
				q++;
			} while (--i);
		}
	}
	return 0;
err:
	return 1;
}

int tek_decode5(int siz, UCHAR *p, UCHAR *q)
{
	UCHAR *p1 = p + siz;
	int dsiz, hed, bsiz, st = 0;
	p += 16;
	if ((dsiz = tek_getnum_s7s(&p)) > 0) {
		hed = tek_getnum_s7s(&p);
		if ((hed & 1) == 0)
			st = tek_lzrestore_tek5(p1 - p + 1, p - 1, dsiz, q);
		else {
			bsiz = 1 << (((hed >> 1) & 0x0f) + 8);
			if (hed & 0x20)
				return 1;
			if (bsiz == 256)
				st = tek_lzrestore_tek5(p1 - p, p, dsiz, q);
			else {
				if (dsiz > bsiz)
					return 1;
				if (hed & 0x40)
					tek_getnum_s7s(&p); /* オプション情報へのポインタを読み飛ばす */
				st = tek_lzrestore_tek5(p1 - p, p, dsiz, q);
			}
		}
	}
	return st;
}
```

haribote/Makefile
```Makefile
OBJS_BOOTPACK = bootpack.obj naskfunc.obj hankaku.obj graphic.obj dsctbl.obj \
				int.obj fifo.obj keybord.obj mouse.obj memory.obj sheet.obj timer.obj \
				mtask.obj window.obj console.obj file.obj tek.obj
```

file.c
```c
struct FILEINFO *file_search(char *name, struct FILEINFO *finfo, int max)
{
	int i, j;
	char s[12];
	for (j = 0; j < 11; j++) {
		s[j] = ' ';
	}
	j = 0;
	for (i = 0; name[i] != 0; i++) {
		if (j >= 11) { return 0; /* 見つからなかった */ }
		if (name[i] == '.' && j <= 8) {
			j = 8;
		} else {
			s[j] = name[i];
			if ('a' <= s[j] && s[j] <= 'z') {
				/* 小文字は大文字に直す */
				s[j] -= 0x20;
			}
			j++;
		}
	}
	for (i = 0; i < max; ) {
		if (finfo->name[0] == 0x00) {
			break;
		}
		if ((finfo[i].type & 0x18) == 0) {
			for (j = 0; j < 11; j++) {
				if (finfo[i].name[j] != s[j]) {
					goto next;
				}
			}
			return finfo + i; /* ファイルが見つかった */
		}
next:
		i++;
	}
	return 0; /* 見つからなかった */
}

char *file_loadfile2(int clustno, int *psize, int *fat)
{
	int size = *psize, size2;
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	char *buf, *buf2;
	buf = (char *) memman_alloc_4k(memman, size);
	file_loadfile(clustno, size, buf, fat, (char *) (ADR_DISKIMG + 0x003e00));
	if (size >= 17) {
		size2 = tek_getsize(buf);
		if (size2 > 0) {	/* tek圧縮がかかっていた */
			buf2 = (char *) memman_alloc_4k(memman, size2);
			tek_decomp(buf, buf2, size2);
			memman_free_4k(memman, (int) buf, size);
			buf = buf2;
			*psize = size2;
		}
	}
	return buf;
}
```

bootpack.c
```c
finfo = file_search("nihongo.fnt", (struct FILEINFO *) (ADR_DISKIMG + 0x002600), 224);
	if (finfo != 0) {
		i = finfo->size;
		nihongo = file_loadfile2(finfo->clustno, &i, fat);
	} else {
		nihongo = (unsigned char *) memman_alloc_4k(memman, 16 * 256 + 32 * 94 * 47);
		for (i = 0; i < 16 * 256; i++) {
			nihongo[i] = hankaku[i]; /* フォントがなかったので半角部分をコピー */
		}
		for (i = 16 * 256; i < 16 * 256 + 32 * 94 * 47; i++) {
			nihongo[i] = 0xff; /* フォントがなかったので全角部分を0xffで埋め尽くす */
		}
	}
```

bootpack.h
```h
void cons_putchar(struct CONSOLE *cons, int chr, char move);
void cons_newline(struct CONSOLE *cons);
void cons_putstr0(struct CONSOLE *cons, char *s);
void cons_putstr1(struct CONSOLE *cons, char *s, int l);
void cons_runcmd(char *cmdline, struct CONSOLE *cons, int *fat, int memtotal);
void cmd_mem(struct CONSOLE *cons, int memtotal);
void cmd_cls(struct CONSOLE *cons);
void cmd_dir(struct CONSOLE *cons);
void cmd_exit(struct CONSOLE *cons, int *fat);
void cmd_start(struct CONSOLE *cons, char *cmdline, int memtotal);
void cmd_ncst(struct CONSOLE *cons, char *cmdline, int memtotal);
void cmd_langmode(struct CONSOLE *cons, char *cmdline);
int cmd_app(struct CONSOLE *cons, int *fat, char *cmdline);
int *hrb_api(int edi, int esi, int ebp, int esp, int ebx, int edx, int ecx, int eax);
int *inthandler0d(int *esp);
int *inthandler0c(int *esp);
void hrb_api_linewin(struct SHEET *sht, int x0, int y0, int x1, int y1, int col);

/* file.c */
struct FILEINFO {
	unsigned char name[8], ext[3], type;
	char reserve[10];
	unsigned short time, date, clustno;
	unsigned int size;
};
void file_readfat(int *fat, unsigned char *img);
void file_loadfile(int clustno, int size, char *buf, int *fat, char *img);
struct FILEINFO *file_search(char *name, struct FILEINFO *finfo, int max);
char *file_loadfile2(int clustno, int *psize, int *fat);

/* tek.c */
int tek_getsize(unsigned char *p);
int tek_decomp(unsigned char *p, char *q, int size);
```

console.c
```c
int cmd_app(struct CONSOLE *cons, int *fat, char *cmdline)
{
	struct MEMMAN *memman = (struct MEMMAN *) MEMMAN_ADDR;
	struct FILEINFO *finfo;
	char name[18], *p, *q;
	struct TASK *task = task_now();
	int i, segsiz, datsiz, esp, dathrb, appsiz;
	struct SHTCTL *shtctl;
	struct SHEET *sht;

	/* コマンドラインからファイル名を生成 */
	for (i = 0; i < 13; i++) {
		if (cmdline[i] <= ' ') {
			break;
		}
		name[i] = cmdline[i];
	}
	name[i] = 0; /* とりあえずファイル名の後ろを0にする */

	/* ファイルを探す */
	finfo = file_search(name, (struct FILEINFO *) (ADR_DISKIMG + 0x002600), 224);
	if (finfo == 0 && name[i - 1] != '.') {
		/* 見つからなかったので後ろに".HRB"をつけてもう一度探してみる */
		name[i    ] = '.';
		name[i + 1] = 'H';
		name[i + 2] = 'R';
		name[i + 3] = 'B';
		name[i + 4] = 0;
		finfo = file_search(name, (struct FILEINFO *) (ADR_DISKIMG + 0x002600), 224);
	}

	if (finfo != 0) {
		/* ファイルが見つかった場合 */
		appsiz = finfo->size;
		p = file_loadfile2(finfo->clustno, &appsiz, fat);
		if (appsiz >= 36 && strncmp(p + 4, "Hari", 4) == 0 && *p == 0x00) {
			segsiz = *((int *) (p + 0x0000));
			esp    = *((int *) (p + 0x000c));
			datsiz = *((int *) (p + 0x0010));
			dathrb = *((int *) (p + 0x0014));
			q = (char *) memman_alloc_4k(memman, segsiz);
			task->ds_base = (int) q;
			set_segmdesc(task->ldt + 0, appsiz - 1, (int) p, AR_CODE32_ER + 0x60);
			set_segmdesc(task->ldt + 1, segsiz - 1, (int) q, AR_DATA32_RW + 0x60);
			for (i = 0; i < datsiz; i++) {
				q[esp + i] = p[dathrb + i];
			}
			start_app(0x1b, 0 * 8 + 4, esp, 1 * 8 + 4, &(task->tss.esp0));
			shtctl = (struct SHTCTL *) *((int *) 0x0fe4);
			for (i = 0; i < MAX_SHEETS; i++) {
				sht = &(shtctl->sheets0[i]);
				if ((sht->flags & 0x11) == 0x11 && sht->task == task) {
					/* アプリが開きっぱなしにした下じきを発見 */
					sheet_free(sht);	/* 閉じる */
				}
			}
			for (i = 0; i < 8; i++) {	/* クローズしてないファイルをクローズ */
				if (task->fhandle[i].buf != 0) {
					memman_free_4k(memman, (int) task->fhandle[i].buf, task->fhandle[i].size);
					task->fhandle[i].buf = 0;
				}
			}
			timer_cancelall(&task->fifo);
			memman_free_4k(memman, (int) q, segsiz);
			task->langbyte1 = 0;
		} else {
			cons_putstr0(cons, ".hrb file format error.\n");
		}
		memman_free_4k(memman, (int) p, appsiz);
		cons_newline(cons);
		return 1;
	}
	/* ファイルが見つからなかった場合 */
	return 0;
}
```

```c
if (i < 8) {
			finfo = file_search((char *) ebx + ds_base,
					(struct FILEINFO *) (ADR_DISKIMG + 0x002600), 224);
			if (finfo != 0) {
				reg[7] = (int) fh;
				fh->size = finfo->size;
				fh->pos = 0;
				fh->buf = file_loadfile2(finfo->clustno, &fh->size, task->fat);
			}
		}
```

app?make.txt
```Makefile
TOOLPATH = ../../../z_tools/
INCPATH  = ../../../z_tools/haribote/
APILIBPATH   = ../apilib/
HARIBOTEPATH = ../haribote/

MAKE     = $(TOOLPATH)make.exe -r
NASK     = $(TOOLPATH)nask.exe
CC1		 = $(TOOLPATH)cc1.exe -I$(INCPATH) -I../ -Os -Wall -quiet
GAS2NASK = $(TOOLPATH)gas2nask.exe -a
OBJ2BIM  = $(TOOLPATH)obj2bim
MAKEFONT = $(TOOLPATH)makefont.exe
BIN2OBJ  = $(TOOLPATH)bin2obj.exe
BIM2HRB  = $(TOOLPATH)bim2hrb.exe
RULEFILE = ..\haribote.rul
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
GOLIB	 = $(TOOLPATH)golib00.exe
COPY     = cmd.exe /C copy
DEL      = rm -f

# default operation

default :
	$(MAKE) $(APP).hrb

# ファイル生成規則

$(APP).bim : $(APP).obj $(APILIBPATH)apilib.lib Makefile ../app_make.txt
	$(OBJ2BIM) @$(RULEFILE) out:$(APP).bim map:$(APP).map stack:$(STACK) \
		$(APP).obj $(APILIBPATH)apilib.lib

haribote.img : ../haribote/ipl20.bin ../haribote/haribote.sys $(APP).hrb \
		Makefile ../app_make.txt
	$(EDIMG)   imgin:../../../z_tools/fdimg0at.tek \
		wbinimg src:../haribote/ipl20.bin len:512 from:0 to:0 \
		copy from:../haribote/haribote.sys to:@: \
		copy from:$(APP).hrb to:@: \
		copy from:../nihongo/nihongo.fnt to:@: \
		imgout:haribote.img 

# rules

%.gas : %.c ../apilib.h Makefile ../app_make.txt
	$(CC1) -o $*.gas $*.c

%.nas : %.gas Makefile ../app_make.txt
	$(GAS2NASK) $*.gas $*.nas

%.obj : %.nas Makefile ../app_make.txt
	$(NASK) $*.nas $*.obj $*.lst

%.org : %.bim Makefile ../app_make.txt
	$(BIM2HRB) $*.bim $*.org $(MALLOC)

%.hrb : %.org Makefile ../app_make.txt
	$(BIM2BIN) -osacmp in:$*.org out:$*.hrb

# コマンド

run :
	$(MAKE) haribote.img
	$(COPY) haribote.img ..\..\..\z_tools\qemu\fdimage0.bin
	$(MAKE) -C ..\..\..\z_tools/qemu

full :
	$(MAKE) -C $(APILIBPATH)
	$(MAKE) $(APP).hrb

run_full :
	$(MAKE) -C $(APILIBPATH)
	$(MAKE) -C ..\haribote
	$(MAKE) run

clean :
	-$(DEL) *.lst
	-$(DEL) *.obj
	-$(DEL) *.map
	-$(DEL) *.bim
	-$(DEL) *.org
	-$(DEL) haribote.img

src_only :
	$(MAKE) clean
	-$(DEL) $(APP).hrb
	
```

次は非短径ウィンドウを作っていきます。   
notrec/Makefile
```Makefile
APP      = notrec
STACK    = 11k
MALLOC   = 0k

include ../app_make.txt
```

notrec.c
```c
#include "apilib.h"

void HariMain(void)
{
    int win;
    char buf[150 * 70];
    win = api_openwin(buf, 150, 70, 255, "notrec");
    api_boxfilwin(win,   0, 50,  34, 69, 255);
    api_boxfilwin(win, 115, 50, 149, 69, 255);
    api_boxfilwin(win,  50, 30,  99, 49, 255);
    for (;;) {
        if (api_getkey(1) == 0x0a) {
            break; /* Enter‚È‚çbreak; */
        }
    }
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
GOLIB	 = $(TOOLPATH)golib00.exe
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
		notrec/notrec.hrb
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
	-$(DEL) haribote.img

refresh :
	$(MAKE) full
	$(MAKE) clean_full
	-$(DEL) haribote.img
```

beautiful ballというアプリを作っていきます。   
bball.c
```c
#include "apilib.h"

void HariMain(void)
{
    int win, i, j, dis;
    char buf[216 * 237];
    struct POINT {
        int x, y;
    };
    static struct POINT table[16] = {
        { 204, 129 }, { 195,  90 }, { 172,  58 }, { 137,  38 }, {  98,  34 },
        {  61,  46 }, {  31,  73 }, {  15, 110 }, {  15, 148 }, {  31, 185 },
        {  61, 212 }, {  98, 224 }, { 137, 220 }, { 172, 200 }, { 195, 168 },
        { 204, 129 }
    };

    win = api_openwin(buf, 216, 237, -1, "bball");
    api_boxfilwin(win, 8, 29, 207, 228, 0);
    for (i = 0; i <= 14; i++) {
        for (j = i + 1; j <= 15; j++) {
            dis = j - i; /* 2つの点の距離 */
            if (dis >= 8) {
                dis = 15 - dis; /* 逆回りに数える */
            }
            if (dis != 0) {
                api_linewin(win, table[i].x, table[i].y, table[j].x, table[j].y, 8 - dis);
            }
        }
    }

    for (;;) {
        if (api_getkey(1) == 0x0a) {
            break; /* Enterならbreak; */
        }
    }
    api_end();
}

```

bball/Makefile
```Makefile
APP      = bball
STACK    = 52k
MALLOC   = 0k

include ../app_make.txt
```

Makefile
```Makefile
TOOLPATH = ..\..\z_tools\\

INCPATH  = ..\..\z_tools\haribote\\

MAKE     = $(TOOLPATH)make.exe -r
EDIMG    = $(TOOLPATH)edimg.exe
IMGTOL   = $(TOOLPATH)imgtol.com
GOLIB	 = $(TOOLPATH)golib00.exe
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
		notrec/notrec.hrb bball/bball.hrb
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
	-$(DEL) haribote.img

refresh :
	$(MAKE) full
	$(MAKE) clean_full
	-$(DEL) haribote.img
```

console.c
```c
else if (edx == 13) {
		sht = (struct SHEET *) (ebx & 0xfffffffe);
		hrb_api_linewin(sht, eax, ecx, esi, edi, ebp);
		if ((ebx & 1) == 0) {
			if (eax > esi) {
				i = eax;
				eax = esi;
				esi = i;
			}
			if (ecx > edi) {
				i = ecx;
				ecx = edi;
				edi = i;
			}
			sheet_refresh(sht, eax, ecx, esi + 1, edi + 1);
		}
```


**実装過程**   
・　[aa3dc855e675804e56b1ffc5ce38b1bc833475a9](aa3dc855e675804e56b1ffc5ce38b1bc833475a9)