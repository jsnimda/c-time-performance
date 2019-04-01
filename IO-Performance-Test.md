IO Performance Test
===
Test setup:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define N 2000000

int main() {
    // setup
    for(int i=0; i<N; ++i) { 
        // code
    }
    return 0;
}
```
```
Empty: 0.006s
```
`gcc c.c -o c; time ./c`

`gcc c.c -o c; time cat case | ./c`

Read Integers
---
`scani` is ~5 times faster than `scanf`.
```c
scanf:
    int c;
    {
        scanf("%d", &c);
    }
    ~0.253s
scani:
    init_buf(&inbuf, N);
    int c;
    {
        c = scani(&inbuf);
    }
    ~0.051s
```
### case
**gen.c** : `gcc gen.c -o gen; ./gen > case`
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define N 2000000

int main() {
    for(int i=0; i<N; ++i) { 
        printf("%d ", i+1);
    }
    return 0;
}
```
Print Integers
---
`printi` is ~3 times faster than `printf`.
`gcc c.c -o c; time ./c > /dev/null`
```
printf:
    {
        printf("%d", i+1);
    }
    ~0.333s
printi:
    init_buf(&inbuf, N);
    {
        printi(&outbuf, i+1);
    }
    pflush(&outbuf);
    ~0.111s
printf + space:
    {
        printf("%d ", i+1);
    }
    ~0.372s
printi + space:
    init_buf(&inbuf, N);
    {
        printi(&outbuf, i+1);
        printc(&outbuf, ' ');
    }
    pflush(&outbuf);
    ~0.125s
```
