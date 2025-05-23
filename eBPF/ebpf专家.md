## 什么是ebpf

.What is ebpf
![[image-2025-03-02-17-43-38-568.png]]

1. 核心
- 11个64位的寄存器
- 一个大小为512字节的栈
- 一套64位的间接指令集

2. 支持ebpf运行的基础设施

- Verifier 验证器，内核中运行代码安全是重中之重，ebpf在真正运行之前都需要通过验证器检查，只有通过了检查的程序才运行运行。
- 即时编译器编译器，将ebpf字节码编译成能直接在底层运行的机器码

3. ebpf使用的数据结构

- Map ebpf使用的主要数据结构是Map，Map是驻留在内核空间的高效的键值仓库，内核空间和用户空间正是使用Map来贡献数据。

4. 辅助函数

- 辅助函数是一组内核提供的函数，ebpf程序可以调用这些函数，从而大大简化ebpf程序的开发难度。

eBPF 是一项安全、高效的通过在沙箱中运行程序以实现内核功能扩展的技术，是对传统的修改内核源代码和编写内核模块方式的革命性创新。eBPF 程序是事件驱动的，当内核或用户程序经过一个 eBPF Hook 时，对应 Hook 点上加载的 eBPF 程序就会被执行。Linux 内核中预定义了一系列常用的 Hook 点，你也可以利用 kprobe 和 uprobe 技术动态增加内核和应用程序的自定义 Hook 点。得益于 Just-in-Time (JIT) 技术，eBPF 代码的运行效率可媲美内核原生代码和内核模块。得益于 Verification 机制，eBPF 代码将会安全的运行，不会导致内核崩溃或进入死循环。

![[image-2025-03-03-11-45-56-424.png]]

[https://ebpf.io/what-is-ebpf/#hook-overview]

### ebpf User Interface

从内核3.18开始，内核提供了ebpf系统调用，以及暴露了 `uapi/linux/bpf.h` 头文件，见 https://elixir.bootlin.com/linux/v5.16.20/source/include/uapi/linux/bpf.h[bpf.h]

用户通过BPF系统调用可以加载BPF程序，创建和读写Map等等。

bpf系统调用的第一个参数为 cmd，用来指定操作类型(eg. BPF_MAP_CREATE, BPF_PROG_LOAD), 第二个参数uattr将ebpf的属性数据，从用户空间传递到内核空间。

```c
SYSCALL_DEFINE3(bpf, int, cmd, union bpf_attr __user*, uattr, unsigned int, size) {
    return __sys_bpf(cmd, USER_BPFPTR(uattr), size);
}

linux/arch/x86/entry/syscalls/syscall_64.tbl
321     common  bpf       sys_bpf
```

BPF系统调用会返回给用户空间一个文件描述符，用户可以通过该文件描述符将ebpf程序挂载到内核子系统，然后内核通过事件触发挂载的ebpf程序。

```c
fd=sys_bpf(BPF_PROG_LOAD, &attr, sizeof(attr));
```

.cmd参数类型
```c
enum bpf_cmd {
	BPF_MAP_CREATE,
	BPF_MAP_LOOKUP_ELEM,
	BPF_MAP_UPDATE_ELEM,
	BPF_MAP_DELETE_ELEM,
	BPF_MAP_GET_NEXT_KEY,
	BPF_PROG_LOAD,
	BPF_OBJ_PIN,
	BPF_OBJ_GET,
	BPF_PROG_ATTACH,
	BPF_PROG_DETACH,
	BPF_PROG_TEST_RUN,
	BPF_PROG_RUN = BPF_PROG_TEST_RUN,
	BPF_PROG_GET_NEXT_ID,
	BPF_MAP_GET_NEXT_ID,
	BPF_PROG_GET_FD_BY_ID,
	BPF_MAP_GET_FD_BY_ID,
	BPF_OBJ_GET_INFO_BY_FD,
	...
};
```

## ebpf是如何工作的

### ebpf Maps

ebpf Map是驻留在内核空间高效的键值仓库，用户空间可以通过ebpf系统调用创建map

```c
map_fd=sys_bpf(BPF_MAP_CREATE, &attr, sizeof(attr));
```

ebpf程序能够通过键值查找，更新或删除Map的元素。

```c
bpf_map_update_elem(&my_map, &loc, &init_val, BPF_ANY);
```

最后用户空间程序可以通过文件描述符来访问map元素，总的来说Map就是用来在内核和用户空间贡献数据的。

```c
bpf_map_lookup_elem(map_fd, key, values);
```

![[image-2025-03-02-22-23-37-189.png]]

































