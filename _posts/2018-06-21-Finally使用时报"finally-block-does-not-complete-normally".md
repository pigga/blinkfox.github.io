在Eclipse中使用try,catch,finally时会遇到警告<br/>
写道<br/>
```
finally block does not complete normally
```
 原因：<br/>
###### 1、不管try块、catch块中是否有return语句，finally块都会执行。
###### 2、finally块中的return语句会覆盖前面的return语句（try块、catch块中的return语句），所以如果finally块中有return语句，Eclipse编译器会报警告“finally block does not complete normally”。
###### 3、如果finally块中包含了return语句，即使前面的catch块重新抛出了异常，则调用该方法的语句也不会获得catch块重新抛出的异常，而是会得到finally块的返回值，并且不会捕获异常。
所以 尽量不要在finally内使用return。