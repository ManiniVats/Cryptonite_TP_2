# ARMssembly 1

Upon opening the file `chall_1.S`, I tried to interpret the code using [https://www.youtube.com/watch?v=1d-6Hv1c39c] and [https://developer.arm.com/documentation/ddi0602/2023-06/Base-Instructions?lang=en]

```
	.arch armv8-a
	.file	"chall_1.c"
	.text
	.align	2
	.global	func
	.type	func, %function
func:
	sub	sp, sp, #32
	str	w0, [sp, 12]
	mov	w0, 58
	str	w0, [sp, 16]
	mov	w0, 2
	str	w0, [sp, 20]
	mov	w0, 3
	str	w0, [sp, 24]
	ldr	w0, [sp, 20]
	ldr	w1, [sp, 16]
	lsl	w0, w1, w0
	str	w0, [sp, 28]
	ldr	w1, [sp, 28]
	ldr	w0, [sp, 24]
	sdiv	w0, w1, w0
	str	w0, [sp, 28]
	ldr	w1, [sp, 28]
	ldr	w0, [sp, 12]
	sub	w0, w1, w0
	str	w0, [sp, 28]
	ldr	w0, [sp, 28]
	add	sp, sp, 32
	ret
	.size	func, .-func
	.section	.rodata
	.align	3
.LC0:
	.string	"You win!"
	.align	3
.LC1:
	.string	"You Lose :("
	.text
	.align	2
	.global	main
	.type	main, %function
main:
	stp	x29, x30, [sp, -48]!
	add	x29, sp, 0
	str	w0, [x29, 28]
	str	x1, [x29, 16]
	ldr	x0, [x29, 16]
	add	x0, x0, 8
	ldr	x0, [x0]
	bl	atoi
	str	w0, [x29, 44]
	ldr	w0, [x29, 44]
	bl	func
	cmp	w0, 0
	bne	.L4
	adrp	x0, .LC0
	add	x0, x0, :lo12:.LC0
	bl	puts
	b	.L6
.L4:
	adrp	x0, .LC1
	add	x0, x0, :lo12:.LC1
	bl	puts
.L6:
	nop
	ldp	x29, x30, [sp], 48
	ret
	.size	main, .-main
	.ident	"GCC: (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0"
	.section	.note.GNU-stack,"",@progbits
```

`.file	"chall_1.c"` means that the challenge was made in `c` and instead of compliing, it was assembled in this assembly language.

`bl atoi`(ASCII to integer) is used to convert a string into an integer.

```
sub	sp, sp, #32
```
This means that the stack is being moved 32 bytes lower so to get space to store values.

```
str	w0, [sp, 12]
```
This, thus, stores the value of the main function argument `A` into the memory location 12 bytes below the current stack pointer, or, at `sp+12`.

Now the values 58, 2, 3 are put or moved in the `w0` register, where 58 is stored at `sp + 16`, 2 is stored at `sp + 20` and 3 is stored at `sp + 24`.
```
	mov	w0, 58
	str	w0, [sp, 16]
	mov	w0, 2
	str	w0, [sp, 20]
	mov	w0, 3
	str	w0, [sp, 24]
```


```
        ldr	w0, [sp, 20]
	ldr	w1, [sp, 16]
	lsl	w0, w1, w0
	str	w0, [sp, 28]
```
Here, 2 stored at `sp + 20` is loaded into `w0` and 58 stored at `sp + 16` is loaded into `w1` using `ldr`. Then, using `lsl`, left shift operation is performed, i.e., _58 << 2_ and the result, 232 is stored in `sp + 28`.

```
	ldr	w1, [sp, 28]
	ldr	w0, [sp, 24]
	sdiv	w0, w1, w0
	str	w0, [sp, 28]
```
Here, 232 stored at `sp + 28` is loaded into `w1`, and 3 stored at `sp + 24` is loaded into `w0`. Then using `sdiv`, _232/3_ is performed, and the result, 77 is stored at `sp + 28`.

```
	ldr	w1, [sp, 28]
	ldr	w0, [sp, 12]
	sub	w0, w1, w0
	str	w0, [sp, 28]
```
Here, 77 stored at `sp + 28` is loaded into `w1`, and `A` stored at `sp + 12` is loaded into `w0`. Then using `sub`, _77-A_ is performed and the result is stored at `sp + 28`

```
	cmp	w0, 0
	bne	.L4
	adrp	x0, .LC0
	add	x0, x0, :lo12:.LC0
	bl	puts
	b	.L6
```
Now using `cmp` and `bne`(branch on not equal), return value of `func` is compared with `0` 
```
.LC0:
	.string	"You win!"
	.align	3
.LC1:
	.string	"You Lose :("
	.text
	.align	2
	.global	main
	.type	main, %function
```
If it is `0`, it prints `You win!`, whereas if not, it prints `You Lose :("`.

```
	ldr	w0, [sp, 28]
	add	sp, sp, 32
	ret
```
From this it can be said that `77-A` stored at `sp + 28` is the return value of `func`.

Now in order for `77-A` to be `0`, `A` needs to be `77` and thus the argument will be `77` in hexadecimal, i.e., `0000004D`.

Thus the flag will be `picoCTF{0000004D}`.

