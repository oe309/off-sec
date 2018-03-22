# ANGSTORM CTF 2018: Number Guess

**Category:** Binary
**Points:** 70
**Description:**

> Ian loves playing number guessing games, so he went ahead and wrote one himself (source). I hope it doesn't have any vulns. The service is running at nc shell.angstromctf.com 1235.

## Write Up

by Ovais Ellahi

The challenge is provided by https://angstromctf.com/problems

We are given two files, an elf 64-bit executable:

```bash
$ file guessPublic64
guessPublic64: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=0b444932c543f786c9e9b95ef43415fa0564c0a6, not stripped
```

and a c source file guessPublic.c.

