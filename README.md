# RustyGenC
this is an assembly code translator to C language written in rust, it reads the assembly code from a specified file and displays the C pseudo code,


## architecture specification
the code is only for x86, and is specifically optimized for x64, although I plan to adapt it for other architectures later
nevertheless, it can still convert 32-bit assembly code, but it will give something like this: 

**assembly code :** 
```asm
formats db 'rdm print: %d', 0

call main

main:
  mov eax, 32
  shl eax, 20
  xor eax, ebx
  mov edx, 10
  mov ecx, 10
  call random_func
  ret
```
**language C :**
```c
#include <stdio.h>
#include <stdint.h>
uint8_t formats[14] = {'r', 'd', 'm', ' ', 'p', 'r', 'i', 'n', 't', ':', ' ', '%', 'd', 0};
main();

void main(){
    uint64_t rax;
    uint64_t rdx;
    uint64_t rcx;
    *(uint32_t*)&rax = 32;
    *(uint32_t*)&rax <<= 20;
    *(uint32_t*)&rax ^= (uint32_t)rbx;
    *(uint32_t*)&rdx = 10;
    *(uint32_t*)&rcx = 10;
    random_func(rcx, rdx);
    return;
}
```
we do "* (uint32_t *)" because the 32bit registers represent the low order 32bits of the 64bit registers, and we are therefore trying to reproduce the fact of modifying the low order 32bits

## Usage
```
Usage: RustygenC <input file(s)>
```

## Example
### asm:
```asm
str_rdm db '%d\n', 0
main:
  push rbp    
  mov rbp, rsp    
  sub rsp, 30h    
  call __main    
  mov dword ptr [rbp-4], 0Ah    
  mov eax, [rbp-4]    
  mov edx, eax    
  shr edx, 1Fh    
  add eax, edx    
  sar eax, 1    
  mov [rbp-8], eax    
  mov eax, [rbp-8]    
  mov edx, eax    
  lea rax, [str_rdm]    
  mov rcx, rax    
  call printf    
  mov eax, 0    
  add rsp, 30h    
  pop rbp   
  ret    
  ``` 
### Pseudo C:
```c
#include <stdio.h>
#include <stdint.h>
uint8_t str_rdm[5] = {'%', 'd', '\n', 0};

uint32_t main(){
    uint64_t rbp;
    uint64_t rax;
    uint64_t rdx;
    uint64_t rcx;
    __main();
    uint32_t st_3 = 0x0A;
    *(uint32_t*)&rax = st_3;
    *(uint32_t*)&rdx = (uint32_t)rax;
    *(uint32_t*)&rdx >>= 0x1F;
    *(uint32_t*)&rax += (uint32_t)rdx;
    *(uint32_t*)&rax >>= 1;
    uint32_t st_4 = (uint32_t)rax;
    *(uint32_t*)&rax = st_4;
    *(uint32_t*)&rdx = (uint32_t)rax;
    rax = &str_rdm;
    rcx = rax;
    printf(rcx, rdx);
    *(uint32_t*)&rax = 0;
    return (uint32_t)rax;
}
```


the C code is still very low level, I know.. 
here is an example of condition
### in assembly : 
```asm
main:
  mov rbp, rsp
  mov [rbp-4], 21
  cmp dword [rbp-4], 5
  jnl caca
``` 
### C genered : 
```c
#include <stdio.h>
#include <stdint.h>

void main(){
    uint32_t st_2 = 21;
    if ((int32_t)st_2 >= (int32_t)5) {
    goto caca;
    }
}
```
WARNING, if you have a function which does not have an epilogue and which has not been called, it is preferable to add an instruction "call theNameOfTheFunction" in your history code so that the generator knows that it is a function 

### without the call statement 
**asm**: 
```asm
formats db 'rdm print: %d', 0

main:
  mov [rsp-4], 21
  cmp dword [rsp-4], 5
  jnl caca

  caca: 
  mov rdx, formats
  mov rcx, 1
  call print
```
**C genered :**
```c
#include <stdio.h>
#include <stdint.h>
uint8_t formats[14] = {'r', 'd', 'm', ' ', 'p', 'r', 'i', 'n', 't', ':', ' ', '%', 'd', 0};

main:
     = 21;
    if ((int32_t) >= (int32_t)5) {
    goto caca;
    }

    caca:
    rcx = &formats;
    rdx = 1;
    print(rcx, rdx);
}
```


here we can see that the variables on the stack are no longer represented in the C code and the supposed function is considered as some label instead of being considered as a function
if we add a `call main` in the code we obtain : 
```C
#include <stdio.h>
#include <stdint.h>
uint8_t formats[14] = {'r', 'd', 'm', ' ', 'p', 'r', 'i', 'n', 't', ':', ' ', '%', 'd', 0};
main();

void main(){
    uint64_t rdx;
    uint64_t rcx;
    uint32_t st_2 = 21;
    if ((int32_t)st_2 >= (int32_t)5) {
    goto caca;
    }

    caca:
    rcx = &formats;
    rdx = 1;
    print(rcx, rdx);
}
```
for function arguments, we support the conventions: Fastcall, Thiscall, Vectorcall, msfastcall
 ### example : 
 **assembly fastcall :** 
 ```asm
mov rdx, 10
mov rcx, 20
call fast_call_func
``` 
**C genered :** 
```c
#include <stdio.h>
#include <stdint.h>
rdx = 10;
rcx = 20;
fast_call_func(rcx, rdx);
}
```
etc for other (I have not yet implemented recognition for arguments on the stack)
if you can try to use the byte, word, dword, qword directives as much as possible because it's easier for the script to deal with

#### THE PROJECT IS UNDER DEVELOPMENT IT IS NORMAL THAT THEY HAVE NOT MANY INSTRUCTIONS SUPPORTED

