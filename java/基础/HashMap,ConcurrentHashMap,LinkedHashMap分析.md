<h2 align="center"> HashMap,ConcurrentHashMap,LinkedHashMap分析</h2>

#### 1.LinkedHashMap实现LRU缓存时用KeySet遍历Map出现ConcurrentModificationException异常

***

```java
/**
 * @author fu
 * @date 2019/6/25 - 18:48
 */

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * 利用继承LinkedHashMap实现LRU缓存
 */
public class LRUcache<K, V> extends LinkedHashMap<K, V> {
    private  final int MAX_SIZE;

    public LRUcache(int size){
   		//LinkedHashMap一种构造方法，最后一个参数为true表示依据访问顺序排序
        super((int) Math.ceil(size / 0.75) + 1, 0.75f, true);
        MAX_SIZE = size;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > MAX_SIZE;//如果数量超过MAX_SIZE，删除最久没访问的元素
    }

    public static void main(String[] args) {
        LRUcache<String,Integer> cache = new LRUcache<>(3);
        cache.put("1", 1);
        cache.put("2", 1);
        cache.put("3", 1);
        cache.get("1");
        for (Map.Entry<String, Integer> entry : cache.entrySet()) {
            System.out.println(entry);
        }
        cache.put("4", 1);
        for (Map.Entry<String, Integer> entry : cache.entrySet()) {
            System.out.println(entry);
        }
        //遍历过程中不能调用get(),主要原因：LinkedHash的accessOrder设为true后，
        //get()操作也会引起modCount++,导致modCount != expectedModCount，
        //相当于遍历过程中还在修改map，所以报异常
        for (String s : cache.keySet()) {
            System.out.println(s + cache.get(s));
        }
    }
}
```



因为LinkedHash的accessOrder设为true后，get()操作也会引起modCount++,相当于遍历过程中还在修改map，调用LinkedHashMap$LinkedHashIterator.nextNode()时会导致modCount != expectedModCount，所以会报异常，而使用entrySet遍历没有调用get()方法，正常遍历。

