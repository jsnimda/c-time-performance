IO
===
`inline` : Haven't tested yet, you might add `inline` to all the functions.

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
        val = val / 10;
        buf[i] = (tmp - val * 10) + '0';
        --i;
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

Performance Test
---
[IO Performance Test](IO-Performance-Test.md)
