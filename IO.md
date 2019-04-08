IO
===
`inline` : Haven't tested yet, you might add `static inline` to all the functions.
### Contents
- [Includes](#Includes)
- [Setup](#Setup)
  ```
  init_buf
  ```
  - [Main](#Main)
- [Read Integers](#Read-Integers)
  ```
  my_atoi
  reads
  scani
  ```
- [Print Integers](#Print-Integers)
  ```
  my_itoa
  pflush
  printi
  printc
  printisp
  printispc
  ```
- [Read Line](#Read-Line)
  ```
  scanln
  scanlnn
  skipln
  skipsp
  ```
Includes
---
```c 
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
```

Setup
---
Put at global scope.
```c 
struct Iobuf {
    char* buf;
    int capacity; // available space of buf
    int size; // scanned data size (!= capacity when eof)
    int ptr;
    int eof;
};
struct Iobuf* init_buf(struct Iobuf* b, int size){
    b->buf = malloc(size+1); //+1 for safety
    b->buf[size] = 0;
    b->capacity = size;
    b->size = 0;
    b->ptr = 0;
    b->eof = 0;
    return b;
}
struct Iobuf inbuf;
struct Iobuf outbuf;
```
`ULLONG_MAX` = `2^64-1` = `18446744073709551615` (20 digits)  
`ULONG_MAX` = `2^32-1` = `4294967295` (10 digits)  
Suggested minimum size should be > 50.  
### Main
```c
int main() {
    init_buf(&inbuf, N);
    init_buf(&outbuf, M);
    // code
}
```

Read Integers
---
Read non-negative integers `type int32` by `scani()`.
```c
int my_atoi(char *str, int *out){ 
    int val = 0;
    char *i = str;
    while(*str && (*str < '0' || *str > '9')){
        ++str;
    }
    if(*str == 0){
        return -1; // the string str does not contain valid numer
    }
    do {
        val = val*10 + (*(str++) - '0');
    } while (*str >= '0' && *str <= '9');
    (*out) = val;
    return str - i; //return the count of chars this func had read
}
int reads(struct Iobuf* inbuf){
    if(inbuf->eof) return -1; // already eof
    int remaining = inbuf->size - inbuf->ptr;
    memcpy(inbuf->buf, inbuf->buf + inbuf->ptr, remaining);
    int count = fread(inbuf->buf + remaining, 1, inbuf->capacity - remaining, stdin);
    inbuf->eof = feof(stdin);
    inbuf->size = remaining + count;
    inbuf->buf[inbuf->size] = 0; //terminal
    inbuf->ptr = 0;
    return count;
}
int scani(struct Iobuf* inbuf){
    // return -1 if read fail
    int remaining = inbuf->size - inbuf->ptr;
    if(remaining < 10 && !inbuf->eof){
        reads(inbuf);
    }
    int val;
    int count = my_atoi(inbuf->buf + inbuf->ptr, &val);
    if(count == -1){
        return -1;
    }
    inbuf->ptr += count;
    return val;
}
```

Print Integers
---
Print non-negative integers `type int32` by `printi()`/`printisp()`/`printispc()`.  
**Important**: You should have a separated input and output buffer if you want to print during read.  
```c
char* my_itoa(int val, int *out_count){
    static char buf[12] = {0};
    int tmp;
    int i = 10;
    do {
        tmp = val;
        buf[i--] = (tmp - (val = val / 10) * 10) + '0';
    } while (val);
    (*out_count) = 10 - i;
    return buf + i+1;
}
void pflush(struct Iobuf* outbuf){
    fwrite(outbuf->buf + outbuf->ptr, 1, outbuf->size - outbuf->ptr, stdout);
    outbuf->ptr = 0;
    outbuf->size = 0;
}
void printi(struct Iobuf* outbuf, int num){
    int count;
    char* res = my_itoa(num, &count);
    int remaining = outbuf->capacity - outbuf->size;
    if(count > remaining){
        pflush(outbuf);
    }
    memcpy(outbuf->buf + outbuf->size, res, count);
    outbuf->size += count;
}
void printc(struct Iobuf* outbuf, char c){
    int remaining = outbuf->capacity - outbuf->size;
    if(1 > remaining){
        pflush(outbuf);
    }
    outbuf->buf[outbuf->size] = c;
    outbuf->size++;
}
void printisp(struct Iobuf* outbuf, int num){ // with space
    int count;
    char* res = my_itoa(num, &count);
    int remaining = outbuf->capacity - outbuf->size;
    if(count + 1 > remaining){
        pflush(outbuf);
    }
    memcpy(outbuf->buf + outbuf->size, res, count);
    outbuf->buf[outbuf->size + count] = ' ';
    outbuf->size += count + 1;
}
void printispc(struct Iobuf* outbuf, int num, char c){ // with char
    int count;
    char* res = my_itoa(num, &count);
    int remaining = outbuf->capacity - outbuf->size;
    if(count + 1 > remaining){
        pflush(outbuf);
    }
    memcpy(outbuf->buf + outbuf->size, res, count);
    outbuf->buf[outbuf->size + count] = c;
    outbuf->size += count + 1;
}
```

Read Line
---
additional ref: https://clc-wiki.net/wiki/memchr  
Please include function `reads()`.  
`\n` is included if scan is not stopped by eof or maximum num is reach.  
Please ensure outstr has size of at least `(num+1)` (for terminating null)
```c
int scanln(struct Iobuf* inbuf, char* outstr){
    // return count, jump to next char after next \n
    // return -1 if fail (outstr unchanged)
    int p = 0;
    int remaining = inbuf->size - inbuf->ptr;
    if(remaining == 0){
        if(inbuf->eof) return -1;
        reads(inbuf);
        remaining = inbuf->size - inbuf->ptr;
    }
    char * pch;
    do {
        pch = (char*) memchr(inbuf->buf + inbuf->ptr, '\n', remaining);
        if(pch != NULL){ // find \n
            int len = pch - (inbuf->buf + inbuf->ptr) + 1; // include \n
            memcpy(outstr + p, inbuf->buf + inbuf->ptr, len);
            inbuf->ptr += len;
            outstr[p += len] = 0;
            return p;
        }
        memcpy(outstr + p, inbuf->buf + inbuf->ptr, remaining);
        inbuf->ptr += remaining;
        p += remaining;
        reads(inbuf);
    } while ((remaining = inbuf->size - inbuf->ptr) != 0);
    // eof
    outstr[p] = 0;
    return p;
}
```
```c
int scanlnn(struct Iobuf* inbuf, char* outstr, int num){
    // return count
    int p = 0;
    int remaining = inbuf->size - inbuf->ptr;
    if(remaining == 0){
        if(inbuf->eof) return -1;
        reads(inbuf);
        remaining = inbuf->size - inbuf->ptr;
    }
    char * pch;
    do {
        pch = (char*) memchr(inbuf->buf + inbuf->ptr, '\n', remaining);
        if(pch != NULL){ // find \n
            int len = pch - (inbuf->buf + inbuf->ptr) + 1; // include \n
            if(p + len > num){
                len = num - p;
            }
            memcpy(outstr + p, inbuf->buf + inbuf->ptr, len);
            inbuf->ptr += len;
            outstr[p += len] = 0;
            return p;
        }
        if(p + remaining > num){
            remaining = num - p;
            memcpy(outstr + p, inbuf->buf + inbuf->ptr, remaining);
            inbuf->ptr += remaining;
            outstr[p += remaining] = 0;
            return p;
        }
        memcpy(outstr + p, inbuf->buf + inbuf->ptr, remaining);
        inbuf->ptr += remaining;
        p += remaining;
        reads(inbuf);
    } while ((remaining = inbuf->size - inbuf->ptr) != 0);
    // eof
    outstr[p] = 0;
    return p;
}
```
```c
int skipln(struct Iobuf* inbuf){
    // return count, jump to next char after next \n
    int p = 0;
    int remaining = inbuf->size - inbuf->ptr;
    if(remaining == 0){
        if(inbuf->eof) return -1;
        reads(inbuf);
        remaining = inbuf->size - inbuf->ptr;
    }
    char * pch;
    do {
        pch = (char*) memchr(inbuf->buf + inbuf->ptr, '\n', remaining);
        if(pch != NULL){ // find \n
            int len = pch - (inbuf->buf + inbuf->ptr) + 1; // include \n
            inbuf->ptr += len;
            p += len;
            return p;
        }
        inbuf->ptr += remaining;
        p += remaining;
        reads(inbuf);
    } while ((remaining = inbuf->size - inbuf->ptr) != 0);
    // eof
    return p;
}
```
```c
int skipsp(struct Iobuf* inbuf){
    // return count, jump to next char after last space of space span
    // " \t\n\v\f\r"
    int p = 0;
    int remaining = inbuf->size - inbuf->ptr;
    if(remaining == 0){
        if(inbuf->eof) return -1;
        reads(inbuf);
        remaining = inbuf->size - inbuf->ptr;
    }
    int spn;
    do {
        spn = strspn(inbuf->buf + inbuf->ptr, " \t\n\v\f\r");
        if(spn < remaining){
            inbuf->ptr += spn;
            p += spn;
            return p;
        }
        // all remaining is space....
        inbuf->ptr += remaining;
        p += remaining;
        reads(inbuf);
    } while ((remaining = inbuf->size - inbuf->ptr) != 0);
    return p;
}
```
Print Line
---

Performance Test
---
#### [IO Performance Test](IO-Performance-Test.md)
