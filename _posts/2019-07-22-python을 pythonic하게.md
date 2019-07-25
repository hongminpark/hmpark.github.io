---
layout: post
title: "python을 pythonic하게"
author: "Hongmin Park"
---

*이 글은 [programmers.co.kr의 파이썬을 파이썬답게](https://programmers.co.kr/learn/courses/4008) 무료 강의를 정리한 글입니다.*

파이썬에는 코딩을 줄여주는 다양한 함수들이 내장되어있는데,<br>
나는 여전히 C 스타일로 코드를 작성하고 있다.<br>
기본부터 뜯어고치기 위해 코딩을 간소화하는 파이썬의 문법들을 강의를 참고하여 정리하였다.<br>
설명은 별로 없고 이런 문법이 있다 정도의 나열글이다. <br>
좀 더 자세히는 위의 강의를 참고하자. 무료강의인데다가 글+문제로 된 강의이고 생각보다 아주 고퀄이다. (강추!)<br><br>

### 몫과 나머지
`a//b, a%n`<br>
`*divmod(a, b)`

함수와 unpacking을 같이 사용

### 10진법 변환기
`int(num, base=10)`

### n칸 기준 정렬
```python
string.ljust(n)
string.center(n)
string.rjust(n)
```

### string 모듈
```python
import string 

string.ascii_lowercase # 소문자 abcdefghijklmnopqrstuvwxyz
string.ascii_uppercase # 대문자 ABCDEFGHIJKLMNOPQRSTUVWXYZ
string.ascii_letters #대소문자 모두 abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ 
```

### zip 함수
iterable한 원소들을 묶어서 tuple로 리턴
```python
mylist = [ [1,2,3], [4,5,6], [7,8,9] ]
new_list = list(map(list, zip(*mylist))) # [[1,4,7], [2,5,8], [3,6,9]]
```

### map 함수
iterable 원소의 type을 일괄변환
```python
list1 = ['1', '100', '33']
list2 = list(map(int, list1)) # list2 = [1, 100, 33]
```
### join 함수
sequence 원소(스트링/정수 등)를 하나로 붙이기
```python
my_list = [1, 100, 33]
answer = ''.join(my_list) # 110033
```

### itertools
곱집합 구하기 
```python
import itertools

iterable1 = 'ABCD'
iterable2 = 'xy'
itertools.product(iterable1, iterable2,) # Ax Ay Bx By Cx Cy Dx Dy 
```

Combination/Permutation(순열과 조합)
```python
import itertools

pool = ['A', 'B', 'C']
print(list(map(''.join, itertools.permutations(pool)))) # 3개의 원소로 수열 만들기
print(list(map(''.join, itertools.permutations(pool, 2)))) # 2개의 원소로 수열 만들기
```

### collections
```python
import collections
my_list = [1, 2, 3, 4, 5, 6, 7, 8, 7, 9, 1, 2, 3, 3, 5, 2, 6, 8, 9, 0, 1, 1, 4, 7, 0]
answer = collections.Counter(my_list)

print(answer[1]) # = 4
print(answer[3])  # = 3
print(answer[100]) # = 0
```

### dictionary
dictionary를 다루다보면 key값이 없는 경우에 Exception이 raise되는 경우가 왕왕 있다.<br>
이 경우에 try로 예외처리를 할 필요 없이, dictionary에서 제공해주는 함수의 기능을 이용해보자.<br>

##### key에 해당하는 value가 없으면 KeyError 아닌 x를 리턴
```python
my_dict.get(key, x)
```
##### key를 제거하고 key의 value를 리턴. key가 없으면 default를 리턴. default도 설정하지 않았을 경우 raise KeyError 
```python
my_dict.pop(key[, default])
```
또한, dictionary도 iter할 경우가 많은데, 아래처럼 깔끔하게 iter하도록 하자.
```python
for key i my_dict:
    ~~~~

for key, value in my_dict.items():
    ~~~~
    
if "key" in my_dict:
    ~~~~
    
dict.keys()
dict.values()
dict.items()
```
##### dict 의 기본값을 세팅
```python
dict.setdefault(key, default)
```

### 이진탐색
코드를 짜다보면 이진탐색을 해야할 경우가 있다. (이진탐색: 정렬된 리스트에서 아이템 찾기, 빠름)<br>
매번 이진탐색을 할 수는 없는 노릇이니 파이썬에서 제공하는 bisect를 이용하자. 
```python
import bisect
mylist = [1, 2, 3, 7, 9, 11, 33]
print(bisect.bisect(mylist, 3))
```

### inf, infinite number
```python
min_val = float('-inf')
max_val = float('inf')
```

### 파일 입출력
파일 close 구문을 작성할 필요 없음, EOF 체크할 필요 없음. (with-as 구문해서 수행)<br>
소켓/HTTP에도 with-as 구문을 활요할 수 있음.
```python
with open('myfile.txt') as file:
  for line in file.readlines():
    print(line.strip().split('\t'))
```

### heap
항상 정렬된 데이터에서 삽입/삭제가 빈번할 경우 heap을 이용할 수 있다.<br>
python에서는 heap 자료구조를 위해 `heapq`라는 모듈과 `PriorityQueue`라는 클래스를 제공한다.<br>
모두 minheap으로 구현되어 있어 부모가 자식보다 작은 값을 가진다.<br>
`heapq`와 `PriorityQueue`는 기능이 조금 다르니 적절하게 사용하도록 하자.(heapq가 연산이 더 빠름)<br>
#### heapq
```python
import heapq

my_list = [13, 2, 1, 5, 10]
heapq.heapify(my_list) # 힙정렬 예시, [1, 2, 13, 5, 10]
heapq.heappop(my_list) # 가장 작은 값 제거, 1 return 후 [2, 13, 5, 10]이 됨 
heapq.heappush(my_list, 1) # heap을 유지한 채 item 넣기,  [1, 2, 13, 5, 10]이 됨 
```
#### PriorityQueue
```python
from queue import PriorityQueue

my_list = [13, 2, 1, 5, 10]
pq = PriorityQueue()

for val in my_list:
    pq.put_nowait(val) # 데이터삽입
    # pq.get_nowait()    # 가장작은 값 얻기
```
