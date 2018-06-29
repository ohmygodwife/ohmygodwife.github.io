---
layout: post
title: "glibc中的File结构"
date: 2018-06-29 15:17:49 +0800
description: ""
category: ctf/bin
tags: []
---

程序执行fopen等函数时会创建FILE结构，并分配在堆中，我们常定义一个指向FILE结构的指针来接收这个返回值。进程中的FILE结构会通过\_chain域彼此连接形成一个单向链表，链表头部用全局变量\_IO_list_all表示，通过这个值我们可以遍历所有的FILE结构，新加一个FILE节点会从链表头插入，同时更新链表头为这个新节点。

在标准I/O库中，每个程序启动时有三个文件流是自动打开的：stdin、stdout、stderr。因此在初始状态下，\_IO_list_all指向了一个有这些文件流构成的链表（\_IO_list_all=**stderr->stdout->stdin**）。我们可以在libc.so中找到stdin\stdout\stderr等符号，这些符号是指向FILE结构的指针，真正结构的符号是

\_IO_2_1_stderr\_/\_IO_2_1_stdout\_/\_IO_2_1_stdin\_

比如在gdb中查看stdin地址：

```assembly
pwndbg>print &IO_2_1_stdin
$2 = (struct IO_FILE_plus *) 0x7ffff7dd18e0 <IO_2_1_stdin_>
```

注意：这三个文件流位于libc.so的数据段，而我们使用fopen创建的文件流是分配在堆内存上的。

## FILE \*fopen(char *filename, *type)

```c
//./libio/iofopen.c
FILE* __fopen_internal (const char *filename, const char *mode, int is32)
{
  struct locked_FILE
  {
    struct _IO_FILE_plus fp; //指向_IO_FILE_plus指针
#ifdef _IO_MTSAFE_IO
    _IO_lock_t lock;
#endif
    struct _IO_wide_data wd;
  } *new_f = (struct locked_FILE *) malloc (sizeof (struct locked_FILE));
  ...
  //libio/libioP.h:#define _IO_JUMPS(THIS) (THIS)->vtable
  IO_JUMPS (&new_f->fp) = &IO_file_jumps; //为创建的FILE初始化vtable
  _IO_file_init (&new_f->fp);
  if (_IO_file_fopen ((FILE *) new_f, filename, mode, is32) != NULL)
    return __fopen_maybe_mmap (&new_f->fp.file);
```

1. 在fopen对应的函数\_\_fopen_internal内部会调用malloc函数，分配locked_FILE（包含_IO_FILE_plus包含FILE）结构的空间。因此我们可以获知FILE结构是存储在堆上的
2. 为创建的FILE初始化vtable
3. 调用\_IO_file_init把新分配的FILE链入\_IO_list_all为起始的FILE链表中
4. 调用\_IO_file_fopen函数打开目标文件，_IO_file_fopen会根据用户传入的打开模式进行打开操作，最后调用系统接口open函数

```c
./libio/libioP.h 
struct _IO_FILE_plus
{
  FILE file; //./libio/bits/types/FILE.h:typedef struct _IO_FILE FILE;
  const struct _IO_jump_t *vtable; //sizeof(FILE)=216=0xd8, FILE* + 0xd8 to access vtable
};

./libio/bits/types/struct_FILE.h
struct _IO_FILE
{
  int _flags;           /* High-order word is _IO_MAGIC; rest is flags. */
  /* The following pointers correspond to the C++ streambuf protocol. */
  char _IO_read_ptr;   / Current read pointer */
  char _IO_read_end;   / End of get area. */
  char _IO_read_base;  / Start of putback+get area. */
  char _IO_write_base; / Start of put area. */
  char _IO_write_ptr;  / Current put pointer. */
  char _IO_write_end;  / End of put area. */
  char _IO_buf_base;   / Start of reserve area. */
  char _IO_buf_end;    / End of reserve area. */
  /* The following fields are used to support backing up and undo. */
  char _IO_save_base; / Pointer to start of non-current get area. */
  char _IO_backup_base;  / Pointer to first valid character of backup area */
  char _IO_save_end; / Pointer to end of non-current get area. */
  struct IO_marker *markers;
  struct IO_FILE *chain;
  int _fileno;
  int _flags2;
  __off_t _old_offset; /* This used to be _offset but it's too small.  */
  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];
  IO_lock_t *lock;
#ifdef _IO_USE_OLD_IO_FILE // undefined, so would be complete struct
};
struct _IO_FILE_complete
{
  struct _IO_FILE _file;
#endif
#if defined _G_IO_IO_FILE_VERSION && _G_IO_IO_FILE_VERSION == 0x20001
  _IO_off64_t _offset;
# if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T
  /* Wide character stream stuff.  */
  struct IO_codecvt *codecvt;
  struct IO_wide_data *wide_data;
  struct IO_FILE *freeres_list;
  void *_freeres_buf;
# else
  void *__pad1;
  void *__pad2;
  void *__pad3;
  void *__pad4;
# endif
  size_t __pad5;
  int _mode;
  /* Make sure we don't get into trouble again.  */
  char _unused2[15 * sizeof (int) - 4 * sizeof (void *) - sizeof (size_t)];
#endif
};

```

