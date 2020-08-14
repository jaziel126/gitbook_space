# 数组的顺序表示及实现

## 1.数组的定义

在c语言中，一个二位数组可以定义为一个一维数组类型分量的一位数组，即：

```cpp
typedef ElemType Array[m][n];
```

等价于：

```cpp
typedef ElemType Array[m];
typedef Array1[n];
```

需要注意，数组一旦被定义，它的维数和维界就**不再改变**，除了初始化和销毁之外，数组只有存取和修改元素值的操作；

## 2.顺序存储表示和实现

### 2.1存储方式

* 以行序为主
* 以列序为主

  一般以行序为主

### 2.2存储结构

```cpp
#include<stdarg.h>//提供宏va_list、va_start、va_end;
                  //用于存取变长参数表
typedef struct{
    ElemType *base;//数组元素基地址
    int dim;//数组的维度
    int* bounds;//数组维界
    int* constants;//用于存储数组映像函数常量Ci
    }Array;
```

> va\_list ap; 定义va\_list类型变量 va\_start\(ap,dim\);初始化ap,dim为...之前的参数 va\_arg\(ap,ElemType\);返回这个类型的值；ElemType为...数据类型 va\_end\(ap\);释放ap [va\_start和va\_end使用详解](https://www.cnblogs.com/hanyonglu/archive/2011/05/07/2039916.html)

## 3.基本操作

### 3.1生成给定维度和维界的数组

步骤：

* 检查给出维度dim是否合法
* 读取变长参数表，存入bounds用于求出数组元素总数elemtotal来确定给数组分配空间大小
* 分配数组空间
* 求映像函数常数ci，存入constants，便与后面求元素在数组中的相对地址

```cpp
//构建维度为dim的数组A；
Status InitArray(Array& A, int dim, ...) {
    //检查dim是否合法
    if (dim<1 || dim>MAX_ARRAY_DIM) return OVERFLOW;
    A.dim = dim;
    //读取变长参数表，存入bounds用于求出数组元素总数elemtotal来确定给数组分配空间大小
    int elemtotal = 1;
    va_list ap;
    va_start(ap, dim);
    A.bounds = (int*)malloc(dim * sizeof(int));
    for (int i = 0;i < dim;i++) {
        A.bounds[i] = va_arg(ap, int);
        elemtotal *= A.bounds[i];
    }
    va_end(ap);
    //分配数组空间
    A.base = (ElemType*)malloc(elemtotal * sizeof(ElemType));
    if (!A.base) return(OVERFLOW);
    //求映像函数常数ci，存入constants，便与后面求元素在数组中的相对地址；
    A.constants = (int*)malloc(dim * sizeof(int));
    A.constants[dim - 1] = 1;//Cn为L，L是每个元素大小；
    for (int i = dim - 2;i >= 0;i--) {
        A.constants[i] = A.bounds[i + 1] * A.constants[i + 1];
    }
    return OK;
}
```

### 3.2销毁数组

```cpp
//销毁数组A
Status DestoryArray(Array &A) {
    if (!A.base) return ERROR;
    free(A.base);
    A.base = NULL;
    if (!A.bounds) return ERROR;
    free(A.bounds);
    A.bounds = NULL;
    if (!A.constants) return ERROR;
    free(A.constants);
    A.constants = NULL;
    return OK;
}
```

### 3.3求元素地址

```cpp
//求元素e，就是va_list变长参数表指出的元素在数组中的位置
Status LocatArray(Array A, va_list ap, int &off) {
    off = 0;
    for (int i = 0;i < A.dim;i++) {
        int a = va_arg(ap, int);
        off += A.constants[i] * a;
    }
    return OK;
}
```

### 3.4存入元素

```cpp
//存入数
Status AssignArray(Array& A, ElemType e, ...) {
    va_list ap;
    va_start(ap, e);
    int off;
    LocatArray(A, ap, off);
    *(A.base + off) = e;
    va_end(ap);
    return OK;
}
```

### 3.5取出元素

```cpp
//取出数
Status ValueArray(ElemType& e, Array A, ...) {
    va_list ap;
    va_start(ap, A);
    int off;
    LocatArray(A, ap, off);
    e = *(A.base + off);
    va_end(ap);
    return OK;
}
```

[仓库地址](https://github.com/jaziel126/text2.git)

