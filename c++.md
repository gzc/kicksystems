# C++

##1. Vtable

    #include<iostream>
    #include<stack>
    using namespace std;

    class Base {
    public:
        virtual void f() { cout << "Base::f" << endl; }
        virtual void g() { cout << "Base::g" << endl; }
        virtual void h() { cout << "Base::h" << endl; }
    };

    int main()
    {
        typedef void(*Fun)(void);
    
	    static int global = 1;
        Base b1;
        Base b2;
    

	    cout << "global variable address : " << &global << endl; 
        cout << "虚函数表地址：" << (int*)*(int*)(&b1) << endl;
        cout << "虚函数表地址：" << (int*)*(int*)(&b2) << endl;
    
        cout << "虚函数表 — 第一个函数地址：" << (int*)*(int*)(&b1) << endl;
        cout << "虚函数表 — 第一个函数地址：" << (int*)*(int*)(&b2) << endl;
    
        // Invoke the first virtual function
        Fun pFun1 = (Fun)(*(int*)(*(int*)(&b1)));
	    Fun pFun2 = (Fun)(*(int*)((*(int*)&b1)+8));
	    Fun pFun3 = (Fun)(*(int*)((*(int*)&b1)+16));
        pFun1();
	    pFun2();
	    pFun3();

        return 0;
    }
    
**output:**

    global variable address : 0x602070
    虚函数表地址：0x400cb0
    虚函数表地址：0x400cb0
    虚函数表 — 第一个函数地址：0x400cb0
    虚函数表 — 第一个函数地址：0x400cb0
    Base::f
    Base::g
    Base::h


