---
tags: [linux]
title: linux内存管理
created: '2020-03-06T02:54:14.427Z'
modified: '2020-03-06T04:27:31.827Z'
---

> linux内存管理

<!-- more -->


## 相关的数据结构

### node数据结构（numa）

* pglist_data（pg_data_t）的表头是pgdat_list

```c++
typedef struct pglist_data {
        zone_t node_zones[MAX_NR_ZONES];                该节点所在管理区为HIGHMEM、NORMAL、DAM三者之一。
        zonelist_t node_zonelists[NR_GFPINDEX];         按照分配时的管理区顺序排列。通过page_alloc.c中的 bulid)zaonelists()建立顺序。
        struct page *node_mem_map;                      struct page数组中的第一个页面,被放置在全局mem_map数组中。
        unsigned long *valid_addr_bitmap;               描述内存节点中空洞的位图，因为并没有实际的内存空间存在。
        struct bootmem_data *bdata;                     指向内存引导程序
        unsigned long node_start_paddr;                 节点的起始物理地址
        unsigned long node_start_mapnr;                 它指出该节点在全局mem_map中的页面偏移，通过计算mem_map与该节点的局部mem_map中成为lmem_map之间的页面数，得到页面偏移。
        unsigned long node_size;                        管理区中的页面总数
        int node_id;                                    节点的ID号（NID），从0开始
        struct pglist_data *node_next;                  指向下一个节点。所有的节点都有一个pgdat_list的链表维护，这些节点都放在该链表中。
} pg_data_t;
```

### ZONE数据结构(内存管理区)
    对于x86机器，管理区的示例如下
* ZONE_DMA  16MB
* ZONE_NORMAL 16MB~896MB
* ZONE_HIGHMEM 896MB~未尾

```c++
typedef struct zone_struct {
        spinlock_t                   lock;                                         并行访问时保护该管理区的自旋锁。
        unsigned long                offset;
        unsigned long                free_pages;                                   该管理区中空闲页面的总数。
        unsigned long                inactive_clean_pages;
        unsigned long                inactive_dirty_pages;
        unsigned long                pages_min, pages_low, pages_high;             管理区极值,pgaes_min(保留的页框池)
        struct list_head             inactive_clean_list;
        free_area_t                  free_area[MAX_ORDER];                         空闲区域位图。
        char                        *name;                                         管理区的字符串名字“DMA”“Normal”“HighMem”
        wait_queue                   *wait_table;                                  等待队列hash表
        unsigned long                wait_table_size;
        unsigned long                wait_table_shift;
        unsigned long                size;                                         该管理区的大小，以页面数计算
        struct pglist_data           *zone_pgdat;                                  指向父pg_data_t
        unsigned long                zone_start_paddr;                             节点的起始物理地址
        unsigned long                zone_start_mapnr;                             节点在全局mem_map中的页面偏移
        struct page                  *zone_mem_map;                                设计的管理区在全局mem_map中的第一页
} zone_t;
```

* unsigned long    pages_min, pages_low, pages_high;管理区极值
pages_min最有可能的值是大于等于20（80KB）个页面小于等于255（1MB）个页面
当空闲页面数达到pages_low时，伙伴分配器就会唤醒kswapd(守护程序)释放页面，pages_low默认值是pages_min的两倍。当空闲页面数达到pages_min时，分配器会以同步方式启动kswapd。kswapd被唤醒并开始释放页面后，在 pages_high个页面被释放以前，是不会认为该管理区已经“平衡”的，当到pages_high值后，kswapd再次睡眠。pages_high默认是pages_min的三倍。


### PAGE
    所有的page都存储在一个全局的mem_map数组中，该数组通常放在ZONE_NORMAL的首部，或者就在小内存系统中为装入内核映像而预留的区域之后。



