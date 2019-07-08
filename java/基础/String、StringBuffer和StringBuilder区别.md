<h4 align="center"> String、StringBuffer和StringBuilder区别</h4>
##### 1.String.intern()在JDK1.6之前和之后的变化

***

在JDK1.6中，intern()方法会把首次遇到的字符串实例复制到永久代中，返回的也是永久代中的引用，而在JDK1.7中，不再复制实例，而只是在常量池中记录首次出现的实例引用。

