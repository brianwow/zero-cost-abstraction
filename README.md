# Zero Cost Abstraction

To explain it in a simple term, Zero Cost Abstraction means you can use cool high-level programming language features without adding unnecessary stuffs behind the scene.

Take a look at the example of this concept in Rust:

#### Imperative Approach

Normally in any other language, to make a loop statement: you must initialize a variable, the stopping rule, and its incremental/decremental expression.\
While Rust doesn't have a lower level C-like syntax `for (int i=0; i < n; i++)`, we can achieve a similiar result by this approach using range `for i in 1..n`:

```rust
pub fn sum(n: i32) -> i32 {
  let mut sum = 0;
  for i in 1..n {
    sum += i;
  }
  sum
}
```

When it compile downs to assembly code, we can see that the code indeed compiles down into what we expected from an assembly code:\
<sub>_Disclaimer: IDK how assembly code should looks like, but I guess it must be this simple_</sub>

```asm
example::sum:
        xor     eax, eax
        cmp     edi, 2
        jl      .LBB0_2
        lea     eax, [rdi - 2]
        lea     ecx, [rdi - 3]
        imul    rcx, rax
        shr     rcx
        lea     eax, [rcx + 2*rdi]
        add     eax, -3
.LBB0_2:
        ret
```

<sub>Run this example yourself in [godbolt.org](https://godbolt.org/z/WrWzPh6zT)</sub>

#### Functional Approach

Another approach is to use the functional approach provided by the standard library in Rust:

```rust
pub fn sum(n: i32) -> i32 {
   (1..n).fold(0, |x, y| x + y)
}
```

This resulting in the same assembly output as the previous approach:

```asm
example::sum:
        xor     eax, eax
        cmp     edi, 2
        jl      .LBB0_2
        lea     eax, [rdi - 2]
        lea     ecx, [rdi - 3]
        imul    rcx, rax
        shr     rcx
        lea     eax, [rcx + 2*rdi]
        add     eax, -3
.LBB0_2:
        ret
```

<sub>Run this example yourself in [godbolt.org](https://godbolt.org/z/rMMnhKGE3)</sub>

#### A More Functional Approach

This code still can be simplify by calling `.sum()`:

```rust
pub fn sum(n: i32) -> i32 {
  (1..n).sum()
}
```

Which then again, resulting in the same result:

```asm
example::sum:
        xor     eax, eax
        cmp     edi, 2
        jl      .LBB0_2
        lea     eax, [rdi - 2]
        lea     ecx, [rdi - 3]
        imul    rcx, rax
        shr     rcx
        lea     eax, [rcx + 2*rdi]
        add     eax, -3
.LBB0_2:
        ret
```

<sub>Run this example yourself in [godbolt.org](https://godbolt.org/z/Ws1rGWhTs)</sub>

#### Oh-God-Why Approach

While this concept make you feel more confident about how you write your code, keep in mind that you should still never use the functionality you don't need, since the Zero Cost Abstraction doesn't fix your logic:

```rust
pub fn sum(n: i32) -> i32 {
   (1..n)
        .rev()
        .collect::<Vec<_>>()
        .iter()
        .rev()
        .sum()
}
```

This miserable code will resulting in a whooping 454 lines of assembly:

```asm
core::ptr::drop_in_place<alloc::vec::Vec<i32>>:
        mov     rsi, qword ptr [rdi + 8]
        test    rsi, rsi
        je      .LBB0_3
        mov     rdi, qword ptr [rdi]
        test    rdi, rdi
        je      .LBB0_3
        shl     rsi, 2
        test    rsi, rsi
        je      .LBB0_3
        mov     edx, 4
        jmp     qword ptr [rip + __rust_dealloc@GOTPCREL]
.LBB0_3:
        ret

alloc::raw_vec::finish_grow:
        push    r15
        push    r14
        push    rbx
        mov     rbx, rsi
        mov     r14, rdi
        test    rdx, rdx
        je      .LBB1_5
        mov     r15, rdx
        mov     rdi, qword ptr [rcx]
        test    rdi, rdi
        je      .LBB1_6
        mov     rsi, qword ptr [rcx + 8]
        test    rsi, rsi
        je      .LBB1_6
        mov     rdx, r15
        mov     rcx, rbx
        call    qword ptr [rip + __rust_realloc@GOTPCREL]
        test    rax, rax
        jne     .LBB1_10
.LBB1_4:
        mov     qword ptr [r14 + 8], rbx
        mov     eax, 1
        mov     rbx, r15
        jmp     .LBB1_11
.LBB1_6:
        test    rbx, rbx
        je      .LBB1_7
        mov     rdi, rbx
        mov     rsi, r15
        call    qword ptr [rip + __rust_alloc@GOTPCREL]
        test    rax, rax
        je      .LBB1_4
.LBB1_10:
        mov     qword ptr [r14 + 8], rax
        xor     eax, eax
        jmp     .LBB1_11
.LBB1_5:
        mov     qword ptr [r14 + 8], rbx
        mov     eax, 1
        xor     ebx, ebx
.LBB1_11:
        mov     qword ptr [r14 + 16], rbx
        mov     qword ptr [r14], rax
        pop     rbx
        pop     r14
        pop     r15
        ret
.LBB1_7:
        mov     rax, r15
        test    rax, rax
        jne     .LBB1_10
        jmp     .LBB1_4

alloc::raw_vec::RawVec<T,A>::reserve::do_reserve_and_handle:
        push    r14
        push    rbx
        sub     rsp, 56
        add     rsi, rdx
        jb      .LBB2_8
        mov     r14, rdi
        mov     rcx, qword ptr [rdi + 8]
        lea     rax, [rcx + rcx]
        cmp     rax, rsi
        cmova   rsi, rax
        cmp     rsi, 5
        mov     edx, 4
        cmovb   rsi, rdx
        xor     ebx, ebx
        mov     rax, rsi
        mul     rdx
        setno   bl
        shl     rbx, 2
        test    rcx, rcx
        je      .LBB2_3
        mov     rdx, qword ptr [r14]
        shl     rcx, 2
        mov     qword ptr [rsp + 32], rdx
        mov     qword ptr [rsp + 40], rcx
        mov     qword ptr [rsp + 48], 4
        jmp     .LBB2_4
.LBB2_3:
        mov     qword ptr [rsp + 32], 0
.LBB2_4:
        lea     rdi, [rsp + 8]
        lea     rcx, [rsp + 32]
        mov     rsi, rax
        mov     rdx, rbx
        call    alloc::raw_vec::finish_grow
        cmp     dword ptr [rsp + 8], 1
        je      .LBB2_5
        mov     rax, qword ptr [rsp + 16]
        mov     rcx, qword ptr [rsp + 24]
        mov     qword ptr [r14], rax
        shr     rcx, 2
        mov     qword ptr [r14 + 8], rcx
        add     rsp, 56
        pop     rbx
        pop     r14
        ret
.LBB2_5:
        mov     rsi, qword ptr [rsp + 24]
        test    rsi, rsi
        jne     .LBB2_6
.LBB2_8:
        call    qword ptr [rip + alloc::raw_vec::capacity_overflow@GOTPCREL]
        ud2
.LBB2_6:
        mov     rdi, qword ptr [rsp + 16]
        call    qword ptr [rip + alloc::alloc::handle_alloc_error@GOTPCREL]
        ud2

.LCPI3_0:
        .long   0
        .long   4294967295
        .long   4294967294
        .long   4294967293
.LCPI3_1:
        .long   4294967291
        .long   4294967291
        .long   4294967291
        .long   4294967291
.LCPI3_2:
        .long   4294967287
        .long   4294967287
        .long   4294967287
        .long   4294967287
.LCPI3_3:
        .long   4294967283
        .long   4294967283
        .long   4294967283
        .long   4294967283
.LCPI3_4:
        .long   4294967279
        .long   4294967279
        .long   4294967279
        .long   4294967279
.LCPI3_5:
        .long   4294967275
        .long   4294967275
        .long   4294967275
        .long   4294967275
.LCPI3_6:
        .long   4294967271
        .long   4294967271
        .long   4294967271
        .long   4294967271
.LCPI3_7:
        .long   4294967267
        .long   4294967267
        .long   4294967267
        .long   4294967267
.LCPI3_8:
        .long   4294967264
        .long   4294967264
        .long   4294967264
        .long   4294967264
.LCPI3_9:
        .long   4294967288
        .long   4294967288
        .long   4294967288
        .long   4294967288
example::sum:
        push    r15
        push    r14
        push    r12
        push    rbx
        sub     rsp, 24
        movsxd  rax, edi
        lea     rcx, [rax - 1]
        xor     r14d, r14d
        cmp     eax, 2
        cmovge  r14, rcx
        mov     ecx, 4
        xor     r12d, r12d
        mov     rax, r14
        mul     rcx
        mov     r15, rax
        setno   al
        jo      .LBB3_40
        mov     ebx, edi
        mov     r12b, al
        shl     r12, 2
        test    r15, r15
        je      .LBB3_2
        mov     rdi, r15
        mov     rsi, r12
        call    qword ptr [rip + __rust_alloc@GOTPCREL]
        test    rax, rax
        je      .LBB3_41
.LBB3_5:
        shr     r15, 2
        mov     qword ptr [rsp], rax
        mov     qword ptr [rsp + 8], r15
        mov     qword ptr [rsp + 16], 0
        cmp     r15, r14
        jae     .LBB3_6
        mov     rdi, rsp
        xor     esi, esi
        mov     rdx, r14
        call    alloc::raw_vec::RawVec<T,A>::reserve::do_reserve_and_handle
        mov     r11, qword ptr [rsp + 16]
        mov     rdi, qword ptr [rsp]
        cmp     ebx, 2
        jge     .LBB3_10
.LBB3_23:
        mov     qword ptr [rsp + 16], r11
        test    r11, r11
        je      .LBB3_24
.LBB3_25:
        lea     rax, [rdi + 4*r11]
        lea     rdx, [4*r11 - 4]
        xor     ebx, ebx
        cmp     rdx, 28
        jb      .LBB3_35
        shr     rdx, 2
        add     rdx, 1
        mov     r8, rdx
        and     r8, -8
        lea     rcx, [r8 - 8]
        mov     r9, rcx
        shr     r9, 3
        add     r9, 1
        test    rcx, rcx
        je      .LBB3_27
        lea     rsi, [rdi + 4*r11]
        add     rsi, -16
        mov     rbx, r9
        and     rbx, -2
        neg     rbx
        pxor    xmm0, xmm0
        mov     rcx, -4
        pxor    xmm1, xmm1
.LBB3_29:
        movdqu  xmm2, xmmword ptr [rsi + 4*rcx - 32]
        movdqu  xmm3, xmmword ptr [rsi + 4*rcx - 16]
        movdqu  xmm4, xmmword ptr [rsi + 4*rcx]
        movdqu  xmm5, xmmword ptr [rsi + 4*rcx + 16]
        pshufd  xmm5, xmm5, 27
        paddd   xmm5, xmm0
        pshufd  xmm4, xmm4, 27
        paddd   xmm4, xmm1
        pshufd  xmm0, xmm3, 27
        paddd   xmm0, xmm5
        pshufd  xmm1, xmm2, 27
        paddd   xmm1, xmm4
        add     rcx, -16
        add     rbx, 2
        jne     .LBB3_29
        add     rcx, 3
        test    r9b, 1
        je      .LBB3_33
.LBB3_32:
        movdqu  xmm2, xmmword ptr [rax + 4*rcx - 28]
        movdqu  xmm3, xmmword ptr [rax + 4*rcx - 12]
        pshufd  xmm2, xmm2, 27
        paddd   xmm1, xmm2
        pshufd  xmm2, xmm3, 27
        paddd   xmm0, xmm2
.LBB3_33:
        paddd   xmm0, xmm1
        pshufd  xmm1, xmm0, 238
        paddd   xmm1, xmm0
        pshufd  xmm0, xmm1, 85
        paddd   xmm0, xmm1
        movd    ebx, xmm0
        cmp     rdx, r8
        je      .LBB3_36
        shl     r8, 2
        sub     rax, r8
.LBB3_35:
        add     ebx, dword ptr [rax - 4]
        add     rax, -4
        cmp     rax, rdi
        jne     .LBB3_35
.LBB3_36:
        mov     rsi, qword ptr [rsp + 8]
        test    rsi, rsi
        jne     .LBB3_37
        jmp     .LBB3_39
.LBB3_6:
        xor     r11d, r11d
        mov     rdi, qword ptr [rsp]
        cmp     ebx, 2
        jl      .LBB3_23
.LBB3_10:
        lea     rdx, [rdi + 4*r11]
        lea     r9d, [rbx - 2]
        cmp     r9d, 7
        jb      .LBB3_20
        lea     r8, [r9 + 1]
        mov     r10, r8
        and     r10, -8
        movd    xmm0, ebx
        pshufd  xmm0, xmm0, 0
        paddd   xmm0, xmmword ptr [rip + .LCPI3_0]
        lea     rcx, [r10 - 8]
        mov     rax, rcx
        shr     rax, 3
        add     rax, 1
        mov     r14d, eax
        and     r14d, 3
        cmp     rcx, 24
        jae     .LBB3_13
        xor     ecx, ecx
        jmp     .LBB3_15
.LBB3_2:
        mov     rax, r12
        test    rax, rax
        jne     .LBB3_5
.LBB3_41:
        mov     rdi, r15
        mov     rsi, r12
        call    qword ptr [rip + alloc::alloc::handle_alloc_error@GOTPCREL]
        ud2
.LBB3_13:
        lea     rsi, [rdi + 4*r11]
        add     rsi, 112
        and     rax, -4
        neg     rax
        xor     ecx, ecx
        pcmpeqd xmm8, xmm8
        movdqa  xmm9, xmmword ptr [rip + .LCPI3_1]
        movdqa  xmm10, xmmword ptr [rip + .LCPI3_2]
        movdqa  xmm11, xmmword ptr [rip + .LCPI3_3]
        movdqa  xmm5, xmmword ptr [rip + .LCPI3_4]
        movdqa  xmm6, xmmword ptr [rip + .LCPI3_5]
        movdqa  xmm7, xmmword ptr [rip + .LCPI3_6]
        movdqa  xmm1, xmmword ptr [rip + .LCPI3_7]
        movdqa  xmm2, xmmword ptr [rip + .LCPI3_8]
.LBB3_14:
        movdqa  xmm3, xmm0
        paddd   xmm3, xmm8
        movdqa  xmm4, xmm0
        paddd   xmm4, xmm9
        movdqu  xmmword ptr [rsi + 4*rcx - 112], xmm3
        movdqu  xmmword ptr [rsi + 4*rcx - 96], xmm4
        movdqa  xmm3, xmm0
        paddd   xmm3, xmm10
        movdqa  xmm4, xmm0
        paddd   xmm4, xmm11
        movdqu  xmmword ptr [rsi + 4*rcx - 80], xmm3
        movdqu  xmmword ptr [rsi + 4*rcx - 64], xmm4
        movdqa  xmm3, xmm0
        paddd   xmm3, xmm5
        movdqa  xmm4, xmm0
        paddd   xmm4, xmm6
        movdqu  xmmword ptr [rsi + 4*rcx - 48], xmm3
        movdqu  xmmword ptr [rsi + 4*rcx - 32], xmm4
        movdqa  xmm3, xmm0
        paddd   xmm3, xmm7
        movdqa  xmm4, xmm0
        paddd   xmm4, xmm1
        movdqu  xmmword ptr [rsi + 4*rcx - 16], xmm3
        movdqu  xmmword ptr [rsi + 4*rcx], xmm4
        add     rcx, 32
        paddd   xmm0, xmm2
        add     rax, 4
        jne     .LBB3_14
.LBB3_15:
        test    r14, r14
        je      .LBB3_18
        add     rcx, r11
        lea     rax, [rdi + 4*rcx]
        add     rax, 16
        shl     r14, 5
        xor     ecx, ecx
        pcmpeqd xmm1, xmm1
        movdqa  xmm2, xmmword ptr [rip + .LCPI3_1]
        movdqa  xmm3, xmmword ptr [rip + .LCPI3_9]
.LBB3_17:
        movdqa  xmm4, xmm0
        paddd   xmm4, xmm1
        movdqa  xmm5, xmm0
        paddd   xmm5, xmm2
        movdqu  xmmword ptr [rax + rcx - 16], xmm4
        movdqu  xmmword ptr [rax + rcx], xmm5
        paddd   xmm0, xmm3
        add     rcx, 32
        cmp     r14, rcx
        jne     .LBB3_17
.LBB3_18:
        cmp     r8, r10
        je      .LBB3_22
        lea     rdx, [rdx + 4*r10]
        sub     ebx, r10d
.LBB3_20:
        add     ebx, 1
.LBB3_21:
        lea     eax, [rbx - 2]
        mov     dword ptr [rdx], eax
        add     rdx, 4
        add     ebx, -1
        cmp     ebx, 2
        jg      .LBB3_21
.LBB3_22:
        add     r11, r9
        add     r11, 1
        mov     qword ptr [rsp + 16], r11
        test    r11, r11
        jne     .LBB3_25
.LBB3_24:
        xor     ebx, ebx
        mov     rsi, qword ptr [rsp + 8]
        test    rsi, rsi
        je      .LBB3_39
.LBB3_37:
        shl     rsi, 2
        test    rsi, rsi
        je      .LBB3_39
        mov     edx, 4
        call    qword ptr [rip + __rust_dealloc@GOTPCREL]
.LBB3_39:
        mov     eax, ebx
        add     rsp, 24
        pop     rbx
        pop     r12
        pop     r14
        pop     r15
        ret
.LBB3_27:
        pxor    xmm0, xmm0
        mov     rcx, -1
        pxor    xmm1, xmm1
        test    r9b, 1
        jne     .LBB3_32
        jmp     .LBB3_33
.LBB3_40:
        call    qword ptr [rip + alloc::raw_vec::capacity_overflow@GOTPCREL]
        ud2
        mov     rbx, rax
        mov     rdi, rsp
        call    core::ptr::drop_in_place<alloc::vec::Vec<i32>>
        mov     rdi, rbx
        call    _Unwind_Resume@PLT
        ud2

DW.ref.rust_eh_personality:
        .quad   rust_eh_personality
```

<sub>_You are not going to run this code by [yourself](https://godbolt.org/z/a8KWGKK7T)_</sub>

Keep your code simple, that will most of the time guarantee a better result.

#### Further readings:

- Zero Cost Abstraction in Rust by Amirreza Askarpour [Blog Post](https://itnext.io/zero-cost-abstractions-in-rust-26d058eb1724)
- An Overview of Rust Programming Language by Senior Mars [Presentation](https://www.youtube.com/watch?v=aBHCTHeLjD0) | [Markdown Source](https://github.com/KarlWithK/rust-presentation/blob/master/rust-pres.md)
