# Trick

##1. 如何直接跳出深层递归而不是一层一层跳出？

1. throw exception，try catch
2. 把递归函数放到一个独立线程执行，在主线程做 condition wait，递归结束时notify下，然后直接退出线程。
3. C语言的setjmp + longjmp
