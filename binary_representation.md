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

Looking at the following program

    string str("");
    cout << ss.length()-1 << endl;