由于**sizeof(FILE)=216=0xd8**，因此通过fopen返回的FILE指针+0xd8即可以访问_IO_FILE_plus中的vtable：

```c
./libio/libioP.h
struct _IO_jump_t
{
    JUMP_FIELD(size_t, __dummy);
    JUMP_FIELD(size_t, __dummy2);
    JUMP_FIELD(_IO_finish_t, __finish);
    JUMP_FIELD(_IO_overflow_t, __overflow);
    JUMP_FIELD(_IO_underflow_t, __underflow);
    JUMP_FIELD(_IO_underflow_t, __uflow);
    JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
    /* showmany */
    JUMP_FIELD(_IO_xsputn_t, __xsputn);
    JUMP_FIELD(_IO_xsgetn_t, __xsgetn);
    JUMP_FIELD(_IO_seekoff_t, __seekoff);
    JUMP_FIELD(_IO_seekpos_t, __seekpos);
    JUMP_FIELD(_IO_setbuf_t, __setbuf);
    JUMP_FIELD(_IO_sync_t, __sync);
    JUMP_FIELD(_IO_doallocate_t, __doallocate);
    JUMP_FIELD(_IO_read_t, __read);
    JUMP_FIELD(_IO_write_t, __write);
    JUMP_FIELD(_IO_seek_t, __seek);
    JUMP_FIELD(_IO_close_t, __close);
    JUMP_FIELD(_IO_stat_t, __stat);
    JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
    JUMP_FIELD(_IO_imbue_t, __imbue);
};

./libio/fileops.c
const struct _IO_jump_t _IO_file_jumps libio_vtable =
{
  //libio/libioP.h#define JUMP_INIT_DUMMY JUMP_INIT(dummy, 0), JUMP_INIT (dummy2, 0)
  //libio/libioP.h:#define JUMP_INIT(NAME, VALUE) VALUE
  0,1 JUMP_INIT_DUMMY, // 2 dummy field
  2 JUMP_INIT(finish, _IO_file_finish),
  3 JUMP_INIT(overflow, _IO_file_overflow),
  4 JUMP_INIT(underflow, _IO_file_underflow),
  5 JUMP_INIT(uflow, _IO_default_uflow),
  6 JUMP_INIT(pbackfail, _IO_default_pbackfail),
  7 JUMP_INIT(xsputn, _IO_file_xsputn),
  8 JUMP_INIT(xsgetn, _IO_file_xsgetn),
  9 JUMP_INIT(seekoff, _IO_new_file_seekoff),
  10 JUMP_INIT(seekpos, _IO_default_seekpos),
  11 JUMP_INIT(setbuf, _IO_new_file_setbuf),
  12 JUMP_INIT(sync, _IO_new_file_sync),
  13 JUMP_INIT(doallocate, _IO_file_doallocate),
  14 JUMP_INIT(read, _IO_file_read),
  15 JUMP_INIT(write, _IO_new_file_write),
  16 JUMP_INIT(seek, _IO_file_seek),
  17 JUMP_INIT(close, _IO_file_close),
  18 JUMP_INIT(stat, _IO_file_stat),
  19 JUMP_INIT(showmanyc, _IO_default_showmanyc),
  20 JUMP_INIT(imbue, _IO_default_imbue)
};
```

vtable保存了一系列函数指针，stdio函数会调用这些函数指针进行相应的操作。

## size_t fread ( void \*buffer, size_t size, size_t count, FILE *stream)

