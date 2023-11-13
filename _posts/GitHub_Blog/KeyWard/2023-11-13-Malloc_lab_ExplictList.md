---
title: "mallob lab - explict free list"
last_modified_at: "2023-11-13T14:30:00"
categories:
  - 크래프톤 정글
  - CS
tags:
  - 크래프톤 정글
  - CS
  - malloc lab
  - explict free list
---

## malloc lab
  implict free list에 대한 내용은 CSAPP의 9.9에 존재하여<br>
  해당 내용을 구현하여 테스트 제출 시, 약 54 점 정도 나왔던 걸로 기억한다<br>

  이후, implict free list 방식을 사용하여 코드를 다시 제작하였다<br>

  다만, 약 3일 정도 삽질을 열심히 하고,<br>
  뭔가 몇가지 아이디어가 부족하다고 느끼게 되어<br>
  구글링을 해본 결과, 내가 놓친 부분이 있다는 것을 다시 확인하게 되었다<br>

## 놓친 아이디어 init과 head 세팅
  처음에 가장 헷갈렸던 부분이 바로,<br>
  프롤로그 헤더에도 'PRED(이전)', 'SUCC(이후)' 포인터가 필요한가?<br>
  였으며, 이 이후에도 명시적 가용 리스트의 'head'를 어떻게 설정할지<br>
  많은 고민을 한 부분이었다<br>

  아이디어를 얻은 부분은<br>
  프롤로그 헤더의 PRED 위치를 'head'로 사용한다는 점이었다<br>

  해당 위치를 기준으로 head는 움직이지 않고,<br>
  head의 'SUCC'블록 부터 탐색을 시키는 방식이다<br>
  가장 마지막 가용 블록이 다시 head를 가리킴으로서<br>
  '마지막에 도착한 여부'를 확인할 수 있도록 한다<br>
  (프롤로그 헤더는 '할당'상태 이므로)<br>
  (또한 묵시적과 다르게, 헤더의 크기가 16바이트 인 점도 맹점이었다)<br>





