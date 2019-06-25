<h4 align="center"> HashMap,ConcurrentHashMap,LinkedHashMap分析</h4>



##### 1.LinkedHashMap实现LRU缓存时用KeySet遍历出现ConcurrentModificationException异常

***

因为LinkedHash的accessOrder设为true后，get()操作也会引起modCount++,相当于遍历过程中还在修改map，调用LinkedHashMap$LinkedHashIterator.nextNode()时会导致modCount != expectedModCount，所以会报异常（待研究）

