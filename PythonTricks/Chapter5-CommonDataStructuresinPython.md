# 5장 파이썬의 일반 데이터 구조
일반적인 데이터 구조에 대한 좋은 책으로 The Algorithm Design Manual by Steven S. Skiena 를 권함

##5.1 딕셔너리, 맵, 해시테이블

#### dict -Your Go-To Dictionary: : 믿음직한 딕셔너리

```
phonebook = {
'bob': 7387,
'alice': 3719,
'jack': 7052,
}
squares = {x: x * x for x in range(6)}
>>> phonebook['alice']
3719
>>> squares
{0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

- 객체가 키로 사용 되려면 몇가지 제약사항을 만족해야함.
    - 수명동안 hash 값이 변경되지 않으며 ( __ hash __ )
    - 다른 객체와 비교 가능해야 함 ( __ eq __ )
    - hashable 한 객 체는 모드 같은 hash value 가져야 함
  
- 일반적인 경우 조회, 삽입, 갱신 및 삭제 의 Time Complexity 는 _O_(1)


#### collections.OrderedDict - Remember the Insertion Order of Keys: 키 삽입 순서 기억

```
>>> import collections
>>> d = collections.OrderedDict(one=1, two=2, three=3)
>>> d
OrderedDict([('one', 1), ('two', 2), ('three', 3)])
>>> d['four'] = 4
>>> d
OrderedDict([('one', 1), ('two', 2),
('three', 3), ('four', 4)])
>>> d.keys()
odict_keys(['one', 'two', 'three', 'four'])
``` 
  - 키의 삽입 순서 기억 
  - 표준 dict 도 Cpython 3.6 이상에서 키의 삽입 순서를 유지하지만 이것은 언어 스펙에 정의되지 않은 side effect

#### collections.defaultdict - Return Default Values for Missing Keys: 누락된 키의 기본값 반환
``` 
>>> from collections import defaultdict
>>> dd = defaultdict(list)
# Accessing a missing key creates it and
# initializes it using the default factory,
# i.e. list() in this example:
>>> dd['dogs'].append('Rufus')
>>> dd['dogs'].append('Kathrin')
>>> dd['dogs'].append('Mr Sniffles')
>>> dd['dogs']
['Rufus', 'Kathrin', 'Mr Sniffles']
```


#### collections.ChainMap - Search Multiple Dictionaries as a Single Mapping: 여러 딕셔너리를 단일 맵핑으로 검색
```
>>> from collections import ChainMap
>>> dict1 = {'one': 1, 'two': 2}
>>> dict2 = {'three': 3, 'four': 4}
>>> chain = ChainMap(dict1, dict2)
>>> chain
ChainMap({'one': 1, 'two': 2}, {'three': 3, 'four': 4})
# ChainMap searches each collection in the chain
# from left to right until it finds the key (or fails):
>>> chain['three']
3
>>> chain['one']
1
>>> chain['missing']
KeyError: 'missing'
```

#### types.MappingProxyType - A Wrapper for Making Read-Only Dictionaries: 읽기 전용 딕셔너리를 만들기 위한 래퍼 
```
>>> from types import MappingProxyType
>>> writable = {'one': 1, 'two': 2}
>>> read_only = MappingProxyType(writable)
# The proxy is read-only:
>>> read_only['one']
1
>>> read_only['one'] = 23
TypeError:
"'mappingproxy' object does not support item assignment"
# Updates to the original are reflected in the proxy:
>>> writable['one'] = 42
>>> read_only
mappingproxy({'one': 42, 'two': 2})
```

#### Dictionaries in Python: Conclusion
- 일반적인 매핑 타입은 dict 사용. 

##5.2 배열 데이터 구조
#### list - Mutable Dynamic Arrays : 가변 동적 배열
- 리스트지만 내부에서 '동적 배열' 로 구현 -> 요소 추가나 제거 시 메모리 할당/해제
- 객체면 요소가 될 수 있음 -> 서로 다른 종류의 타입이 섞이는 것이 가능 -> 데이터가 조밀하게 저장되지 못해 결과적으로 더 많은 공간이 소비
```
>>> arr = ['one', 'two', 'three']
>>> arr[0]
'one'
# Lists have a nice repr:
>>> arr
['one', 'two', 'three']
# Lists are mutable:
>>> arr[1] = 'hello'
>>> arr
['one', 'hello', 'three']
>>> del arr[1]
>>> arr
['one', 'three']
# Lists can hold arbitrary data types:
>>> arr.append(23)
>>> arr
['one', 'three', 23]
```   

#### tuple - Immutable Containers: 불변 컨테이너
- 불변객체
- 임의 타입을 요소러 추가 가능 -> 타입 지정 배열보다 공간을 더 차지

```
>>> arr = 'one', 'two', 'three'
>>> arr[0]
'one'
# Tuples have a nice repr:
>>> arr
('one', 'two', 'three')
# Tuples are immutable:
>>> arr[1] = 'hello'
TypeError:
"'tuple' object does not support item assignment"
>>> del arr[1]
TypeError:
"'tuple' object doesn't support item deletion"
# Tuples can hold arbitrary data types:
# (Adding elements creates a copy of the tuple)
>>> arr + (23,)
('one', 'two', 'three', 23)
```

#### array.array - Basic Typed Arrays: 기본적인 타입 지정 배열
- 지정된 C스타일 기본 데이터 타입을 담을 수 있는 메모리 효율적인 저장 공간. 
- 리스트 및 튜플보다 공간 효율적

```
>>> import array
>>> arr = array.array('f', (1.0, 1.5, 2.0, 2.5))
>>> arr[1]
1.5
# Arrays have a nice repr:
>>> arr
array('f', [1.0, 1.5, 2.0, 2.5])
# Arrays are mutable:
>>> arr[1] = 23.0
>>> arr
array('f', [1.0, 23.0, 2.0, 2.5])
>>> del arr[1]
>>> arr
array('f', [1.0, 2.0, 2.5])
>>> arr.append(42.0)
>>> arr
array('f', [1.0, 2.0, 2.5, 42.0])
# Arrays are "typed":
>>> arr[1] = 'hello'
TypeError: "must be real number, not str"
```

#### str - Immutable Arrays of Unicode Characters: 유니코드 문자의 불변 배열
- str 은 재귀적 데이터 구조 : 각 문자는 길이가 1인 str 객체 
- 단일 데이터 타입에 특화되어 있으므로 공간 효율적 -> 유니토크 텍스트를 저장하는 경우 이를 사용.
- 문자열을 수정하려고 하면 수정된 복사본이 만들어짐
```
>>> arr = 'abcd'
>>> arr[1]
'b'
>>> arr
'abcd'
# Strings are immutable:
>>> arr[1] = 'e'
TypeError:
"'str' object does not support item assignment"
>>> del arr[1]
TypeError:
"'str' object doesn't support item deletion"
# Strings can be unpacked into a list to
# get a mutable representation:
>>> list('abcd')
['a', 'b', 'c', 'd']
>>> ''.join(list('abcd'))
'abcd'
# Strings are recursive data structures:
>>> type('abc')
"<class 'str'>"
>>> type('abc'[0])
"<class 'str'>"
```

#### bytes - Immutable Arrays of Single Bytes: 단일 바이트의 불변 배열
-  Endian 은 신경 안써도 되는가 ?? 
```
>>> arr = bytes((0, 1, 2, 3))
>>> arr[1]
1
# Bytes literals have their own syntax:
>>> arr
b'\x00\x01\x02\x03'
>>> arr = b'\x00\x01\x02\x03'
# Only valid "bytes" are allowed:
>>> bytes((0, 300))
ValueError: "bytes must be in range(0, 256)"
# Bytes are immutable:
>>> arr[1] = 23
TypeError:
"'bytes' object does not support item assignment"
>>> del arr[1]
TypeError:
"'bytes' object doesn't support item deletion"
```

#### bytearray - Mutable Arrays of Single Bytes: 단일 바이트의 가변 배열

- bytes 와 달리 자유롭게 수정 가능
- bytearray 를 byte 로 변환 가능하나, _O_(n) 의 시간이 걸림
```
>>> arr = bytearray((0, 1, 2, 3))
>>> arr[1]
1
# The bytearray repr:
>>> arr
bytearray(b'\x00\x01\x02\x03')
# Bytearrays are mutable:
>>> arr[1] = 23
>>> arr
bytearray(b'\x00\x17\x02\x03')
>>> arr[1]
23
# Bytearrays can grow and shrink in size:
>>> del arr[1]
>>> arr
bytearray(b'\x00\x02\x03')
>>> arr.append(42)
>>> arr
bytearray(b'\x00\x02\x03*')
# Bytearrays can only hold "bytes"
# (integers in the range 0 <= x <= 255)
>>> arr[1] = 'hello'
TypeError: "an integer is required"
>>> arr[1] = 300
ValueError: "byte must be in range(0, 256)"
# Bytearrays can be converted back into bytes objects:
# (This will copy the data)
>>> bytes(arr)
b'\x00\x02\x03*'
```

#### 요점정리
- __여러 타입의 임의 객체 저장__ : list or tuple
- __숫자(정수 또는 부동소수점) 데이터가 있고 메모리 효율과 성능이 중요__ : array.array  사용해보고, NumPy, Pandas
- __유니코드 문자로 표시된 텍스트 데이터__ : 내장 str.  가변 문자열이 필요하면 list
- __연속적인 바이트 블록을 저장__ : bytes. 변경 필요하면 bytearray

## 5.3 레코드, 구조체, 데이터 전송 객체 
#### dict - Simple Data Objects: 간단한 데이터 객체
- 딕셔너리를 사용해서 구현하면 변경, 필드 추가 제거 가 자유롭지만 오타 등 버그 유발 가능성 높음

```
car1 = {
'color': 'red',
'mileage': 3812.4,
'automatic': True,
}
car2 = {
'color': 'blue',
'mileage': 40231,
'automatic': False,
}
# Dicts have a nice repr:
>>> car2
{'color': 'blue', 'automatic': False, 'mileage': 40231}
# Get mileage:
>>> car2['mileage']
40231
# Dicts are mutable:
>>> car2['mileage'] = 12
>>> car2['windshield'] = 'broken'
>>> car2
{'windshield': 'broken', 'color': 'blue',
'automatic': False, 'mileage': 12}
# No protection against wrong field names,
# or missing/extra fields:
car3 = {
'colr': 'green',
'automatic': False,
'windshield': 'broken',
}
```

#### tuple - Immutable Groups of Objects: 불변 객체 그룹
- 임의의 객체를 그룹화하는 가장 간단한 데이터 구조 
    - (https://docs.python.org/3/library/stdtypes.html#tuple)
- 변경 불가능
- CPython의 리스트보다 약간 적은 메모리를 차지하며 더 빨리 생성됨 
    - (https://github.com/python/cpython/blob/1a5856bf9295fa73995898d576e0bedf016aee1f/Include/tupleobject.h#L10-L34) (https://github.com/python/cpython/blob/b879fe82e7e5c3f7673c9a7fa4aad42bd05445d8/Include/listobject.h#L4-L41)
```
>>> import dis
>>> dis.dis(compile("(23, 'a', 'b', 'c')", '', 'eval'))
0 LOAD_CONST 4 ((23, 'a', 'b', 'c'))
3 RETURN_VALUE
>>> dis.dis(compile("[23, 'a', 'b', 'c']", '', 'eval'))
0 LOAD_CONST 0 (23)
3 LOAD_CONST 1 ('a')
6 LOAD_CONST 2 ('b')
9 LOAD_CONST 3 ('c')
12 BUILD_LIST 4
15 RETURN_VALUE
```

- 리스트에서 튜플로 전환하여 추가 성능을 쥐어짜려는 시도는 잘못된 접근 일수있다 (개인 comment: 암달의 법칙 및 Big _O_ 줄이기가 중요)
- 데이터 가죠오는 방법이 정수 인덱스 뿐이라 가독성에 영향을 줄 수 있음
- ad-hoc 구조. -> 실수로 인한 버그를 만들기 쉽다

```
# Fields: color, mileage, automatic
>>> car1 = ('red', 3812.4, True)
>>> car2 = ('blue', 40231.0, False)
# Tuple instances have a nice repr:
>>> car1
('red', 3812.4, True)
>>> car2
('blue', 40231.0, False)
# Get mileage:
>>> car2[1]
40231.0
# Tuples are immutable:
>>> car2[1] = 12
TypeError:
"'tuple' object does not support item assignment"
# No protection against missing/extra fields
# or a wrong order:
>>> car3 = (3431.5, 'green', True, 'silver')
```

#### Writing a Custom Class - More Work, More Control
- __init__ 및 __repr__ 작성 필요 
- 접근제어 세밀히 가능
- 비즈니스 로직과 동작을 추가하려는 경우 유용
```
class Car:
def __init__(self, color, mileage, automatic):
self.color = color
self.mileage = mileage
self.automatic = automatic
>>> car1 = Car('red', 3812.4, True)
>>> car2 = Car('blue', 40231.0, False)
# Get the mileage:
>>> car2.mileage
40231.0
# Classes are mutable:
>>> car2.mileage = 12
>>> car2.windshield = 'broken'
# String representation is not very useful
# (must add a manually written __repr__ method):
>>> car1
<Car object at 0x1081e69e8>
```
#### collections.namedtuple - Convenient Data Objects: 편리한 데이터 객체

- 변경 불가
- 클래스에 비해 메모리 사용량에 잇점
```
>>> from collections import namedtuple
>>> from sys import getsizeof
>>> p1 = namedtuple('Point', 'x y z')(1, 2, 3)
>>> p2 = (1, 2, 3)
>>> getsizeof(p1)
72
>>> getsizeof(p2)
72
```

- 딕셔너리를 네임드튜플로 바꾸면 코드의 의도가 좀 더 명확해짐(?)

```
>>> from collections import namedtuple
>>> Car = namedtuple('Car' , 'color mileage automatic')
>>> car1 = Car('red', 3812.4, True)
# Instances have a nice repr:
>>> car1
Car(color='red', mileage=3812.4, automatic=True)
# Accessing fields:
>>> car1.mileage
3812.4
# Fields are immtuable:
>>> car1.mileage = 12
AttributeError: "can't set attribute"
>>> car1.windshield = 'broken'
AttributeError:
"'Car' object has no attribute 'windshield'"
```

#### typing.NamedTuple - Improved Namedtuples: 개선된 네임드튜플
- python 3.6에 추가
- 타입 힌트 지원

```
>>> from typing import NamedTuple
class Car(NamedTuple):
color: str
mileage: float
automatic: bool
>>> car1 = Car('red', 3812.4, True)
# Instances have a nice repr:
>>> car1
Car(color='red', mileage=3812.4, automatic=True)
# Accessing fields:
>>> car1.mileage
3812.4
# Fields are immutable:
>>> car1.mileage = 12
AttributeError: "can't set attribute"
>>> car1.windshield = 'broken'
AttributeError:
"'Car' object has no attribute 'windshield'"
# Type annotations are not enforced without
# a separate type checking tool like mypy:
>>> Car('red', 'NOT_A_FLOAT', 99)
Car(color='red', mileage='NOT_A_FLOAT', automatic=99)
```


#### struct.Struct - Serialized C Structs: 직렬화된 C 구조체
- 파이썬 bytes 객체로 직렬화된 C 구조체와 파이썬 값 사이의 변환을 수행
- 주로 데이터 교환 형식으로 사용

```
>>> from struct import Struct
>>> MyStruct = Struct('i?f')
>>> data = MyStruct.pack(23, False, 42.0)
# All you get is a blob of data:
>>> data
b'\x17\x00\x00\x00\x00\x00\x00\x00\x00\x00(B'
# Data blobs can be unpacked again:
>>> MyStruct.unpack(data)
(23, False, 42.0)
```


#### types.SimpleNamespace - Fancy Attribute Access : 세련된 속성 접근
- python 3.3 추가
- 모든 키를 클래스 속성으로 노출 
- 대괄호 인덱스 문법대신 닷 속성 접근을 사용 할 수 있음 

```
>>> from types import SimpleNamespace
>>> car1 = SimpleNamespace(color='red',
... mileage=3812.4,
... automatic=True)
# The default repr:
>>> car1
namespace(automatic=True, color='red', mileage=3812.4)
# Instances support attribute access and are mutable:
>>> car1.mileage = 12
>>> car1.windshield = 'broken'
>>> del car1.automatic
>>> car1
namespace(color='red', mileage=12, windshield='broken')
```

#### 요점정리
- _2~3개의 필드_ : 필드 순서 기억 및 필드명 불필요시 튜플, ex) 3차원 공간의 (x, y, z)
- 불변 필드 필요 : 일반 튜플, collections.namedtuple, typing.NamedTuple
- _오타 발생하지 않도록 필드 이름 고정 필요_ : collections.namedtuple , typing.NamedTuple
- _간단하게 유지_ : 일반 dict
- _데이터 구조를 완전히 제어할 필요_ : @property getter/setter 사용하여 클래스 작성
- _객체에 동작을 추가_ : 클래스 작성 또는 collections.namedtuple, typing.NamedTuple
- _데이터를 빽빽하게 담아야 한다_ : struct.Struct

 
## 5.4 Sets and MultiSets

#### set - Your Go-To Set: 믿음직한 Set
- 변경가능(mutable), 요소를 동적으로 삽입/삭제
- dict 와 동일한 성능 특성 공유 ( backed by dict 라서)
- hash 가능한 객체면 set 에 저장 가능
```
>>> vowels = {'a', 'e', 'i', 'o', 'u'}
>>> 'e' in vowels
True
>>> letters = set('alice')
>>> letters.intersection(vowels)
{'a', 'e', 'i'}
>>> vowels.add('x')
>>> vowels
{'i', 'a', 'u', 'o', 'x', 'e'}
>>> len(vowels)
6
```
#### frozenset – Immutable Sets: 불변 Set
- set 의 불변버전
- hashable -> dictionary 키 또는 다른 set 의 요소로 사용가능
```
>>> vowels = frozenset({'a', 'e', 'i', 'o', 'u'})
>>> vowels.add('p')
AttributeError:
"'frozenset' object has no attribute 'add'"
# Frozensets are hashable and can
# be used as dictionary keys:
>>> d = { frozenset({1, 2, 3}): 'hello' }
>>> d[frozenset({1, 2, 3})]
'hello'
```
#### collections.Counter – Multisets: 멀티 Set
- 요소가 두번이상 나타날 수 있는 multi set (or bag)
- 포함된 횟수 추적 시 유용 (dic)
```
>>> from collections import Counter
>>> inventory = Counter()
>>> loot = {'sword': 1, 'bread': 3}
>>> inventory.update(loot)
>>> inventory
Counter({'bread': 3, 'sword': 1})
>>> more_loot = {'sword': 1, 'apple': 1}
>>> inventory.update(more_loot)
>>> inventory
Counter({'bread': 3, 'sword': 2, 'apple': 1})