## 구현 코드 (C)
```
/*
 * mm-naive.c - The fastest, least memory-efficient malloc package.
 *
 * In this naive approach, a block is allocated by simply incrementing
 * the brk pointer.  A block is pure payload. There are no headers or
 * footers.  Blocks are never coalesced or reused. Realloc is
 * implemented directly using mm_malloc and mm_free.
 *
 * NOTE TO STUDENTS: Replace this header comment with your own header
 * comment that gives a high level description of your solution.
 */
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <unistd.h>
#include <string.h>

#include "mm.h"
#include "memlib.h"

team_t team = {
    "krafton_j_3_6",
    "Hyun JaeHoon",
    "hnjogo@gmail.com",
    "",
    ""};

/* Basic constants and macros */
#define WSIZE 4             /* WORD and header / footer size*/
#define DSIZE 8             /* Double Word Size*/
#define CHUNKSIZE (1 << 12) /* Extend heap by this amount (bytes) */

#define MAX(x, y) ((x) > (y) ? (x) : (y))

/* Pack a size and allocated bit into a word */
#define PACK(size, alloc) ((size) | (alloc))

/* Read and write a word at address p */
#define GET(p) (*(unsigned int *)(p))
#define PUT(p, val) (*(unsigned int *)(p) = (val))

/* Read the size and allocated fields from address p */
#define GET_SIZE(p) (GET(p) & ~0x7) /* 하위 3비트를 제외한 녀석들이 블록의 크기 비트를 표현한다  */
#define GET_ALLOC(p) (GET(p) & 0x1)

/* Given block ptr bp, compute address of its header and footer */
#define HDRP(bp) ((char *)(bp)-WSIZE)
#define FTRP(bp) ((char *)(bp) + GET_SIZE(HDRP(bp)) - DSIZE)

#define PRED_P(bp) ((char*)(bp))
#define SUCC_P(bp) ((char*)(bp) + WSIZE)

#define SET_PRED_P(bp,targetBp) (PUT(PRED_P(bp),targetBp))
#define SET_SUCC_P(bp,targetBp) (PUT(SUCC_P(bp),targetBp))

#define GET_PRED_P(bp) ((void*)(GET(PRED_P(bp))))
#define GET_SUCC_P(bp) ((void*)(GET(SUCC_P(bp))))

/* Given block ptr bp, compute address of next and previous blocks */
#define NEXT_BLKP(bp) ((char *)(bp) + GET_SIZE((char *)(bp) - WSIZE))
#define PREV_BLKP(bp) ((char *)(bp) - GET_SIZE((char *)(bp) - DSIZE))

static void *extend_heap(size_t words);
static void *coalesce(void *bp);
static void *find_fit(size_t asize);
static void place(void *bp, size_t asize);
static void arrageBlock(void *targetBp);
static void headInsert(void* bp);

void *heap_listp = NULL;

/*
 * mm_init - initialize the malloc package.
 */
int mm_init(void)
{
    /* 초기화 */
    if((heap_listp = mem_sbrk(6*WSIZE)) == (void *)-1)
        return -1;
    PUT(heap_listp, 0);
    PUT(heap_listp + (1*WSIZE), PACK(2 * DSIZE, 1));        // 프롤로그 header (총 사이즈 16바이트)
    PUT(heap_listp + (2*WSIZE), heap_listp+(3 * WSIZE));    // 프롤로그 PRED_P (현재는 자신의 SUCC 가리킴)
    PUT(heap_listp + (3*WSIZE), heap_listp+(2 * WSIZE));    // 프롤로그 SUCC_P (현재는 자신의 PRED 가리킴)
    PUT(heap_listp + (4*WSIZE), PACK(2 * DSIZE, 1));        // 프롤로그 footer
    PUT(heap_listp + (5*WSIZE), PACK(0 , 1));               // 에필로그 header
    heap_listp += 2*WSIZE; // 현재는 프롤로그의 '이전'을 첫번째 head로 가리킨다 (시작점, 아래에서 바뀐다)
    if (extend_heap(CHUNKSIZE/WSIZE) == NULL)
        return -1;
    return 0;
}

/* 새 가용 블록으로 힙 확장 */
static void *extend_heap(size_t words)
{
    char *bp;
    size_t size;

    /* size 홀수이면 1 더해서 WSIZE 곱해주고 대입*/
    size = (words % 2) ? (words + 1) * WSIZE : words * WSIZE;

    /* 힙 할당 해주고 bp에 넣어줌*/
    if ((long)(bp = mem_sbrk(size)) == -1)
        return NULL; /* 힙 할당은... 실패다...*/

    /* 새로운 가용블록의 헤더랑 푸터랑 초기화해주기 */
    PUT(HDRP(bp), PACK(size, 0));
    PUT(FTRP(bp), PACK(size, 0));

    /* 새로운 에필로그 헤더 생성*/
    PUT(HDRP(NEXT_BLKP(bp)), PACK(0, 1));

    return coalesce(bp);
}

/*
 * mm_malloc - Allocate a block by incrementing the brk pointer.
 *     Always allocate a block whose size is a multiple of the alignment.
 */
void *mm_malloc(size_t size)
{
    size_t asize;
    size_t extendsize;
    char *bp;

    if (size == 0)
        return NULL;

    if (size <= DSIZE)
    {
        /* 최소 할당 크기 : 16*/
        asize = DSIZE * 2;
    }
    else
    {
        /* 8 바이트 정렬 요건 만족을 위한 할당 사이즈*/
        asize = DSIZE * ((size + DSIZE + DSIZE - 1) / DSIZE);
    }

    if ((bp = find_fit(asize)) != NULL)
    {
        place(bp, asize);
        return bp;
    }

    extendsize = MAX(asize, CHUNKSIZE);
    if ((bp = extend_heap(extendsize / WSIZE)) == NULL)
        return NULL;

    place(bp, asize);
    return bp;
}

static void *find_fit(size_t asize)
{
    /* first fit*/
    void *bp;

    for (bp = GET_SUCC_P(heap_listp);   // 시작 : heap_listp가 다음으로 가리키는 블록 (어차피 heap_listp는 프롤로그 헤더의 'PRED'에 존재하고 거기서 head 역할을 해준다)
     !GET_ALLOC(HDRP(bp));          // 종료조건 : 할당되어 있다면 프롤로그 블록까지 온 상황이므로 종료시킨다
      bp = GET_SUCC_P(bp))              // bp를 다음 지정된 가용 블록으로 이동시킨다
    {
        // 할당 안된거라면 사이즈 재본다
        if (asize <= GET_SIZE(HDRP(bp)))
        {
            return bp;
        }
    }

    return NULL;
}

/* 요청한 블록을 가용 블록의 시작 부분에 배치,
나머지 부분의 크기가 최소 블록 크기와 같거나 큰 경우에만 분할 */
static void place(void *bp, size_t asize)
{
    size_t csize = GET_SIZE(HDRP(bp));
    // 일단 이 함수에 들어왔다는 것은
    // bp가 할당된 공간의 위치를 가리킬 예정이므로
    // 먼저 list 내부를 정리시켜 준다
    arrageBlock(bp);

    /* 할당 사이즈(asize)가 시작 블록 사이즈(cSize)보다 
       16바이트(최소 할당 크기)보다 큰 경우 잘라내기 위함*/
    if ((csize - asize) >= (2 * DSIZE))
    {
        PUT(HDRP(bp), PACK(asize, 1));
        PUT(FTRP(bp), PACK(asize, 1));

        // 할당 해제할 블록들
        bp = NEXT_BLKP(bp);
        PUT(HDRP(bp), PACK(csize - asize, 0));
        PUT(FTRP(bp), PACK(csize - asize, 0));

        // 새로 할당된 블록이므로
        // 해당 블록을 head로 설정 (LIFO)
        headInsert(bp);
    }
    else /* 할당 사이즈와 시작 블록 사이즈의 차이가, 최소 할당크기보다 작다면 내버려둠*/
    {
        PUT(HDRP(bp), PACK(csize, 1));
        PUT(FTRP(bp), PACK(csize, 1));
    }
}

static void arrageBlock(void *targetBp)
{
    if (targetBp == NULL)
        return;

    // 이중 연결 리스트의 정리 과정이다
    // 해당 블록의 이전은 '다음 블록'의 이전으로 보내고
    // 해당 블록의 '다음'은 '이전 블록'의 다음으로 보낸다
    void *predBp = GET_SUCC_P(targetBp);
    void *succBp = GET_PRED_P(targetBp);

    if (predBp != NULL)
    {
        SET_PRED_P(predBp,succBp);
    }

    // 둘 중 하나가 비어 있다면 null이 들어간다
    if (succBp != NULL)
    {
        SET_SUCC_P(succBp,predBp);
    }
}

// 가장 처음으로 연결 시킨다
// 이중 연결 리스트
static void headInsert(void* bp)
{
    
    SET_SUCC_P(bp,GET_SUCC_P(heap_listp));        // 해당 요소의 다음은 heap_listp(head)의 다음으로
    SET_PRED_P(bp,heap_listp);                // 해당 요소의 이전을 head로 설정하기
    SET_PRED_P(GET_SUCC_P(heap_listp),bp);        // head가 원래 가리키던 블록의 '이전'을 현재 bp로 설정하고
    SET_SUCC_P(heap_listp,bp);                // head의 다음 블록으로 bp 설정하기
}

void mm_free(void *bp)
{
    if (bp == NULL)
        return;

    size_t size = GET_SIZE(HDRP(bp));

    PUT(HDRP(bp), PACK(size, 0));
    PUT(FTRP(bp), PACK(size, 0));
    coalesce(bp);
}

// 블록 병합
// 이쪽으로 들어오는 경우는
// place와는 반대로, '블록'이 '해제'되거나 새로 생기는 경우이다
static void *coalesce(void *bp)
{
    size_t prev_alloc = GET_ALLOC(FTRP(PREV_BLKP(bp))); // 이전 footer
    size_t next_alloc = GET_ALLOC(HDRP(NEXT_BLKP(bp)));
    size_t size = GET_SIZE(HDRP(bp));

    if (prev_alloc && !next_alloc)
    {
        /* case 2 이전 블록은 할당되었고, 다음 블록은 가용상태 */
        size += GET_SIZE(HDRP(NEXT_BLKP(bp))); /* 다음 블록의 사이즈만큼 더함 (다음 헤더에서 가지고 온다) */

        // 가용 상태인 다음 블록을 정리해준다
        arrageBlock(NEXT_BLKP(bp));

        PUT(HDRP(bp), PACK(size, 0)); /*size | 0 은 size와 같음, 그러나 가독성과 통일성을 위해 사용*/
        PUT(FTRP(bp), PACK(size, 0));
    }
    else if (!prev_alloc && next_alloc)
    {
        /* case 3 이전 블록은 가용 상태, 다음 블록은 할당상태 */

        // 가용 상태인 이전 블록을 정리해준다
        arrageBlock(PREV_BLKP(bp));
        size += GET_SIZE(HDRP(PREV_BLKP(bp)));
        
        PUT(FTRP(bp), PACK(size, 0));
        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));

        // 이전 블록과 합쳤으므로 이전 블록의 블록 포인터로 옮긴다
        bp = PREV_BLKP(bp);
    }
    else if (!prev_alloc && !next_alloc)
    {
        /* case 4 이전 블록과 다음 블록 모두 가용 상태*/
        // 두 블록 모두 정리해준다
        arrageBlock(PREV_BLKP(bp));
        arrageBlock(NEXT_BLKP(bp));

        size += GET_SIZE(HDRP(PREV_BLKP(bp))) + GET_SIZE(FTRP(NEXT_BLKP(bp)));

        PUT(HDRP(PREV_BLKP(bp)), PACK(size, 0));
        PUT(FTRP(NEXT_BLKP(bp)), PACK(size, 0));

        // 이전 블록의 블록 포인터로 옮긴다
        bp = PREV_BLKP(bp);
    }

    // 정리 완료된 블록을
    // head에 넣어준다
    headInsert(bp);

    return bp;
}

/*
 * mm_realloc - Implemented simply in terms of mm_malloc and mm_free
 */
void *mm_realloc(void *ptr, size_t size)
{
    void *oldptr = ptr;
    void *newptr;
    size_t copySize;

    newptr = mm_malloc(size);
    if (newptr == NULL)
        return NULL;

    copySize = GET_SIZE(HDRP(oldptr));
    if (size < copySize)
        copySize = size;
    memcpy(newptr, oldptr, copySize);
    mm_free(oldptr);
    return newptr;
}


```