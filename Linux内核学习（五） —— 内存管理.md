# Linux内核学习（五） —— 内存管理

## 页

内存管理单元（MMU，管理内存并把虚拟地址转换成物理地址的硬件）通常以页为单位进行处理。因此，`内核把页作为内存管理的基本单位`。

```c
struct page {
    unsigned long flags;    // 存放页的状态
    atomic_t _count;        // 存放页的引用计数，-1表示内核没有引用这一页
    atomic_t _mapcount;     
    unsigned long private;
    struct address_space *mapping;  // 当页作为页缓存使用时，mapping域指向和这个页关联的address_space对象
    pgoff_t index;
    struct list_head lru;
    void *virtual;          // 页的虚拟地址，也就是页在虚拟内存中的地址
}
```

## 区

内核中将页划分为不同的区（zone），从而对具有相似特性的页进行分组。

+ ZONE_DMA——这个区包含的页能用来执行DMA操作（直接内存访问）。
+ ZONE_DMA32——与前者类似，但只能被32位设备访问。
+ ZONE_NORMAL——这个区包含的都是能正常映射的页。
+ ZONE_HIGHEM——这个区包含“高端内存”，其中的页并不能永久地映射到内核地址空间。

Linux把页划分为区，形成不同的内存池，方便根据用途进行分配。但值得注意的是，区的划分没有任何物理意义，只不过是内核为了管理页而采取的一种逻辑上的分组。

```c
struct zone {
	
	unsigned long watermark[NR_WMARK];

	unsigned long nr_reserved_highatomic;

	long lowmem_reserve[MAX_NR_ZONES];

	struct pglist_data	*zone_pgdat;
	struct per_cpu_pageset __percpu *pageset;

	unsigned long		zone_start_pfn;

	unsigned long		managed_pages;
	unsigned long		spanned_pages;
	unsigned long		present_pages;

	const char		*name;

	int initialized;

	/* Write-intensive fields used from the page allocator */
	ZONE_PADDING(_pad1_)

	/* free areas of different sizes */
	struct free_area	free_area[MAX_ORDER];

	/* zone flags, see below */
	unsigned long		flags;

	/* Primarily protects free_area */
	spinlock_t		lock;

	/* Write-intensive fields used by compaction and vmstats. */
	ZONE_PADDING(_pad2_)

	/*
	 * When free pages are below this point, additional steps are taken
	 * when reading the number of free pages to avoid per-cpu counter
	 * drift allowing watermarks to be breached
	 */
	unsigned long percpu_drift_mark;

	bool			contiguous;

	ZONE_PADDING(_pad3_)
	/* Zone statistics */
	atomic_long_t		vm_stat[NR_VM_ZONE_STAT_ITEMS];
} ____cacheline_internodealigned_in_smp;
```

其中`lock`域是一个自旋锁，它防止该结构被并发。值得注意的是，这个域只保护结构，而不保护驻留在这个区中的所有页。

`watermark`数组持有该区的最小值、最低和最高水位值。内核使用水位为每个内存区设置合适的内存消耗基准。该水位随空闲内存的多少而变化.

`name`域是一个以NULL结束的字符串，表示这个区的名字。有三个可能值，**“DMA”**，**“Normal”**和**“HighMem”**。

## 获得页

`struct page * alloc_pages(gfp_t gfp_mask, unsigned int order)`

该函数分配`2^order`（1<<order）个连续的物理页，并返回一个指针，该指针指向第一个页的page结构体。

`void * page_address(struct page *page)`

该函数能把给定的页转换成它的逻辑地址。

## 释放页

```c
void __free_pages(struct page *page, unsigned int order);
void free_pages(unsigned long addr, unsigned int order);
void free_page(unsigned long addr);
```

这些函数可以让你在不需要页的时候释放它们。

## gfp_mask标志

+ 行为修饰符——表示内核应当如何分配所需的内存
+ 区修饰符——表示从哪儿分配内存
+ 类型——组合了行为修饰符和区修饰符进行归类

## kmalloc()和vmalloc()

### kmalloc()

kmalloc()函数确保页在物理地址上是连续的，同时虚拟地址自然也是连续的。

### vmalloc()

vmalloc()函数只确保页在虚拟地址空间内是连续的，它通过分配非连续的物理内存块，再修正页表，把内存映射到逻辑地址空间的连续空间中。


