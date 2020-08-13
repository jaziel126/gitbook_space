# index

## 数组的定义

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

## 顺序存储表示和实现

### 存储方式

* 以行序为主
* 以列序为主

  一般以行序为主

  **存储结构**

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

## 基本操作

### 生成给定维度和维界的数组

```cpp
Status InitArray(Array& A,int dim,...) {//构建维度为dim的数组A；
    //检查dim是否合法
    if (dim<1 && dim>MAX_ARRAY_DIM) return OVERFLOW;
    A.dim = dim;
    //读取变长参数表，即将维界存入bounds；并求出数组元素总数elemtotal
    int elemtotal;
    va_list ap;
    va_start(ap, dim);
    for (int i = 0;i < dim;i++) {
        A.bounds[i] = va_arg(ap, int);
        elemtotal *= A.bounds[i];
    }
    va_end(ap);
    //分配数组空间
    A.base = (ElemType*)malloc(elemtotal * sizeof(ElemType));
    if (!A.base) return(OVERFLOW);
    //求映像函数常数ci；存入constants
    A.constants[dim - 1] = 1;//Cn为L，L是每个元素大小；
    for (int i = dim - 2;i >= 0;i--) {
        A.constants[i] = A.bounds[i + 1] * A.constants[i + 1];
    }
    return OK;
}
```

### 销毁数组

### 存入元素

### 取出元素