>>> len(inventory)
3 # Unique elements
>>> sum(inventory.values())
6 # Total no. of elements
```
#### 요점정리
- set: 변경 가능 set
- frozenset : hashable -> dict 의 key, set 의 요소로 사용 가능 
- collections.Counter : multi-set  
## 5.5 스택(LIFO)
#### list - Simple, Built-In Stacks
- _O_(1) 시간에 push / pop 수행 (append(), pop() 메소드만 사용해야함)
- list가 동적 배열로으나, 저장소를 필요보다 많이 할당해 두므로, push / pop 수행시마다 크기 조정하는 것은 아님. 
- 위 사항이 연결 리스트 기반 구현보다 안정적이지 않기는 하지만, 요소로의 무작위 접근을 _O_(1) 에 수행하는 잇점이 있음
```
>>> s = []
>>> s.append('eat')
>>> s.append('sleep')
>>> s.append('code')
>>> s
['eat', 'sleep', 'code']
>>> s.pop()
'code'
>>> s.pop()
'sleep'
>>> s.pop()
'eat'
>>> s.pop()
IndexError: "pop from empty list"
```

#### collections.deque - Fast & Robust Stacks: 빠르고 강력한 스택
- _O_(1) 시간에 양쪽 어디에서든 추가, 삭제 할 수 있음 
- 이중연결리스트로 구현 임의 원소 접근은 _O_(n)
- 큐, 스택으로 모두 사용가능 
```
>>> from collections import deque
>>> s = deque()
>>> s.append('eat')
>>> s.append('sleep')
>>> s.append('code')
>>> s
deque(['eat', 'sleep', 'code'])
>>> s.pop()
'code'
>>> s.pop()
'sleep'
>>> s.pop()
'eat'
>>> s.pop()
IndexError: "pop from an empty deque"
```

#### queue.LifoQueue - Locking Semantics for Parallel Computing: 병렬 컴퓨팅을 위한 잠금 체계
- 동기방식으로 구현
- 잠금체계가 필요없으면 list 나 deque 사용 (lock 구현의 불필요한 부하)
```
>>> from queue import LifoQueue
>>> s = LifoQueue()
>>> s.put('eat')
>>> s.put('sleep')
>>> s.put('code')
>>> s
<queue.LifoQueue object at 0x108298dd8>
>>> s.get()
'code'
>>> s.get()
'sleep'
>>> s.get()
'eat'
>>> s.get_nowait()
queue.Empty
>>> s.get()
# Blocks / waits forever...
```

#### Comparing Stack Implementations in Python: 파이썬 스택 구현 비교
- list: 동적 배열
    - pros: 빠른 임의접근, _O_(1) (append / pop 만 사용)
    - cons: 가끔씩 크기 조정
- deque: 이중연결리스트
    - pros: 일관성있는 _O_(1) 
    - cons: _O_(n) 의 임의접근
    
#### 요점정리
- deque 가 빠른 범용 스택 구현
- list 를 사용할 수 있지만 성능 저하를 피하려면 append(), pop() 만 사용 

## 5.6 큐(FIFO)
#### list - Terribly Sloooow Queues: 끔찍하게 느린 큐
- 요소를 맨 앞에 삽입/삭제 시 다른 요소를 하나씩 변경 _O_(n)
```
>>> q = []
>>> q.append('eat')
>>> q.append('sleep')
>>> q.append('code')
>>> q
['eat', 'sleep', 'code']
# Careful: This is slow!
>>> q.pop(0)
'eat'
```

#### collections.deque - Fast & Robust Queues: 빠르고 강건한 큐
- 이중 연결리스트 구현
- 어느 끝에 추가/삭제 하더라도 _O_(1)
- 임의접근 _O_(n)

```
>>> from collections import deque
>>> q = deque()
>>> q.append('eat')
>>> q.append('sleep')
>>> q.append('code')
>>> q
deque(['eat', 'sleep', 'code'])
>>> q.popleft()
'eat'
>>> q.popleft()
'sleep'
>>> q.popleft()
'code'
>>> q.popleft()
IndexError: "pop from an empty deque"
```


#### queue.Queue - Locking Semantics for Parallel Computing

- 동기 방식 구현 -> 여러 생산자와 소비자에게 잠금 체계 제공

```
>>> from queue import Queue
>>> q = Queue()
>>> q.put('eat')
>>> q.put('sleep')
>>> q.put('code')
>>> q
<queue.Queue object at 0x1070f5b38>
>>> q.get()
'eat'
>>> q.get()
'sleep'
>>> q.get()
'code'
>>> q.get_nowait()
queue.Empty
>>> q.get()
# Blocks / waits forever...
```
#### multiprocessing.Queue - Shared Job Queues: 공유 작업 큐
- 큐안의 항목을 여러 작업자가 동시에 처리 할 수 있게 해주는 공유 작업 큐의 구현
- 단일 인터프리터 프로세스에서는 일부 병렬 실행을 못하게 제한하는 전역 인터프리터 잠금(GIL) 때문에 CPython 에서는 프로세스 기반 병렬화가 널리 사용
    - 이런 타입의 큐는 프로세스 경계를 넘어서 pickle 가능한 객체를 저장하고 전송 가능

```
>>> q.put('code')
>>> q
<multiprocessing.queues.Queue object at 0x1081c12b0>
>>> q.get()
'eat'
>>> q.get()
'sleep'
>>> q.get()
'code'
>>> q.get()
# Blocks / waits forever...
```
#### 요점정리
- list: 사용 x
- deque: 병럴처리 지원 불필요한 일반적인 상황에서 사용

## 5.7 우선순위 큐

- totally ordered set 으로 된 키가 있는 레코드 집합을 관리하는 컨테이너 구조
- 일반적으로 스케줄링 문제처리에 사용

#### list - Maintaining a Manually Sorted Queue
- 가장 작은 혹은 큰 항목 신속히 삭제 
- 새 항목 삽입 _O_(n)
- 삽입지점은 bisect.insort 를 사용하여 _O_(_log_ n) 에 찾을수 있지만 삽입 단계가 느려서 속도 개선 효과가 없다 
```
q = []
q.append((2, 'code'))
q.append((1, 'eat'))
q.append((3, 'sleep'))
# NOTE: Remember to re-sort every time
# a new element is inserted, or use
# bisect.insort().
q.sort(reverse=True)
while q:
next_item = q.pop()
print(next_item)
# Result:
# (1, 'eat')
# (2, 'code')
# (3, 'sleep')
```

#### heapq - List-Based Binary Heaps: 리스트 기반 이진 힙
- list 에 의해 뒷받침되는 이진 힙 구현
- _O_( _log_ n) 시간의 가장 작은 항목 삽입/추출
- 최소 힙 구현만 제공하기때문에 실용적인 우선순위 큐에서 일반적으로 요구하는 정렬 안정성과 다른 기능 보장하려면 추가작업 필요
    - (https://docs.python.org/3/library/heapq.html#priority-queue-implementation-notes)
```
import heapq
q = []
heapq.heappush(q, (2, 'code'))
heapq.heappush(q, (1, 'eat'))
heapq.heappush(q, (3, 'sleep'))
while q:
next_item = heapq.heappop(q)
print(next_item)
# Result:
# (1, 'eat')
# (2, 'code')
# (3, 'sleep')
```

#### queue.PriorityQueue - Beautiful Priority Queues: 아름다운 우선순위 큐
- 내부적으로 heapq 사용하고 동일한 시간/공간 복잡성
    - (https://docs.python.org/3/library/queue.html#queue.PriorityQueue)
- 동기방식이며 생산자/소비자를 위한 잠금 체계 제공  
```
from queue import PriorityQueue
q = PriorityQueue()
q.put((2, 'code'))
q.put((1, 'eat'))
q.put((3, 'sleep'))
while not q.empty():
next_item = q.get()
print(next_item)
# Result:
# (1, 'eat')
# (2, 'code')
# (3, 'sleep')
```
#### 요점정리
- PriorityQueue 추천
- PriorityQueue 의 잠금부하를 피하려면 heapq 사용