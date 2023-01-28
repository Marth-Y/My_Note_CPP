- Hey folks, Long time no C.
- https://arjunsreedharan.org/
- Generally in C, this is how we find the length of an array arr :
- `n = sizeof(arr) / sizeof(arr[0]);`
- Here we take the size of the array in bytes; then divide it by the size of an individual array element.
- What if I tell you that we can get rid of sizeof and have a cooler way to calculate size? like this:
- `n = (&arr)[1] - arr;`
- Alright, let’s see how that could be correct!
- Have you ever thought what’s the difference between arr and &arr?
- Well let’s check that out by printing the memory addresses of both:
- ```cpp
  int arr[5] = {1, 2, 3, 4, 5};
  printf("Address of arr  is %p\n", (void*)arr);
  printf("Address of &arr is %p\n", (void*)&arr);
  ```
- and here’s the output:
- ```cpp
  $ gcc -o a size.c
  $ ./a
  ```
- Address of `arr` 		is `0x7fff57266870`
- Address of `&arr` 	is `0x7fff57266870`
- As you can see in the output, both arr and &arr point to the exact same memory location `0x7fff57266870`.
- Now, let’s increment both the pointers by 1 and check their memory address.
- Here’s the code to check for the memory address of arr + 1 and &arr + 1 :
- ```cpp
  int arr[5] = {1, 2, 3, 4, 5};
  printf("Address of arr      is %p\n", (void*)arr);
  printf("Address of &arr     is %p\n", (void*)&arr);
  printf("Address of arr + 1  is %p\n", (void*)(arr + 1));
  printf("Address of &arr + 1 is %p\n", (void*)(&arr + 1));
  ```
- and the output:
- ```cpp
  $ gcc -o a size.c
  $ ./a
  ```
- Address of `arr` 		is `0x7fff57266870`
- Address of `&arr` 	is `0x7fff57266870`
- Address of `arr + 1` 	is `0x7fff57266874`
- Address of `&arr + 1` 	is `0x7fff57266884`
- We find that:
- `(arr + 1)` points to 874 which is 4 bytes away from arr, which points to 870 (I have removed the higher order bits of the address for brevity). An int on my machine takes up 4 bytes, so `(arr + 1)` points to the second element of the array.
- `(&arr + 1)` points to 884 which is 20 bytes away from arr (points to 870).
- (884 - 870 = 14 in hex = 20 in decimal)
- Taking the size of int into consideration, `(&arr + 1)` is 5 int-sizes away from the beginning of the array.
- 5 also happens to be the size of the array. So, `(&arr + 1)` points to the memory address after the end of the array.
- Why is `(arr + 1)` and`(&arr + 1)`  different though arr and &arr point to the same location?
  The answer - While `(arr + 1)` and `(&arr + 1)` have the values, they are different types.
  arr is of the type int `*`, where as`&arr`  is of the type `int (*)[size]`.
- So, `&arr` points to the entire array where as arr points to the first element of the array.
- image
- This brings us to something useful - length of the array.
- `* (&arr + 1)` gives us the address after the end of the array and arr that of the first element of the array.
- Subtracting latter from former would thus give the length of the array.
- `n = *(&arr + 1) - arr;`
- We can simplify this using array indexes (since `x[1]` is same as `*(x+1)`), and here you go:
- `n = (&arr)[1] - arr;`
- Pretty cool right? But, please don’t really use this in real life, unless there’s a compelling reason. sizeof is an operator evaluated at compile time (execpt for C99 variable-length arrays), so it’s also pretty cool, though it doesn’t look it!
- PS:
- This works only for arrays, not when you take pointers (as in `char *str` for strings).
- ```cpp
  void reverse_str(char *str)
  { 
    //wrong
    int strlength = (&str)[1] - str;
  }
  ```
- In this case `&str` is a pointer that points to the pointer `str`. Remember, in C arrays are not pointers.
- Addendum:
- Here’s an interesting Question/Answer I found on stackoverflow regarding the same topic.
- Q: Accessing the first address after an array seems to be undefined behavior. For example: if your array located at the end of an address-space, the referencing address causing an overflow and your resulting size could be anything. Then how could you access (&arr)[1] ?
-
- A: C doesn’t allow access to memory beyond the end of the array. It does, however, allow a pointer to point at one element beyond the end of the array. The distinction is important.