```c
./libio/iofread.c:
size_t _IO_fread (void *buf, size_t size, size_t count, FILE *fp)
{
  bytes_read = _IO_sgetn (fp, (char *) buf, bytes_requested);

./libio/genops.c:
size_t _IO_sgetn (FILE *fp, void *data, size_t n)
{
  /* FIXME handle putback buffer here! */
  return _IO_XSGETN (fp, data, n);
}

libio/libioP.h:
#define _IO_XSGETN(FP, DATA, N) JUMP2 (__xsgetn, FP, DATA, N)
typedef size_t (*_IO_xsgetn_t) (FILE *FP, void *DATA, size_t N);
```

\_IO_XSGETN是\_IO_FILE_plus.vtable中的xsgetn函数指针（默认是_IO_file_xsgetn），index=8，并且，FILE指针fp会作为第一个参数传递给该函数指针。

```c
libio/fileops.c:
size_t _IO_file_xsgetn (FILE *fp, void *data, size_t n)
{
  count = _IO_SYSREAD (fp, s, count);

libio/libioP.h:
#define _IO_SYSREAD(FP, DATA, LEN) JUMP2 (__read, FP, DATA, LEN)
typedef ssize_t (*_IO_read_t) (FILE *, void *, ssize_t);
```

\_IO_file_xsgetn会调用vtable中的\_IO_read（默认是_IO_file_read），index=14，并且，FILE指针fp会作为第一个参数传递给该函数指针。最终会调用系统接口read函数：

```c
./libio/fileops.c:
ssize_t _IO_file_read (FILE *fp, void *buf, ssize_t size)
{
  return (_builtin_expect (fp->flags2 & _IO_FLAGS2_NOTCANCEL, 0)
          ? _read_nocancel (fp->fileno, buf, size)
          : _read (fp->fileno, buf, size));
}
```

因此fread对于调用vtable的顺序为：

**8 JUMP_INIT(xsgetn, _IO_file_xsgetn)**

**14 JUMP_INIT(read, _IO_file_read)**

## size_t fwrite(const void\* buffer, size_t size, size_t count, FILE* stream)

```c
./libio/iofwrite.c:
size_t _IO_fwrite (const void *buf, size_t size, size_t count, FILE *fp)
{
  written = _IO_sputn (fp, (const char *) buf, request);

libio/libioP.h:
#define _IO_sputn(fp, s, n) _IO_XSPUTN (fp, s, n)
#define _IO_XSPUTN(FP, DATA, N) JUMP2 (__xsputn, FP, DATA, N)
typedef size_t (*_IO_xsputn_t) (FILE *FP, const void *DATA, size_t N);
```

\_IO_XSPUTN是\_IO_FILE_plus.vtable中的xsputn函数指针（默认是_IO_new_file_xsputn），index=7，并且，FILE指针fp会作为第一个参数传递给该函数指针。

```c
./libio/fileops.c
size_t _IO_new_file_xsputn (FILE *f, const void *data, size_t n)
{
  /* Next flush the (full) buffer. */
  if (_IO_OVERFLOW (f, EOF) == EOF)

libc_hidden_ver (_IO_new_file_xsputn, _IO_file_xsputn)
libc_hidden_ver (_IO_new_file_overflow, _IO_file_overflow)

libio/libioP.h:
#define _IO_OVERFLOW(FP, CH) JUMP1 (__overflow, FP, CH)
typedef int (*_IO_overflow_t) (FILE *, int);
```

由于\_IO_new_file_xsputn会调用vtable中的\_IO_overflow（默认是_IO_new_file_overflow），index=3，并且，FILE指针fp会作为第一个参数传递给该函数指针。

```c
./libio/fileops.c:
int _IO_new_file_overflow (FILE *f, int ch)
{
  return IO_do_write (f, f->IO_write_base,
                         f->IO_write_ptr - f->IO_write_base);

versioned_symbol (libc, _IO_new_do_write, _IO_do_write, GLIBC_2_1)

int _IO_new_do_write (FILE *fp, const char *data, size_t to_do)
{
  return (to_do == 0
          || (size_t) new_do_write (fp, data, to_do) == to_do) ? 0 : EOF;
}

static size_t new_do_write (FILE *fp, const char *data, size_t to_do)
{
  count = _IO_SYSWRITE (fp, data, to_do);

libio/libioP.h:
#define _IO_SYSWRITE(FP, DATA, LEN) JUMP2 (__write, FP, DATA, LEN)
typedef ssize_t (*_IO_write_t) (FILE *, const void *, ssize_t)
```

