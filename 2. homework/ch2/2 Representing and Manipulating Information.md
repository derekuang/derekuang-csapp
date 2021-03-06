# 2 Representing and Manipulating Information



## 2.60

- Problem

  > Suppose we number the bytes in a w-bit word from 0 (least significant) to w/8 − 1 (most significant). Write code for the following C function, which will return an unsigned value in which byte i of argument x has been replaced by byte b: 
  >
  > unsigned replace_byte (unsigned x, int i, unsigned char b); 
  >
  > ​		Here are some examples showing how the function should work:
  >
  > replace_byte(0x12345678, 2, 0xAB) --> 0x12AB5678 
  >
  > replace_byte(0x12345678, 0, 0xAB) --> 0x123456AB 
  >
  
- Solution

  ```c
  unsigned replace_byte(unsigned x, int i, unsigned char b)
  {
      unsigned char* p = (unsigned char*)&x;
      p[i] = b;
  
      return x;
  }
  ```



## Bit-Level Integer Coding Rules 

In several of the following problems, we will artificially restrict what programming constructs you can use to help you gain a better understanding of the bit-level, logic, and arithmetic operations of C. In answering these problems, your code must follow these rules:

- Assumptions
  - Integers are represented in two’s-complement form.
  - Right shifts of signed data are performed arithmetically.
  - Data type int is w bits long. For some of the problems, you will be given a specific value for w, but otherwise your code should work as long as w is a multiple of 8. You can use the expression sizeof(int)<<3 to compute w.
- Forbidden
  - Conditionals (if or ?:), loops, switch statements, function calls, and macro invocations.
  - Division, modulus, and multiplication.
  - Relative comparison operators (<, >, <=, and >=).
- Allowed operations
  - All bit-level and logic operations. 
  - Left and right shifts, but only with shift amounts between 0 and w − 1. 
  - Addition and subtraction. 
  - Equality (==) and inequality (!=) tests. (Some of the problems do not allow these.) 
  - Integer constants INT_MIN and INT_MAX. 
  - Casting between data types int and unsigned, either explicitly or implicitly.

​        Even with these rules, you should try to make your code readable by choosing descriptive variable names and using comments to describe the logic behind your solutions. As an example, the following code extracts the most significant byte from integer argument x:

```c
/* Get most significant byte from x */ 
int get_msb(int x) { 
	/* Shift by w-8 */
	int shift_val = (sizeof(int)-1)<<3; 
	/* Arithmetic shift */
	int xright = x >> shift_val;
	/* Zero all but LSB */ 
	return xright & 0xFF;
}
```



## 2.61

- Problem

  > Write C expressions that evaluate to 1 when the following conditions are true and to 0 when they are false. Assume x is of type int.
  >
  > ​		A. Any bit of x equals 1. 
  >
  > ​		B. Any bit of x equals 0.
  >
  > ​		C. Any bit in the least significant byte of x equals 1.
  >
  > ​		D. Any bit in the most significant byte of x equals 0.
  >
  > Your code should follow the bit-level integer coding rules (page 164), with the additional restriction that you may not use equality (==) or inequality (!=) tests.

- Solution

  ```c
  // A
  !!x;
  // B
  !!(~x);
  // C
  !!(x & 0xFF);
  // D
  !!(~x & 0xFF000000);
  ```



## 2.65

- Problem

  > Write code to implement the following function:
  >
  > /* Return 1 when x contains an odd number of 1s; 0 otherwise. Assume w=32 */
  >
  > int odd_ones(unsigned x);
  >
  > ​		Your function should follow the bit-level integer coding rules (page 164), except that you may assume that data type int has w = 32 bits. 
  >
  > ​		Your code should contain a total of at most 12 arithmetic, bitwise, and logical operations.

- Solution

  ```c
  int odd_ones(unsigned x)
  {
      x ^= x >> 16;
      x ^= x >> 8;
      x ^= x >> 4;
      x ^= x >> 2;
      x ^= x >> 1;
  
      return x & 1;
  }
  ```



## 2.86

- Problem/Solution

  > Intel-compatible processors also support an “extended-precision” floating-point format with an 80-bit word divided into a sign bit, k = 15 exponent bits, a single integer bit, and n = 63 fraction bits. The integer bit is an explicit copy of the implied bit in the IEEE floating-point representation. That is, it equals 1 for normalized values and 0 for denormalized values. Fill in the following table giving the approximate values of some “interesting” numbers in this format:
  >
  > ---
  >
  > ​																	  		  Extended precision
  >
  > **Description**														  Value		   Decimal
  >
  > Smallest positive denormalized 					   2^-16445^	       10^-4933^
  >
  > Smallest positive normalized						     2^-16382^   	     10^-4914^
  >
  > Largest normalized								         2^16384^-2^16320^    10^4914^-10^4796^
  >
  > ---
  >
  > ​		This format can be used in C programs compiled for Intel-compatible machines by declaring the data to be of type long double. However, it forces the compiler to generate code based on the legacy 8087 floating-point instructions. The resulting program will most likely run much slower than would be the case for data type float or double.



