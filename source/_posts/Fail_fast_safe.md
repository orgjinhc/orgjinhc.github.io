## fail-fast

```java
/**
 * fail-fast
 *     java.util包下集合类在用迭代器遍历一个集合对象时，遍历过程中对集合对象的结构进行了修改（增加、删除），则会抛出Concurrent Modify Exception
 *     原因：迭代器遍历集合时，使用一个modCount变量来记录当遍历过程中集合对象结构发生了变化
 *          使用hashNext.next()就会检测modCount和expected值是否相等，否则抛出cme
 *     modCount:列表在结构上被修改的次数
 */
@Slf4j
public class failFast {

    public static void main(String[] args) {
        final List<Integer> list = new ArrayList();
        list.add(1);
        Iterator<Integer> iterator = list.iterator();
        new Thread(new Runnable() {
            public void run() {
                modify(list);
            }
        }).start();

        while (iterator.hasNext()) {
            Integer next = iterator.next();
            log.info("next:[{}]", next.toString());
        }
    }

    private static void modify(List<Integer> list) {
        list.remove(list.size() - 1);
    }
}
```

## ArrayList

![结构](http://lion-heart.online/blog/2020-01-28-020958.png)

## fail-safe

```java
/**
 * fail-safe
 * java.util.concurrent包下集合类都是安全失败,在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。
 * 原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Cme
 */
@Slf4j
public class failSafe {
    public static void main(String[] args) {
        final ConcurrentHashMap<Integer, Integer> map = new ConcurrentHashMap<Integer, Integer>();
        map.put(1, 1);
        map.put(2, 1);
        map.put(3, 1);

        new Thread(new Runnable() {
            public void run() {
                modify(map);
            }
        }).start();

        Iterator<Map.Entry<Integer, Integer>> iterator = map.entrySet().iterator();
        while (iterator.hasNext()) {
            log.info("next:[{}]", iterator.next());
        }
    }

    private static void modify(ConcurrentMap<Integer, Integer> map) {
        map.remove(3);
    }
}
```