```c++
/*
 * Each physical page in the system has a struct page associated with
 * it to keep track of whatever it is we are using the page for at the
 * moment. Note that we have no way to track which tasks are using
 * a page, though if it is a pagecache page, rmap structures can tell us
 * who is mapping it.
 *
 * The objects in struct page are organized in double word blocks in
 * order to allows us to use atomic double word operations on portions
 * of struct page. That is currently only used by slub but the arrangement
 * allows the use of atomic double word operations on the flags/mapping
 * and lru list pointers also.
 */
struct page {
    /* First double word block */
    unsigned long flags;        /* Atomic flags, some possibly updated asynchronously
                                              描述page的状态和其他信息  */
    union
    {
        struct address_space *mapping;  /* If low bit clear, points to
                         * inode address_space, or NULL.
                         * If page mapped as anonymous
                         * memory, low bit is set, and
                         * it points to anon_vma object:
                         * see PAGE_MAPPING_ANON below.
                         */
        void *s_mem;            /* slab first object */
        atomic_t compound_mapcount;     /* first tail page */
        /* page_deferred_list().next     -- second tail page */
    };
 
    /* Second double word */
    struct {
        union {
            pgoff_t index;      /* Our offset within mapping.
            在映射的虚拟空间（vma_area）内的偏移；
            一个文件可能只映射一部分，假设映射了1M的空间，
            index指的是在1M空间内的偏移，而不是在整个文件内的偏移。 */
            void *freelist;     /* sl[aou]b first free object */
            /* page_deferred_list().prev    -- second tail page */
        };
 
        union {
#if defined(CONFIG_HAVE_CMPXCHG_DOUBLE) && \
    defined(CONFIG_HAVE_ALIGNED_STRUCT_PAGE)
            /* Used for cmpxchg_double in slub */
            unsigned long counters;
#else
            /*
             * Keep _refcount separate from slub cmpxchg_double
             * data.  As the rest of the double word is protected by
             * slab_lock but _refcount is not.
             */
            unsigned counters;
#endif
 
            struct {
 
                union {
                    /*
                     * Count of ptes mapped in mms, to show
                     * when page is mapped & limit reverse
                     * map searches.
                     * 页映射计数器
                     */
                    atomic_t _mapcount;
 
                    struct { /* SLUB */
                        unsigned inuse:16;
                        unsigned objects:15;
                        unsigned frozen:1;
                    };
                    int units;      /* SLOB */
                };
                /*
                 * Usage count, *USE WRAPPER FUNCTION*
                 * when manual accounting. See page_ref.h
                 * 页引用计数器
                 */
                atomic_t _refcount;
            };
            unsigned int active;    /* SLAB */
        };
    };
 
    /*
     * Third double word block
     *
     * WARNING: bit 0 of the first word encode PageTail(). That means
     * the rest users of the storage space MUST NOT use the bit to
     * avoid collision and false-positive PageTail().
     */
    union {
        struct list_head lru;   /* Pageout list, eg. active_list
                     * protected by zone->lru_lock !
                     * Can be used as a generic list
                     * by the page owner.
                     */
        struct dev_pagemap *pgmap; /* ZONE_DEVICE pages are never on an
                        * lru or handled by a slab
                        * allocator, this points to the
                        * hosting device page map.
                        */
        struct {        /* slub per cpu partial pages */
            struct page *next;      /* Next partial slab */
#ifdef CONFIG_64BIT
            int pages;      /* Nr of partial slabs left */
            int pobjects;   /* Approximate # of objects */
#else
            short int pages;
            short int pobjects;
#endif
        };
 
        struct rcu_head rcu_head;       /* Used by SLAB
                         * when destroying via RCU
                         */
        /* Tail pages of compound page */
        struct {
            unsigned long compound_head; /* If bit zero is set */
 
            /* First tail page only */
#ifdef CONFIG_64BIT
            /*
             * On 64 bit system we have enough space in struct page
             * to encode compound_dtor and compound_order with
             * unsigned int. It can help compiler generate better or
             * smaller code on some archtectures.
             */
            unsigned int compound_dtor;
            unsigned int compound_order;
#else
            unsigned short int compound_dtor;
            unsigned short int compound_order;
#endif
        };
 
#if defined(CONFIG_TRANSPARENT_HUGEPAGE) && USE_SPLIT_PMD_PTLOCKS
        struct {
            unsigned long __pad;    /* do not overlay pmd_huge_pte
                         * with compound_head to avoid
                         * possible bit 0 collision.
                         */
            pgtable_t pmd_huge_pte; /* protected by page->ptl */
        };
#endif
    };
 
    /* Remainder is not double word aligned */
    union {
        unsigned long private;      /* Mapping-private opaque data:
                         * usually used for buffer_heads
                         * if PagePrivate set; used for
                         * swp_entry_t if PageSwapCache;
                         * indicates order in the buddy
                         * system if PG_buddy is set.
                         * 私有数据指针，由应用场景确定其具体的含义
                         */
#if USE_SPLIT_PTE_PTLOCKS
#if ALLOC_SPLIT_PTLOCKS
        spinlock_t *ptl;
#else
        spinlock_t ptl;
#endif
#endif
        struct kmem_cache *slab_cache;  /* SL[AU]B: Pointer to slab */
    };
 
#ifdef CONFIG_MEMCG
    struct mem_cgroup *mem_cgroup;
#endif
 
    /*
     * On machines where all RAM is mapped into kernel address space,
     * we can simply calculate the virtual address. On machines with
     * highmem some memory is mapped into kernel virtual memory
     * dynamically, so we need a place to store that address.
     * Note that this field could be 16 bits on x86 ... ;)
     *
     * Architectures with slow multiplication can define
     * WANT_PAGE_VIRTUAL in asm/page.h
     */
#if defined(WANT_PAGE_VIRTUAL)
    void *virtual;          /* Kernel virtual address (NULL if
                       not kmapped, ie. highmem) */
#endif /* WANT_PAGE_VIRTUAL */
 
#ifdef CONFIG_KMEMCHECK
    /*
     * kmemcheck wants to track the status of each byte in a page; this
     * is a pointer to such a status block. NULL if not tracked.
     */
    void *shadow;
#endif
 
#ifdef LAST_CPUPID_NOT_IN_PAGE_FLAGS
    int _last_cpupid;
#endif
}
/*
 * The struct page can be forced to be double word aligned so that atomic ops
 * on double words work. The SLUB allocator can make use of such a feature.
 */
#ifdef CONFIG_HAVE_ALIGNED_STRUCT_PAGE
    __aligned(2 * sizeof(unsigned long))
#endif
;
```
