---
title: "HTB: Baby RE"
category: hacking
tags: htb sre hacking
---

> **Clue**: Show us your basic skills! (P.S. There are 4 ways to solve this, are you willing to try them all?)

As with most [HTB](https://www.hackthebox.eu/) challenges, the first stage is to download the challenge archive and extract its content. The password for the archive is _hackthebox_.
<!--ex-->

## Dealing with the archive

Before extracting the archive, it's a good idea to get an idea of what you are about to extract. With a zip file, you can do the following:

```bash
$ unzip -l Baby_RE.zip

Archive:  Baby_RE.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
    16760  10-13-2019 09:19   baby
---------                     -------
    16760                     1 file
```

To extract the zip archive, I tried running:

```bash
$ unzip Baby_RE.zip
```

You might find you come across an error:

```bash
Archive:  Baby_RE.zip
   skipping: baby                    unsupported compression method 99
```

This is due to lack of AES support in unzip; you will need an alternate tool such as p7zip, which can be installed with [homebrew](https://brew.sh/) on a Mac.

To list the files in an archive with 7z, run:

```bash
$ 7z l archive.zip

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=utf8,Utf16=on,HugeFiles=on,64 bits,6 CPUs x64)

Scanning the drive for archives:
1 file, 2885 bytes (3 KiB)

Listing archive: Baby_RE.zip

--
Path = Baby_RE.zip
Type = zip
Physical Size = 2885

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2019-10-13 07:19:12 ....A        16760         2705  baby
------------------- ----- ------------ ------------  ------------------------
2019-10-13 07:19:12              16760         2705  1 files
```

To extract the archive, run:

```bash
$ 7z -phackthebox x Baby_RE.zip

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=utf8,Utf16=on,HugeFiles=on,64 bits,6 CPUs x64)

Scanning the drive for archives:
1 file, 2885 bytes (3 KiB)

Extracting archive: Baby_RE.zip
--
Path = Baby_RE.zip
Type = zip
Physical Size = 2885

Everything is Ok

Size:       16760
Compressed: 2885
```

Once extacted, the only content is a file called _baby_.

## The extracted file

Running the command _file_ on _baby_, we can see it's a Linux binary executable.

```bash
$ file baby
baby: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=25adc53b89f781335a27bf1b81f5c4cb74581022, for GNU/Linux 3.2.0, not stripped
```

If we inspect the strings in the binary, we come across some interesting examples. We have what looks like a partial flag and a potential key.

```bash
> strings
...
HTB{B4BYH
_R3V_TH4H
TS_Ef
[]A\A]A^A_
Dont run `strings` on this challenge, that is not the way!!!!
Insert key:
abcde122313
...
```

As we can see, the flag is split across a few lines and we can't guarantee the lines are entirely correct.

Taking note of the hint, here, perhaps using some static analysis could help. 

## Static analysis

Open the binary in your favourite disassembler and decompiler and navigate to the main function. 

Using Ghidra, we can see:

```c
undefined8 main(void)

{
  int iVar1;
  undefined8 local_48;
  undefined8 local_40;
  undefined4 local_38;
  undefined2 local_34;
  char local_28 [24];
  char *local_10;
  
  local_10 = "Dont run `strings` on this challenge, that is not the way!!!!";
  puts("Insert key: ");
  fgets(local_28,0x14,stdin);
  iVar1 = strcmp(local_28,"abcde122313\n");
  if (iVar1 == 0) {
    local_48 = 0x594234427b425448;
    local_40 = 0x3448545f5633525f;
    local_38 = 0x455f5354;
    local_34 = 0x7d5a;
    puts((char *)&local_48);
  }
  else {
    puts("Try again later.");
  }
  return 0;
}
```

This program is reading up to 20 bytes from stdin and storing it in a variable _local_28_. 

If this variable matches the string _abcde122313\n_ various hexadecimal values are assigned to local variables. We can guess these might refer to the flag.

The _puts_ call writes out the str pointed to by _local_48_ and continues to write out to stdout until it _reaches the terminating null character ('\0')_ - [puts docs](http://www.cplusplus.com/reference/cstdio/puts/). As these variables are local, they are written to a contiguous block of memory on the stack.

A quick way to get the flag here is to grab the values of variables local_48, local_40, local_38 and local_34, convert each to big endian and concatenate them.

```bash
LE -> BE

0x594234427b425448 -> 0x4854427b42344259
0x3448545f5633525f -> 0x5f5233565f544834
0x455f5354 -> 0x54535f45
0x7d5a -> 0x5a7d

Concatenated:

0x4854427b423442595f5233565f54483454535f455a7d

Hex -> Ascii

HTB{...}
```

Another way to get to the answer is to simply run the binary and pass in the key found with strings.

```bash
> ./baby
Insert key:
abcde122313
HTB{...}
```