## Bit-Level Floating-Point Coding Rules

In the following problems, you will write code to implement floating-point functions, operating directly on bit-level representations of floating-point numbers. 

Your code should exactly replicate the conventions for IEEE floating-point operations, including using round-to-even mode when rounding is required. 

​		To this end, we define data type float_bits to be equivalent to unsigned:

```c
/* Access bit-level representation floating-point number */
typedef unsigned float_bits;
```

​		Rather **than** using data type float in your code, you will use float_bits. You may use both int and unsigned data types, including unsigned and integer constants and operations. You may not use any unions, structs, or arrays. Most significantly, you may not use any floating-point data types, operations, or constants. Instead, your code should perform the bit manipulations that implement the specified floating-point operations. 

​		The following function illustrates the use of these coding rules. For argument f , it returns ±0 if f is denormalized (preserving the sign of f ), and returns f otherwise.

```c
/* If f is denorm, return 0. Otherwise, return f */
float_bits float_denorm_zero(float_bits f) {
	/* Decompose bit representation into parts */
	unsigned sign = f>>31;
	unsigned exp = f>>23 & 0xFF;
	unsigned frac = f & 0x7FFFFF;
	if (exp == 0) {
		/* Denormalized. Set fraction to 0 */
		frac = 0;
	}
	/* Reassemble bits */
	return (sign << 31) | (exp << 23) | frac;
}
```



## 2.95

- Problem

  > Following the bit-level floating-point coding rules, implement the function with the following prototype:
  >
  > /* Compute 0.5*f. If f is NaN, then return f. */ 
  >
  > float_bits float_half(float_bits f);
  >
  > ​		For floating-point number f , this function computes 0.5 * f . If f is NaN, your function should simply return f . 
  >
  > ​		Test your function by evaluating it for all 2^32^ values of argument f and comparing the result to what would be obtained using your machine’s floating-point operations.

- Solution

  ```c
  typedef unsigned float_bits;
  
  float_bits float_half(float_bits f)
  {
      size_t s = f & 0x80000000;
      size_t e = f & 0x7F800000;
      size_t m = f & 0x007FFFFF;
  
      // 规格化的值
      if (e != 0 && e != 0x7F800000) {
          if (e == 0x00800000) {
              size_t em = e | m;
              em = ((em & 2) == 2) ? em+1 >> 1 : em >> 1;
              return s | em;
          }
          e -= 0x00800000;
          return s | e | m;
      }
      // NaN
      else if(e == 0x7F800000 && m != 0) {
          return f;
      }
      // 非规格化的值
      else {
          m = ((m & 2) == 2) ? m+1 >> 1 : m >> 1;
          return s | e | m;
      }
  }
  ```



## 2.97

- Problem

  > Following the bit-level floating-point coding rules, implement the function with the following prototype:
  >
  > /* Compute (float) i */ 
  >
  > float_bits float_i2f(int i);
  >
  > ​		For argument i, this function computes the bit-level representation of (float) i. 
  >
  > ​		Test your function by evaluating it for all 2^32^ values of argument f and comparing the result to what would be obtained using your machine’s floating-point operations.

- Solution

  ```c
  typedef unsigned float_bits;
  
  /* Compute (float) i */
  float_bits float_i2f(int i)
  {
      if (i == 0) {
          return 0;
      }
  
      size_t s, e, m, b;
  
      s = (i >> 31) & 1;
      i = (s == 0) ? i : (~i)+1; // 转换成非负数操作
  
      e = m = 0;
      while (i != 0) {
          b = i & 1; // 取lsb(最低位)
          m = (b << 31) | (m >> 1); // 放在msb(最高位)
          e++;
          i = (unsigned)i >> 1; // 确保最高位用0补充
      }
  
      // normalize m
      m <<= 1;
      e--;
  
      // round
      size_t lsb = m & 0x200;
      size_t t = m & 0x1FF;
      if (t > 0x100 || (t == 0x100 && lsb == 0x200)) {
          m += 0x200;
          e = (m >> 9 == 0) ? e+1 : e;
      }
  
      s <<= 31;
      e = (e+127) << 23; // 加上bias
      m >>= 9; // 舍掉
      return s | e | m;
  }
  ```
  
  