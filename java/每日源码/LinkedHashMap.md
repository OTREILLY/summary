### 每日源码系列——LinkedHashMap

1. LinkedHashMap Demo </br>
2. LinkedHashMap源码解读
3. LinkedHashMap与LRU

---

### 1. LinkedHashMap Demo

1）demo1: HashMap“无序”遍历

```
public static void main(String[] args) {
//    LinkedHashMap<String, String> map = new LinkedHashMap<>();
    Map<String, String> map = new HashMap<>();
    map.put("a", "1");
    map.put("f", "6");
    map.put("d", "4");
    map.put("e", "5");
    map.get("d");
    for(Map.Entry<String, String> entry: map.entrySet()){
      System.out.println(entry.getKey() + ": " + entry.getValue());
    }
//    "无序"遍历
//    a: 1
//    d: 4
//    e: 5
//    f: 6
  }

```

2）demo2: LinkedHashMap 插入顺序遍历
```
public static void main(String[] args) {
    LinkedHashMap<String, String> map = new LinkedHashMap<>();
//    Map<String, String> map = new HashMap<>();
    map.put("a", "1");
    map.put("f", "6");
    map.put("d", "4");
    map.put("e", "5");
    map.get("d");
    for(Map.Entry<String, String> entry: map.entrySet()){
      System.out.println(entry.getKey() + ": " + entry.getValue());
    }
//    "插入顺序"遍历
//    a: 1
//    f: 6
//    d: 4
//    e: 5
  }
```

3）LinkedHashMap 访问顺序遍历

```
  public static void main(String[] args) {
    LinkedHashMap<String, String> map = new LinkedHashMap<>(16, 0.75f, true);
//    Map<String, String> map = new HashMap<>();
    map.put("a", "1");
    map.put("f", "6");
    map.put("d", "4");
    map.put("e", "5");
    map.get("d");   //访问一个元素"d"
    for(Map.Entry<String, String> entry: map.entrySet()){
      System.out.println(entry.getKey() + ": " + entry.getValue());
    }
//    "访问顺序"遍历
//    a: 1
//    f: 6
//    e: 5
//    d: 4
  }

```

---

### 2. LinkedHashMap源码解读 (基于jdk1.8)

1) 数据结构： HashMap + 双向循环链表

![](https://github.com/OTREILLY/summary/blob/master/screenshots/linkedhashmap1.png)

![](https://github.com/OTREILLY/summary/blob/master/screenshots/linkedhashmap2.png)



2）get(Object o)



---

### 3. LinkedHashMap与LRU
