---
title: "HTB: Impossible Password"
category: hacking linux
tags: htb sre hacking elf
---

> **Clue**: Are you able to cheat me and get the flag?

In this post, I take a look at the Hack the Box challenge _Impossible Password_. I also briefly discuss the ELF format and break down some of the execution flow in a Linux binary.

Inside the archive is a single file, _impossible_password.bin_.

Running _file_ on the file, we see this is a ELF linux binary - we came across ELF in [Baby RE](https://benjamin.software/hacking/HTB-Baby-RE/) too:

```bash
> file impossible_password.bin
impossible_password.bin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=ba116ba1912a8c3779ddeb579404e2fdf34b1568, stripped
```

## What is ELF?

Whilst I'm writing this post during the Christmas holidays, I'm afraid to say this might not be what you are expecting. 

Executable and Linkable Format is cross platform format for _[executable](https://www.wikiwand.com/en/Executable) files, [object code](https://www.wikiwand.com/en/Object_code "Object code"), [shared libraries](https://www.wikiwand.com/en/Library_(computing)), and [core dumps](https://www.wikiwand.com/en/Core_dump)_. 

It essentially provides a way to package up executable objects and other related object files in a standardised way.

As far as I understand it, the format was born out of the complications of managing shared libraries in large packages.

It makes things simpler by simultaneously documenting both program linking and program execution in a single format and it does this with the concepts of sections and segments. 

For a great in-depth look at ELF, check out stacksmashing's [video](https://www.youtube.com/watch?v=nC1U1LJQL8o).

### What does the ELF header tell us?

By running _readelf_ on the executable we start to get a sense of the kinds of things stored in the header and what the operating system might make use of before attempting to execute. With the _-a_ option, you will also get a list of the sections and segments in your file.

```bash
> readelf -a impossible_password.bin
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x4006a0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          4512 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         28
  Section header string table index: 27
...
```

## Analysis

Opening the file in _Ghidra_, I started by looking at the symbol tree and navigated to _entry_. Which takes me to where the execution begins.

```c++
void entry(undefined8 param_1,undefined8 param_2,undefined8 param_3)

{
  undefined8 in_stack_00000000;
  undefined auStack8 [8];
  
  __libc_start_main(main,in_stack_00000000,&stack0x00000008,FUN_004009e0,FUN_00400a50,param_3,
                    auStack8);
  do {
                    /* WARNING: Do nothing block with infinite loop */
  } while( true );
}
```

```assembly
                             //
                             // .text 
                             // SHT_PROGBITS  [0x4006a0 - 0x400a53]
                             // ram:004006a0-ram:00400a53
                             //
                             **************************************************************
                             *                          FUNCTION                          *
                             **************************************************************
                             undefined entry()
             undefined         AL:1           <RETURN>
             undefined8        Stack[-0x10]:8 local_10                                XREF[1]:     004006ae(*)  
                             entry                                           XREF[5]:     Entry Point(*), 00400018(*), 
                                                                                          00400aa8, 00400af8(*), 
                                                                                          _elfSectionHeaders::00000350(*)  
        004006a0 31 ed           XOR        EBP,EBP
        004006a2 49 89 d1        MOV        R9,RDX
        004006a5 5e              POP        RSI
        004006a6 48 89 e2        MOV        RDX,RSP
        004006a9 48 83 e4 f0     AND        RSP,-0x10
        004006ad 50              PUSH       RAX
        004006ae 54              PUSH       RSP=>local_10
        004006af 49 c7 c0        MOV        R8=>FUN_00400a50,FUN_00400a50
                 50 0a 40 00
        004006b6 48 c7 c1        MOV        RCX=>FUN_004009e0,FUN_004009e0
                 e0 09 40 00
        004006bd 48 c7 c7        MOV        RDI=>main,main
                 5d 08 40 00
        004006c4 e8 47 ff        CALL       __libc_start_main                                undefined __libc_start_main()
                 ff ff
```

If you're familiar with this challenge, you may notice I have already renamed some of the functions to help make sense of what's going on. 

In the assembly view directly above, you see _main_ being moved into the RDI general purpose register as part of the stack setup for ___libc_start_main_.

If you now look up to the first code block, you'll see this reflected in the decompiled view of ___libc_start_main_. It calls a function that I named _main_ - as you would expect it to.

### _main()_

Again, the following shows the decompiled code after my renaming of variables and function calls from the _main_ function. I've done this to try explain what's going on with fewer code blocks.

```c++
void main(void)

{
  int user_input_matches_generated_key;
  char *generated_key;
  byte flag_var [20];
  char user_input [20];
  int user_input_matches_key;
  char *key;
  
  key = "SuperSeKretKey";
  flag_var[0] = 0x41;
  flag_var[1] = 0x5d;
  flag_var[2] = 0x4b;
  flag_var[3] = 0x72;
  flag_var[4] = 0x3d;
  flag_var[5] = 0x39;
  flag_var[6] = 0x6b;
  flag_var[7] = 0x30;
  flag_var[8] = 0x3d;
  flag_var[9] = 0x30;
  flag_var[10] = 0x6f;
  flag_var[11] = 0x30;
  flag_var[12] = 0x3b;
  flag_var[13] = 0x6b;
  flag_var[14] = 0x31;
  flag_var[15] = 0x3f;
  flag_var[16] = 0x6b;
  flag_var[17] = 0x38;
  flag_var[18] = 0x31;
  flag_var[19] = 0x74;
  printf("* ");
  __isoc99_scanf(&DAT_00400a82,user_input);
  printf("[%s]\n",user_input);
  user_input_matches_key = strcmp(user_input,key);
  if (user_input_matches_key != 0) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  printf("** ");
  __isoc99_scanf(&DAT_00400a82,user_input);
  generated_key = (char *)generate_key(0x14);
  user_input_matches_generated_key = strcmp(user_input,generated_key);
  if (user_input_matches_generated_key == 0) {
    get_flag(flag_var);
  }
  return;
}
```

#### Stage 1 - Super SeKret

Immediately I could see the string _SuperSekretKey_, which I renamed to _key_.

The assignment of various values to local variables looked a lot like values being added to a char array of length 20, so I retyped this and renamed the variable as so. In Ghidra this is very easy to do, just by right clicking on the variable.

Below this section, there is a scanf call, which I assumed was being assigned from _user_input_.

There is then a call to _strcmp_ which compares _user_input_ and the _key_, to see if they're equal. In the context of the program, we just need to enter _SuperSekretKey_ and we'll move past this code and into the next section.

#### Stage 2 - Key Generation

```c++
void * generate_key(int length_of_key)

{
  int iVar1;
  time_t tVar2;
  void *pvVar3;
  int local_c;
  
  tVar2 = time((time_t *)0x0);
  DAT_00601074 = DAT_00601074 + 1;
  srand(DAT_00601074 + (int)tVar2 * length_of_key);
  pvVar3 = malloc((long)(length_of_key + 1));
  if (pvVar3 != (void *)0x0) {
    local_c = 0;
    while (local_c < length_of_key) {
      iVar1 = rand();
      *(char *)((long)local_c + (long)pvVar3) = (char)(iVar1 % 0x5e) + '!';
      local_c = local_c + 1;
    }
    *(undefined *)((long)pvVar3 + (long)length_of_key) = 0;
    return pvVar3;
  }
                    /* WARNING: Subroutine does not return */
  exit(1);
}
```

In the next section, following _**_ being printed, there is another call to _scanf_, for additional user input.

There is then a call to a function which I've named _generate_key_, which takes an int and returns a value of that length to be compared with the user_input. 

If the two match, the flag appears to be returned.

Spending a moment to eyeball the code here, it looks slight convoluted and when we look back at the _main_ function, whilst it is important in reaching the flag, it doesn't have any bearing on the contents of the fag. 

So might it be possible to determine how the flag is generated and simply generate it ourselves?

#### Stage 3 - CTF

I've renamed the function _get_flag_ and it looks like this:

```c++
void get_flag(byte *flag_array)

{
  int i;
  byte *flag_array_;
  
  i = 0;
  flag_array_ = flag_array;
  while ((*flag_array_ != 9 && (i < 0x14))) {
    putchar((int)(char)(*flag_array_ ^ 9));
    flag_array_ = flag_array_ + 1;
    i = i + 1;
  }
  putchar(10);
  return;
}
```
I've also renamed a couple of other things here, something I determined to be an index, which I've named _i_. I've also changed the function parameter to _flag_array_ and the variable it's assigned to as _flag_array__.

If we look back at _main_, we can see the char array _flag_array_ contains the following characters: _A]Kr=9k0=0o0;k1?k81t_

The _get_flag_ function is quite simple and just moves through this array and applies _xor 9_ on each character. It does this until the end of the array, or it comes across an ascii horizontal tab (9).

To replicate this code in Python, we can copy the characters into a list, apply the xor on each character and concatenate the list entries into a single string.

```python
>>> flag = ['A',']','K','r','=','9','k','0','=','0','o','0',';','k','1','?','k','8','1','t']
>>> "".join([chr(ord(i) ^ 9) for i in flag])
'HTB{...}'
```

