# Binary Representation

For example, the C expression <code>(3.14+1e20)-1e20</code> will evaluate to <code>0.0</code> on most machines, while <code>3.14+(1e20- 1e20)</code> will evaluate to <code>3.14</code>. Be careful!


**little endian** : the least significant byte comes first—is referred to as little endian.

**big endian** : the most significant byte comes first—is referred to as big endian.

    //No need to take a local variable. No performance improvement.
    void inplace_swap(int *x, int *y) {
        *y = *x ^ *y;
        *x = *x ^ *y;
        *y = *x ^ *y;
    }
<br />

Looking at the following program(my machine is 64 bit)

    string str("");
    cout << str.length()-1 << endl;
    cout << (((unsigned long)2 << 63)-1) << endl;

What will the program output? <code>-1</code>? NO! In my machine, it outputs <code> 18446744073709551615 </code>. Because <code>length()</code> return a size_t data type, so there is a type conversion. <code> 18446744073709551615 </code> is the maximum value of unsigned long in my machine.