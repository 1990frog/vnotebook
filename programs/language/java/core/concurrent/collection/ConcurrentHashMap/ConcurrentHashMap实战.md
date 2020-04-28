[TOC]

# 常用方法
## put
添加元素
## get
获取元素
## putIfAbsent
putIfAbsent方法主要是在向ConcurrentHashMap中添加键—值对的时候，它会先判断该键值对是否已经存在，如果没有值就添加值，如果存在就返回已有的值

等同于：
```java
if(!map.containsKey(key))
    return map.put(key,value);
else
    return map.get(key);
```
## replace
基于CAS，会返回一个boolean值，代表操作是否成功
# 组合操作并不保证线程安全

```java
/**
 * 描述：
 * 组合操作并不保证线程安全
 */
public class OptionsNotSafe implements Runnable {

    private static ConcurrentHashMap<String, Integer> scores = new ConcurrentHashMap<String, Integer>();

    public static void main(String[] args) throws InterruptedException {
        scores.put("小明", 0);
        Thread t1 = new Thread(new OptionsNotSafe());
        Thread t2 = new Thread(new OptionsNotSafe());
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(scores);
    }

    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            // 线程不安全
//            Integer score = scores.get("小明");
//            Integer newScore = score +1;
//            scores.put("小明",newScore);

            // 线程安全，CAS聚合操作
            while (true) {
                Integer score = scores.get("小明");
                Integer newScore = score + 1;
                boolean b = scores.replace("小明", score, newScore);
                if (b) {
                    break;
                }
            }
        }

    }
}
```

