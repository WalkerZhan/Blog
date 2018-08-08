# Solidity关键字 constant view pure

在Solidity中constant、view、pure三个函数修饰词的作用是告诉编译器，函数不改变/不读取状态变量，这样函数执行就可以不消耗gas了（是完全不消耗！），因为不需要矿工来验证。所以用好这几个关键词很重要，不言而喻，省gas就是省钱！

这三个关键词有什么区别和联系，简单来说，在Solidity v4.17之前，只有constant，后来有人嫌constant这个词本身代表变量中的常量，不适合用来修饰函数，所以将constant拆成了view和pure。view的作用和constant一模一样，可以读取状态变量但是不能改；pure则更为严格，pure修饰的函数不能改也不能读状态变量，否则编译通不过。

```javascript
pragma solidity ^0.4.21;

contract constantViewPure{
    string name;
    uint public age;
    
    function constantViewPure() public{
        name = "liushiming";
        age = 29;
    }
    
    function getAgeByConstant() public constant returns(uint){
        age += 1;  //声明为constant，在函数体中又试图去改变状态变量的值，编译会报warning, 但是可以通过
        return age;  // return 30, 但是！状态变量age的值不会改变，仍然为29！
    } 
    
    function getAgeByView() public view returns(uint){
        age += 1; //view和constant效果一致，编译会报warning，但是可以通过
        return age; // return 30，但是！状态变量age的值不会改变，仍然为29！
    }
    
    function getAgeByPure() public pure returns(uint){
        return age; //编译报错！pure比constant和view都要严格，pure完全禁止读写状态变量！
        return 1;
    }
}
```

