---
layout: post
title: "Java Cheat Sheet for Algorithms"
author: "Hongmin Park"
---

파이썬으로 코딩테스트를 준비하고 있다. 그런데 모 기업에서 코딩테스트 언어로 `C, C++, Java`만 지원하겠다는 
청천벽력같은 소리를 들었다. 
일단 되든 안되든 지푸라기라도 잡아야될 것 같아서 그동안 풀었던 문제를 Java로 풀어보면서 알고리즘 구현에 
자주 사용되는 함수/클래스/객체 등등을 정리해보려 한다. 

### Hash
파이썬에서 dictionary로 구현하던 것을 자바에서는 `HashMap`을 사용한다. 
```java
HashMap<String, Integer> hm = new HashMap<>();
hm.put(key, value);
hm.put(key, hm.getOrDefault(key, 0) + 1);
hm.get(key);
for (String key: hm.keySet()) {
    if (part_.get(key) != 0) answer = key;
}
```

### Array
```java
String[] a;
Arrays.sort(a);
```

### Stream
python에서 많이 사용하던 `lambda, list comprehension, map, reduce, filter` 와 같은 함수와 기능들을 
java에서도 비슷하게 구현할 수 있다. java8에서부터 추가된 `Stream API`를 사용하자. 
자세한 내용은 [이 글](https://www.geeksforgeeks.org/stream-in-java/)에 센스있게 정리되어 있다.
```Java
// map
List<Integer> number = Arrays.asList(2,3,4,5);
List<Integer> square = number.stream().map(x -> x*x).collect(Collectors.toList());

// filter
List<String> names = Arrays.asList("Reflection","Collection","Stream");
List<String> filtered = names.stream().filter(x -> x.startswith("R")).collect(Collectors.toList());
```
