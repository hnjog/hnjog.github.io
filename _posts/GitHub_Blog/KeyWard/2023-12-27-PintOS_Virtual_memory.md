---
title: "Pintos 3 VM"
last_modified_at: "2023-12-27T11:30:00"
categories:
  - 크래프톤 정글
  - CS
  - OS
tags:
  - 크래프톤 정글
  - CS
  - OS
  - PintOS
  - 가상 메모리
---

## pintOS Project 3
![non_extra_complete](https://github.com/hnjog/hnjog.github.io/assets/43630972/d984e2d4-1079-4964-855d-e38312f05079)

 개인적으로 많은 것을 고민하게 된 프로젝트 였다<br>
 '빠르게 코드'와 '동작'을 파악하고, 이에 걸맞게 '구현'을 한 뒤,<br>
 '디버깅'을 통해 '테스트 케이스'를 통과할 수 있었다면<br>
 가장 성공적인 공부를 할 수 있었다 말할 수 있겠지만...<br>

 생각보다 2주라는 시간은 나에게는 짧았던 모양이고<br>
 남은 선택지는<br>
 - 모든 테스트 케이스를 통과하지 못하더라도 가능한 나의 힘으로 코드를 짜본다<br>
 - 외부 코드를 참고하고, 현재 프로젝트 상황에 맞게 수정하여 반영한 후,<br>
   코드를 이해하도록 노력한다<br>

 나는 후자를 택했는데,<br>
 스스로 '문제가 요구하는' 알고리즘을 구현하는 능력이 아직 부족하였던 것 같다<br>
 'pintOS'를 '공부'한다는 점에서 전자의 선택이 더 가치가 있지 않나.. 고민도 하였지만<br>
 문제는 스스로 고민하여도 답은 안나오고 시간만 흐르는 것이 가장 큰 문제인 것 같았다<br>
 
 해당 내용에서 가능한 이번 project 3 에 대하여 적은 내용을 추가적으로 정리하려 한다<br>

## Memory Managemnet
 해당 과제의 요점은<br>
 'spt'와 '물리 메모리 매핑'이다<br>

 project2 까지는 'pml4'를 사용하였다<br>
 (이는 multi level page table 방식이며 이에 관련된 블로깅을 남겼었다)<br>
 [https://hnjog.github.io/%ED%81%AC%EB%9E%98%ED%94%84%ED%86%A4%20%EC%A0%95%EA%B8%80/cs/os/Multi_Level_Page_Table/]<br>

 다만 기존의 방식은 가상 주소와 물리 주소의 1 대 1 매핑이기에<br>
 메인 메모리(DRAM)가 부족한 경우에 대하여 Swap을 구현할 수 없는 상황이기에<br>
 그에 따라 'Page'와 'Frame'이라는 구조체를 사용하여<br>
 swap 및 page fault에 따른 자원 관리를 용이하게 처리하도록 한다<br>

 Page 구조체<br>
```
struct page_operations {
	bool (*swap_in) (struct page *, void *);
	bool (*swap_out) (struct page *);
	void (*destroy) (struct page *);
	enum vm_type type;
};

struct page {
	const struct page_operations *operations;
	void *va;              /* Address in terms of user space */
	struct frame *frame;   /* Back reference for frame */

	bool isWritable;

	struct hash_elem spt_hash_elem;

	/* Per-type data are binded into the union.
	 * Each function automatically detects the current union */
	union {
		struct uninit_page uninit;
		struct anon_page anon;
		struct file_page file;
	};
};
```
 <br>
 'Page'에 대한 개념이 담긴 구조체로서<br>
 가상 주소와 매핑되는 'Frame' 구조체를 필드로 가지게 된다<br>
 operations 라는 추가적인 구조체와<br>
 union 필드를 통하여<br>
 '현재' 설정된 '페이지 타입'에 따라 다른 함수를 호출하도록 하였다<br>
 (함수 포인터를 이용하여, 처음 'init'을 할 때, 해당 페이지 타입의<br>
 init 함수를 호출하는 점이 인상적이었다)<br><br>

```
typedef bool(*initializerFunc)(struct page *, enum vm_type, void *);
initializerFunc initializer = NULL;

// vm_type에 따라 다른 initializer를 부른다.
switch(VM_TYPE(type)){
    case VM_ANON:
        initializer = anon_initializer;
        break;
    case VM_FILE:
        initializer = file_backed_initializer;
        break;
}
```

Frame 구조체<br>
```
struct frame {
	void *kva;
	struct page *page;
	struct list_elem frame_elem;
};
```

 kva(PA)와 직접적인 매핑이 되는<br>
 frame 구조체이다<br>
 실제 kva의 위치가 'user pool'에 할당된다<br>
 (user pool의 위치에 존재하는 frame 들이 페이지 교체의 대상이 된다)<br>
 (kern pool의 위치에 존재하는 커널 코드들이 페이지 교체의 대상이 되는 것은<br>
 여러 모로 단점이 더 많은 상황이 되기에, 늘 안정적으로 동작하거나 참조해야 한다면<br>
 커널에 palloc을 해주는 것이 좋다는 것을 알았다)<br>
 [관련 TMI]<https://hnjog.github.io/%ED%81%AC%EB%9E%98%ED%94%84%ED%86%A4%20%EC%A0%95%EA%B8%80/cs/os/Pintos3_vm%EC%A7%84%ED%96%89%EC%A4%912/><br><br>

 thread의 'spt'는 자신의 'Page' 구조체를 가지는 hash Table로 구현하였고<br>
 'Page' 할당을 요청하는 경우, 해당 spt에 넣어<br>
 관리되게 된다<br>

 또한 'lazy_load' 방식이기에<br>
 처음 page를 할당할 때, spt에 넣어둔 후,<br>
 swap_in 이 호출 시, 해당 페이지 타입에 따라 초기화를 호출하게 된다<br>
 (이 때, unit_type으로 설정해둔 뒤<br>
 실제 할당을 할 때, lazy_load_segment 함수와<br>
 타입에 따른 initalize 함수를 호출하는 방식이다)<br>

```
아래의 함수 호출 시,
vm_alloc_page_with_initializer (VM_ANON, upage,
					writable, lazy_load_segment, aux)

bool
vm_alloc_page_with_initializer (enum vm_type type, void *upage, bool writable,
		vm_initializer *init, void *aux) {

	ASSERT (VM_TYPE(type) != VM_UNINIT)

	struct supplemental_page_table *spt = &thread_current ()->spt;

	/* Check wheter the upage is already occupied or not. */
	if (spt_find_page (spt, upage) == NULL) {
		/*
			페이지를 만들고 VM 유형에 따라 이니셜을 가져온 다음
			uninit_new를 호출하여 "uninit" 페이지 구조를 만듭니다.
			uninit_new를 호출한 후 필드를 수정해야 합니다.
		*/

		struct page* newPage = (struct page *)malloc(sizeof(struct page));
		// 이랬을 때, 할당 못받을 경우에 대하여???
		if(newPage == NULL)
		{
			goto err;
		}

		typedef bool(*initializerFunc)(struct page *, enum vm_type, void *);
        initializerFunc initializer = NULL;

        // vm_type에 따라 다른 initializer를 부른다.
        switch(VM_TYPE(type)){
            case VM_ANON:
                initializer = anon_initializer;
                break;
            case VM_FILE:
                initializer = file_backed_initializer;
                break;
        }

		// 이대로 그냥 넣으면 null이라서 내부에서 assert
		uninit_new(newPage,upage,init,type,aux,initializer);

		newPage->isWritable = writable;

		return spt_insert_page(spt,newPage);
	}
err:
	return false;
}
```

- TMI : Page 와 Frame 구조체에 대한 정보는 '커널 영역'에서 관리하도록<br>
        malloc을 통해 할당해준다<br>
        (엄밀히 말하자면 pintos에서 힙 영역은 따로 관리되지 않으며,<br>
         이는 kernel pool에 palloc을 해준 것과 비슷하게 동작된다)<br>
        (아마 커널 영역에 '힙'이 존재하지 않기에 이와 같은 처리를 해준듯 하다)<br><br>
        우리가 page 교체를 해주는 영역은 'user' 영역이기에<br>
        이러한 데이터를 'user'영역에 할당하는 경우<br>
        'swap'을 위한 구조체 데이터 역시, 'swap'의 대상이 되어버리기에<br>
        성능 뿐 아니라 안정적인 면에서 해당 데이터들은 kernel 영역에 선언되는 것이<br>
        바람직하다<br>
        
 - 페이지 교체 알고리즘<br>
   : LRU와 Clock 중 어떠한 것을 써야할지 잠시 생각하였으나<br>
     직관적이고 구현이 쉬운 Clock을 사용하였다<br>
     다만 해당 부분을 'while'문으로 구현하던 중<br>
     다음 page로 넘어가는 '조건문'을 넣어주지 않아<br>
     swap-file 테스트 케이스를 통과하지 못했었다<br>
     ~~(근데 왜 나머지는 통과했을까...)~~

## Anonymous Page, File-backed page
 위에서 일부 설명한 lazy_loading 에 관련된 내용이 포함되었다<br>
 lazy loading 방식은 필요할 때까지 DRAM 즉, 메인 메모리의 할당을 늦추어<br>
 자원을 효율적으로 사용할 수 있도록 한다<br>

 이를 위하여 lazy_load_segment를 이용하며,<br>
 uninit_new로 만들어진 페이지 내부에서 'swap-in' 될 때<br>
 아래의 'page_initializer' 필드가 lazy_load_segment를 담고 있기에<br>
 lazy_load_segment 이후 각 page의 init 함수가 호출되도록 제작한다<br>

```
static const struct page_operations uninit_ops = {
	.swap_in = uninit_initialize,
	.swap_out = NULL,
	.destroy = uninit_destroy,
	.type = VM_UNINIT,
};

void
uninit_new (struct page *page, void *va, vm_initializer *init,
		enum vm_type type, void *aux,
		bool (*initializer)(struct page *, enum vm_type, void *)) {
	ASSERT (page != NULL);

	*page = (struct page) {
		.operations = &uninit_ops,
		.va = va,
		.frame = NULL, /* no frame for now */
		.uninit = (struct uninit_page) {
			.init = init,
			.type = type,
			.aux = aux,
			.page_initializer = initializer,
		}
	};
}

static bool
uninit_initialize (struct page *page, void *kva) {
	struct uninit_page *uninit = &page->uninit;

	/* Fetch first, page_initialize may overwrite the values */
	vm_initializer *init = uninit->init;
	void *aux = uninit->aux;

	/* TODO: You may need to fix this function. */
	return uninit->page_initializer (page, uninit->type, kva) &&
		(init ? init (page, aux) : true);
}
``` 

Swap-In / Swap-Out 에 따라 각각의 구현이 달라진다는 점도 유의해야 했다<br>
이전에 권영진 교수님 강의에 적어 놓았듯,<br>

Page Fault 에 따른 swap 발생 시,
Anonymous<br>
  - Swap In : disk의 Swap 영역에서 찾는다 <br>
    (그렇기에 Anon-Page 에선 swap out 시, index를 저장하고<br>
     이를 찾아 저장해놓은 disk 위치를 찾음)<br>
  - Swap Out : disk의 비어있는 Swap 영역에 데이터를 쓰고<br>
               페이지의 'present' 비트를 해제하여<br>
               물리 메모리에 적재되지 않았음을 표기한다<br>
               (이 때, disk에 저장한 위치를 저장해 놓아야 한다)<br>

File-backed<br>
  - Swap In : 지정된 offset의 위치의 파일 데이터를 읽어온다<br>
  - Swap Out : dirty beat를 체크하고, 그에 따라 file의 위치에 써준다<br>


## stack growth
 스택이 현재 가진 '크기'보다 추가적으로 필요로 할 시<br>
 스택의 크기를 늘려준다<br>

 먼저 '접근'하려는 주소가 spt 테이블에 존재한다면<br>
 단순히, 해당 주소에 대한 물리 주소를 spt_find_page를 통하여<br>
 바로 반환한다면 문제없지만<br>
 그렇지 않은 경우, 추가적으로 stack의 크기를 늘려 해결해야 한다<br>
 
```
static void
vm_stack_growth (void *addr UNUSED) 
{
	if(vm_alloc_page(VM_ANON | VM_MARKER_0, addr, true))
	{
        vm_claim_page(addr);
        thread_current()->stack_bottom -= PGSIZE;
    }
}

bool
vm_try_handle_fault (struct intr_frame *f UNUSED, void *addr UNUSED,
		bool user UNUSED, bool write UNUSED, bool not_present UNUSED) {
	struct supplemental_page_table *spt UNUSED = &thread_current ()->spt;

	// addr은 '가상 메모리'의 위치
	// 따라서 'user' 영역이 아니면 안됨
	if (is_kernel_vaddr(addr) || addr == NULL || not_present == false)
	{
		return false;
	}

	// 페이지 요청했는데 존재한다
	if(vm_claim_page(addr) == true)
	{
		return true;
	}
	
	// 0x100000 == (0001 0000 0000 0000 0000 0000) == 1 << 20
	// pintOS에서 stack의 크기를 1MB로 제한하기에 (by git book)
    const uintptr_t one_megaByte = (1 << 20);
	uintptr_t stack_limit = USER_STACK - one_megaByte;
	uintptr_t rsp = user ? f->rsp : thread_current()->user_rsp;

	// rsp - 8 == addr 인 경우가 존재하고, 이를 stack_grow로 해결할 수 있음
	// -> PUSH 명령 (stack에 데이터 추가)
	// rsp의 다음 위치에 데이터를 추가하려 할 때, stack의 크기를 늘려줌으로서 이를 해결할 수 있음
	// by git book
	/*
		However, the x86-64 PUSH instruction checks access permissions before it adjusts the stack pointer, 
		so it may cause a page fault 8 bytes below the stack pointer.
	*/
	if (addr >= rsp - 8 && 
	addr <= USER_STACK  &&
	 addr >= stack_limit)
	{
		vm_stack_growth(thread_current()->stack_bottom - PGSIZE);
		return true;
    }

	return false;
}
```
 
## Memory Mapped File
 mmap 시스템 콜과 unmap 시스템 콜에 대한 구현이 필요한 과제이다<br>
 물리 메모리에 'file-backed' 메모리로 page를 할당한다<br>
 기본적으로는 'load-segment'와 비슷한 방식으로 구현<br>

 또한 unmap의 경우는 file-backed 메모리가 swap-out 하는 방식과 유사하게 구현된다<br>

```
void *
do_mmap (void *addr, size_t length, int writable, struct file *file, off_t offset) 
{
	// addr 부터 연속된 user virtual address space에
	// page들을 만들어 offset 부터 length 까지
	// 해당하는 file의 정보를 각 page에 저장

	// 기본적으로 'process'의 'load segment' 와 비슷하게 처리된다

	// 파일을 다시 열어준다
	// 정확히는 내부의 'open_cnt'를 조절하여
	// 누가 함부로 닫지 않도록 해준다
	struct file* reopenFile = file_reopen(file);

	void* origin_addr = addr;
	size_t read_bytes = length > file_length(file) ? file_length(file) : length;
	size_t zero_bytes = PGSIZE - (read_bytes % PGSIZE);

	ASSERT((read_bytes + zero_bytes) % PGSIZE == 0);
    ASSERT(pg_ofs(addr) == 0);      // upage가 페이지 정렬되어 있는지 확인
    ASSERT(offset % PGSIZE == 0); // ofs가 페이지 정렬되어 있는지 확인

	while (read_bytes > 0 || zero_bytes > 0) 
	{
		size_t page_read_bytes = read_bytes < PGSIZE ? read_bytes : PGSIZE;
		size_t page_zero_bytes = PGSIZE - page_read_bytes;

		struct loadArgs* largP = (struct loadArgs*)malloc(sizeof(struct loadArgs));
		if(largP == NULL)
		{
			return false;
		}

		largP->file = reopenFile;
		largP->fileOfs = offset;
		largP->readByte = page_read_bytes;

		void *aux = largP;
		if (!vm_alloc_page_with_initializer (VM_FILE, addr,
					writable, lazy_load_segment, aux))
		{
			free(largP);
			return false;
		}

		/* Advance. */
		read_bytes -= page_read_bytes;
		zero_bytes -= page_zero_bytes;
		addr += PGSIZE;
		// 나중에 읽어줄 거라
		// 해당 시점에서의 ofs가 따로 필요함
		// '반복문'을 돌면서 page를 따로 할당할 수 있음
		offset += page_read_bytes;
	}

	// 시작 위치
	return origin_addr;
}

/* Do the munmap */
void
do_munmap (void *addr)
{
	struct thread *curr = thread_current();
	while (true)
	{
		struct page *targetPage = spt_find_page(&curr->spt, addr);
		if (targetPage == NULL)
			return;

		struct loadArgs *aux = (struct loadArgs *)targetPage->uninit.aux;

		// null 인 경우 아래에서 null 참조가 일어나게 된다
		// dirty check
		if (pml4_is_dirty(curr->pml4, targetPage->va) == true)
		{
			file_write_at(aux->file, addr, aux->readByte, aux->fileOfs);
			pml4_set_dirty(curr->pml4, targetPage->va, false);
		}

		pml4_clear_page(curr->pml4, targetPage->va);

		addr += PGSIZE;
	}
}
```

 이전에 포스팅한 mmap 관련 내용<br>
 <https://hnjog.github.io/%ED%81%AC%EB%9E%98%ED%94%84%ED%86%A4%20%EC%A0%95%EA%B8%80/cs/memory/Mmap/>
