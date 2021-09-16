## Chapter 6. Making Comments Precise and Compact

  - KEY IDEA: Comments should have a high information-to-space ratio..
    - 설명하지 말아야 하는 것.
    - 코딩을 수행하면서 머릿속에 있는 정보를 기록하지
    - 코드를 읽는 사람의 입장에서 필요한 정보가 무엇인지 유추하기
    
### Keep Comments Compact
  
  ```cpp
    // The int is the CategoryType.
    // The first float in the inner pair is the 'score',
    // the second is the 'weight'.
    typedef hash_map<int, pair<float, float> > ScoreMap;
  ```

  위 예제의 주석은 아래처럼 한줄이면 충분

  ```cpp
  // CategoryType -> (score, weight)
    typedef hash_map<int, pair<float, float> > ScoreMap;
  ```

  위 comments 내용으로 3줄 쓰기엔 공간낭비

### Avoid Ambiguous Pronouns

“Who’s on First?” 처러 대명사는 상황을 매우 혼란스럽게 만든다.

  ```
    // Insert the data into the cache, but check if it's too big first.
  ```

  위 문장에서 it 은 data 혹은 cache 를 의미할 수 있다. 결국 코드 나머지 부분을 읽어야 의미파악이 되는데 그러면 주석이 있는 이유가 무엇인가?

  조금이라도 혼동의 여지가 있으면 **대명사를 원래 명사로 대체**

  ```
    // If the data is small enough, insert it into the cache.
  ```

  혹은 

  ```
    // Insert the data into the cache, but check if the data is too big first.
  ```
### Polish Sloppy Sentences
  대개 주석을 명확히하는 작업은 간단히 하는 작업과 함께 이루어진다.

  ```
    # Depending on whether we've already crawled this URL before, give it a different priority.
  ```

  위 문장은 어느정도 괜찮지만 다음과 비교해보자

  ```
    # Give higher priority to URLs we've never crawled before.
  ```

  아래가 간단하고 짧고 직접적이다. 또한 방문하지 않은 url 에 높은 우선순위를 준다는 사실까지 설명.

### Describe Function Behavior Precisely

  ```
    // Return the number of lines in this file.
    int CountLines(string filename) { ... }
  ```

  line 을 정의하는 방법이 아래처럼 여러가지다.
  - "" (an empty file)  : 0 or 1
  - "hello" : 0 or 1
  - "hello\n" : 1 or 2
  - "hello\n world": 1 or 2
  - "hello\n\r cruel\n world\r": 2 or 3 or 4

  가장 단순한 구현은 \n 을 세는 것.( Unix 명령어 wc 와 이와 같이 작동)


  ```
    // Count how many newline bytes ('\n') are in the file.
    int CountLines(string filename) { ... }
  ```
  주석이 이전에 비해 많이 길어지지 않았지만 더 많은 정보를 담고 있다.

### Use Input/Output Examples That Illustrate Corner Cases
  주석 작성 시 신중히 선택된 입출력 예제는 천마디 말보다 가치있다.

  ```java
  // Remove the suffix/prefix of 'chars' from the input 'src'.
  String Strip(String src, String chars) { ... }
  ```

  위 예는 아래 두가지가 명확하지 않음
  
  - chars가 제거되야하는 정확한 부분 문자열을 의미하는가? 아니면 특정한 순서가 없는 문자의 집합을 의미하는가?
  - src 끝에 chars 가 여러번 있으면 어떻게 되는가?

  ```java
  // Example: Strip("abba/a/ba", "ab") returns "/a/"
  String Strip(String src, String chars) { ... }
  ```

  위 예는 Strip 기능 전체를 보여주지만, 아래의 간단한 예는 유용하지 않다.

  ```java
  // Example: Strip("ab", "a") returns "b"
  ```

  다음은 또 다른 예이다.

  ```cpp
  // Rearrange 'v' so that elements < pivot come before those >= pivot;
  // Then return the largest 'i' for which v[i] < pivot (or -1 if none are < pivot)
  int Partition(vector<int>* v, int pivot);
  ```

  위 주석은 아주 명확하지만 시각적으로 혼란스럽다.
  다음이 오히려 낫다.

  ```cpp
  // Example: Partition([8 5 9 8 2], 8) might result in [5 2 | 8 9 8] and return 1
  int Partition(vector<int>* v, int pivot);
  ```

  위에의 입출력에 관련해 몇 군데 짚고 넘어갈 부분이다.
  - 벡터 안에 존재하는 값을 pivot으로 사용하여 경계가 분할되는 방식을 설명.
  - 벡터가 중복된 값을 허용하는걸 보여주기 위해 중복된 값인 8을 포함
  - 결과값을 담은 벡터를 일부러 정렬하지 않음. 만약 정렬하면 잘못된 idea를 get.
  - 1이 return 되기 때문에, 혼동을 피하기 위해 1을 포함시키지 않음


