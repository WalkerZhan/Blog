# Solidity

## 源文件结构

```javascript
pragma solidity ^0.4.0;

//此语句将从 “filename” 中导入所有的全局符号到当前全局作用域中
import "filename";

//创建一个新的全局符号 symbolName，其成员均来自 "filename" 中全局符号。
import * as symbolName from "filename";
import "filename" as symbolName

//创建新的全局符号 alias 和 symbol2，分别从 "filename" 引用 symbol1 和 symbol2 。
import {symbol1 as alias, symbol2} from "filename";

//导入当前源文件同目录下的文件 x
import "./x" as x;
```

## 合约结构

```javascript
pragma solidity ^0.4.0;

contract SimpleStorage {
    //状态变量
    uint storedData;
    
    //函数
    function bid() public payable { 
        //...
    }  
}
```

## 数据位置特性详解

代码在执行前，一般会编译成指令。指令就是一个个逻辑，逻辑操作的是数据。代码，或者说业务，操作的其实是数据。非区块链中，代码操作的数据，一般会存到数据库中。

在区块链里，区块链本身就是一个数据库。如果你使用区块链标记物产的所有权，归属信息将会被记录到区块链上，所有人都无法篡改，以标明不可争议的拥有权。所以在区块链中编程中，有一个数据位置的属性用来标识变量是否需要持久化到区块链中。

### 1.数据位置的类型

`数据位置`，变量的存储位置属性。有三种类型，`memory`，`storage`和`calldata`。最后一种数据位置比较特殊，一般只有外部函数的参数（不包括返回参数）被强制指定为`calldata`。这种数据位置是只读的，不会持久化到区块链。一般我们可以选择指定的是`memory`和`storage`。

### 2.默认数据位置

#### 2.1参数及局部变量

默认的函数参数，包括返回的参数，他们是`memory`。而默认的局部变量是`storage`的。

```
pragma solidity ^0.4.0;
contract SimpleAssign {
    struct S{string a; uint b;}
    //默认参数是memory
    function assign(S s) internal {
        //默认的变量是storage的指针
        //Type struct MemoryToLocalVar.S memory is not implicitly convertible to wxpected type struct MemoryToLocalVar.S storage pointer.
        //S tmp = s;
    }
}
```

在上述示例中，`assign()`尝试将一根参数赋值给一根局部变量。具体的报错信息`Error:Type struct memory is not implicitly convertible to expected type storage pointer.`。

从报错可以看出，默认的参数是`memory`的，而默认的局部变量是`storage`的。

#### 2.2状态变量

状态变量，合约内生命的共有变量。默认的数据位置是`storage`。

```
prama solidity ^0.4.4;
contract StateVariable {
    struct S{string a; uint b;}
    //状态变量，默认是storage
    S s;
}
```

### 3.数据位置间相互转换

#### 3.1storage转换为storage

```
pragma solidity ^0.4.0;
contract StorageConvertToStorage {
    struct S{string a; uint b;}
    //默认是storage的
    S s;
    function convertStorage(S storage s) internal {
        S tmp = s;
        tmp.a = "Test";
    }
    function call() returns (string) {
        convertStorage(s);
        return s.a;//Test
    }
}
```

在上面的代码中，我们将传入的`storage`变量，赋值给另一个临时的`storage`类型的`tmp`时，并修改`tmp.a = "Test"`，最后我们发现合约的状态变量`s`也被修改了。

#### 3.2memory转换为storage

因为局部变量和状态变量的类型都可能是`storage`。所以我们要分开来说这两种情况。

##### 3.2.1memory赋值给状态变量

将一个`memory`类型的变量赋值给一个状态变量时，实际是将内存变量拷贝到存储中。

```
pragma solidity ^0.4.0;
contract MemoryConvertToStateVariable {
    struct S{sting a; uint b;}
    //默认是storage的
    S s; 
    function memoryToState(S memory tmp) internal {
        s = tmp;//从内存中赋值到状态变量中。
        //修改旧memory中的值，并不会影响状态变量
        tmp.a = "Test";
    }
    function call() returns(string) {
        S memory tmp = S("memory", 0);
        memoryToState(tmp);
        return s.a;
    }
}
```

通过上例，我们发现，在`memoryToState()`中，我们把`tmp`赋值给`s`后，再修改`tmp`值，并不能产生任何变化。赋值时，完成了值拷贝，后续他们不再有任何联系。

##### 3.2.2memory赋值给局部变量

由于在区块链中，`storage`必须是静态分配存储空间的。局部变量虽然是一个`storage`的，但它仅仅是一个`storage`类型的指针。如果进行这样的赋值，实际会产生一个错误。

```
pragma solidity ^0.4.0;
contract MemoryConvertToStateVariable {
    struct S{sting a; uint b;}
    //默认是storage的
    S s; 
    function memoryToState(S memory tmp) internal {
        s = tmp;//从内存中赋值到状态变量中。
        //修改旧memory中的值，并不会影响状态变量
        tmp.a = "Test";
    }
    function call() returns(string) {
        S memory tmp = S("memory", 0);
        memoryToState(tmp);
        return s.a;
    }
}
```

通过上面的代码，我们可以看到这样的赋值的确不被允许。你可以通过将变量`tmp`改为`memory`来完成这样的赋值。

#### 3.3storage转为memory

将`storage`转为`memory`，实际是将数据从`storage`拷贝到`memory`中。

```
pragma solidity ^0.4.0;
contrat StorageToMemory {
    struct S{string a; uint b;}
    S s = S("storage", 1);
    function storageToMemory(S storage x) internal {
        S memory tmp = x;//由Storage拷贝到memory中
        //memory的修改不影响storage
        tmp.a = "Test";
    }
    function call() returns (string) {
        storageToMemory(s);
        return s.a;
    }
}
```

在上面的例子中，我们看到，拷贝后对`tmp`变量的修改，完全不会影响到原来的`storage`变量。

#### 3.4memory转为memory

`memory`之间是引用传递，并不会拷贝数据。我们来看看下面的代码。

```
pragma solidity ^0.4.0;
contract MemoryToMemory {
    struct S{string a; uint b;}
    //默认参数是memory
    function memoryToMemory(S s) internal {
        S memory tmp = s;
        //引用传递
        tmp.a = "other memory"
    }
    function call() returns (string) {
        S memory mem = S("memory", 1);
        memoryToMemory(mem);
        return mem.a;//other memory
    }
}
```

在上面的代码中，`memoryToMemory()`传递进来一个`memory`类型的变量，在函数内将之赋值给`tmp`，修改`tmp`的值，发现外部的`memory`也被改为了`other memory`。



