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
`a//b, a%n`
`*divmod(a, b)`

함수와 unpacking을 같이 사용

### 10진법 변환기
`int(num, base=10)`

### n칸 기준 정렬
`string.ljust(n)`
`string.center(n)`
`string.rjust(n)`

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