### State the Intent of Your Code
  
  주석 달기는 나중에 읽을 사람에게 코드를 잘성하면서 당신이 생각하던 것을 이야기 하는 것이다.
  그러나 많은 주석이 코드의 동작을 설명하는데 그친다.

  ```cpp
  void DisplayProducts(list<Product> products) {
    products.sort(CompareProductByPrice);
    // Iterate through the list in reverse order
    for (list<Product>::reverse_iterator it = products.rbegin(); it != products.rend(); ++it)
      DisplayPrice(it->price);
  }
  ```

  서술만 하는 위 주석 보다 아래가 낫다.

  ```cpp
    // Display each price, from highest to lowest
    for (list<Product>::reverse_iterator it = products.rbegin(); ... )
  ```

  위 주석 자체적으로 틀린 것은 없지만 의도가 드러나지 않는다. 그러나 아래 주석은 작성자의 의도를 나타내며, 위에 sort 가 한번 되어 코드의 실제 동작이 그와 반대라는 사실을 더 잘 눈치채게 된다.



### “Named Function Parameter” Comments

  ```
  Connect(10, false);
  ```
  불리언 값이 무엇을 뜻하는지 불분명.
  
  파이썬 같은 언어는 아래와 같이 가능

  ```python
  def Connect(timeout, use_encryption): 
    ...
  # Call the function using named parameters
  Connect(timeout = 10, use_encryption = False)
  ```

  cpp 나 Java 에서는 아래처럼 가능

  ```cpp
  void Connect(int timeout, bool use_encryption) { ... }
  // Call the function with commented parameters
  Connect(/* timeout_ms = */ 10, /* use_encryption = */ false);
  ```

  불리언의 경우 주석을 앞에 놓아라 

  ```cpp
  // Don't do this!
  Connect( ... , false /* use_encryption */);
  // Don't do this either!
  Connect( ... , false /* = use_encryption */);
  ```

  암호화를 하라는 건지 말라는 건지 분명하지 않다.


### Use Information-Dense Words
  프로그래밍을 몇년간 하다보면 같은 문제가 지속적으로 반복된다는 사실을 알 것이다. 그 같은 패턴이나 관용구를 묘사하기 위한 단어나 문구를 사용하자.

  ```cpp
  // This class contains a number of members that store the same information as in the
  // database, but are stored here for speed. When this class is read from later, those
  // members are checked first to see if they exist, and if so are returned; otherwise the
  // database is read from and that data stored in those fields for next time.
  ```

  위와 같은 설명은 아래처럼 줄여 쓸수 있다.
  ```cpp
  // This class acts as a caching layer to the database.
  ```

  아래는 다른 예제이다.

  
  ```cpp
  // Remove excess whitespace from the street address, and do lots of other cleanup
  // like turn "Avenue" into "Ave." This way, if there are two different street addresses
  // that are typed in slightly differently, they will have the same cleaned-up version and
  // we can detect that these are equal
  ```

  위와 같은 설명은 아래처럼 줄여 쓸수 있다.
  ```cpp
  // Canonicalize the street address (remove extra spaces, "Avenue" -> "Ave.", etc.)
  ```

### Summary
  - 대명사가 여러 가지를 가리킬 수 있다면 사용하지 않는 것이 좋다
  - 가능한 정확하게 함수 동작을 설명하라
  - 신중하게 선택된 입출력으로 주석을 달아라
  - 자명한 세부사항 보다 high-level 의도를 서술하라
  - 불분명한 함수의 변수를 설명하기 위해 inline-comment 를 사용해라
  - 많은(다양한 x) 의미를 담는 단어를 사용하여 주석을 간단히 유지하라.