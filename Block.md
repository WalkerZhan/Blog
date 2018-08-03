# Block

## 内存管理

block是Objective-C对象

### MRC

- block没有引用外部局部变量时，放在全局区中。
- block引用外部局部变量时，放在栈中。
- block声明为属性的时候要用copy修饰，不能用retain。用copy修饰栈中block会拷贝一个block放在堆中。用copy修饰全局区中block会拷贝一个block放在全局区中。用retain修饰的block不会改变block在内存中的位置。方法执行结束后会销毁栈中的block，之后再调用这个block会报空指针异常。

### ARC

- block没有引用外部局部变量时，放在全局区中。
- block引用外部局部变量时，放在堆中。
- blcok声明为属性的时候用最好用strong。copy也可以。

## 循环引用

block会对内部强指针变量强引用一次

```objective-c
// 循环引用
@property (nonatomic, strong) void(^block)(void);
_block = ^{
    NSLog(@"%@", self);
};
// 解决循环引用
__weak typeof(self) weakSelf = self;
_block = ^{
    NSLog(@"%@", weakSelf);
};
// 解决self提前释放后，block中调用self为nil
__weak typeof(self) weakSelf = self;
_block = ^{
    __strong typeof(weakSelf) strongSelf = weakSelf;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"%@", strongSelf);
    });
};
```

## 变量传递

- 局部变量，block是值传递
- 静态变量、全局变量、__block修饰的变量，block是指针传递