\_IO_new_file_overflow会调用vtable中的\_IO_write（默认是_IO_new_file_write），index=15，并且，FILE指针fp会作为第一个参数传递给该函数指针。最终会调用系统接口write函数：

```c
./libio/fileops.c:
ssize_t _IO_new_file_write (FILE *f, const void *data, ssize_t n)
{
  ssize_t count = (_builtin_expect (f->flags2
                   & _IO_FLAGS2_NOTCANCEL, 0)
                   ? _write_nocancel (f->fileno, data, to_do)
                   : _write (f->fileno, data, to_do));
```

## int printf (const char \* format, ...)/int puts (const char * str)

```c
./libio/ioputs.c:
int _IO_puts (const char *str)
{
  if ((IO_vtable_offset (IO_stdout) != 0
       || IO_fwide (IO_stdout, -1) == -1)
      && IO_sputn (IO_stdout, str, len) == len
      && _IO_putc_unlocked ('\n', _IO_stdout) != EOF)
```

printf和puts是常用的输出函数，在printf的参数是以'\n'结束的纯字符串时，printf会被优化为puts函数并去除换行符。

puts在源码中实现的函数是\_IO_puts，这个函数的操作与fwrite的流程大致相同，可见，同样是调用\_IO_sputn即调用vtable中的xsputn函数指针（默认是_IO_new_file_xsputn），index=7，并且，FILE指针fp会作为第一个参数传递给该函数指针。最终会调用系统接口write函数。

如printf的调用栈为：

vfprintf+11 -> \_IO_file_xsputn ->_IO_file_overflow -> funlockfile -> _IO_file_write -> write

因此fwrite/printf/puts对于调用vtable的顺序为：

**7 JUMP_INIT(xsputn, _IO_file_xsputn)**

**3 JUMP_INIT(overflow, _IO_file_overflow)**

**15 JUMP_INIT(write, _IO_new_file_write)**

## int fclose(FILE \*stream)

```c
./libio/iofclose.c:
int _IO_new_fclose (FILE *fp)
{
  IO_un_link ((struct _IO_FILE_plus ) fp); //将FILE从chain链表中删除
  status = _IO_file_close_it (fp);
  _IO_FINISH (fp);
  if (fp != _IO_stdin && fp != _IO_stdout && fp != _IO_stderr)
  {
    fp->_flags = 0;
    free(fp);
  }

./libio/fileops.c
int _IO_new_file_close_it (FILE *fp)
{
  int close_status = ((fp->_flags2 & _IO_FLAGS2_NOCLOSE) == 0
                      ? _IO_SYSCLOSE (fp) : 0);

libc_hidden_ver (_IO_new_file_close_it, _IO_file_close_it)

libio/libioP.h:
#define _IO_SYSCLOSE(FP) JUMP0 (__close, FP)
typedef int (*_IO_close_t) (FILE ); / finalize */
#define _IO_FINISH(FP) JUMP1 (__finish, FP, 0)
typedef void (*_IO_finish_t) (FILE , int); / finalize */
```

fclose依次调用vtable中的\_IO_close指针，index=17和_IO_finish指针，index=2。最终会调用系统调用close函数。

```c
./libio/fileops.c:  
int _IO_file_close (FILE *fp)
{
  /* Cancelling close should be avoided if possible since it leaves an
     unrecoverable state behind.  */
  return _close_nocancel (fp->fileno);
}

void _IO_new_file_finish (FILE *fp, int dummy)
{
  if (_IO_file_is_open (fp))
    {
      _IO_do_flush (fp);
      if (!(fp->_flags & _IO_DELETE_DONT_CLOSE))
        _IO_SYSCLOSE (fp);
    }
  _IO_default_finish (fp, 0);
}

libc_hidden_ver (_IO_new_file_finish, _IO_file_finish)

./sysdeps/generic/not-cancel.h:#define close_nocancel(fd) close (fd)
```

因此fclose对于调用vtable的顺序为：

**17 JUMP_INIT(close, _IO_file_close)**

**2 JUMP_INIT(finish, _IO_file_finish)**