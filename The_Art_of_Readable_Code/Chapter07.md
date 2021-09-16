# Part II Simplifying Loops and Logic

## Chapter 7. Making Control Flow Easy to Read

  - KEY IDEA: Make all your conditionals, loops, and other changes to control flow as “natural” as possible—written in a way that doesn’t make the reader stop and reread your code.
    
### The Order of Arguments in Conditionals
  
  둘 중 어떤 코드가 더 읽기 쉬운가?

  ```
  if (length >= 10)
  ```

  ```
  if (10 <= length)
  ```

  다음 예제에서는 어떤가 ?

  ```
  while (bytes_received < bytes_expected)
  ```

  ```
  while (bytes_expected > bytes_received)
  ```

  첫번째 코드가 읽기 편하다. 

  다음은 우리가 발견한 유용한 guideline 이다.

  |Left-hand side|Right-hand side|
  |------|---|
  |조건 문에 체크되는, 더 유동적인 value|값이 더 고정적인 비교대상 표현|

  이러한 가이드 라인은 일반적인 어순과 일치.



<!-- Todo Yoda 표기법은 유용한가 -->

### The Order of if/else Blocks


  ```cpp
  if (a == b) {
    // Case One ...
  } else {
    // Case Two ...
  }
  ```

    ```cpp
  if (a != b) {
    // Case One ...
  } else {
    // Case Two ...
  }
  ```

  위 경우를 선택하라.  이유는
  - 부정이 대신에 긍정을 선호해라. 
    - if (!debug) 대신에 if (debug) 사용
  - 간단한 case 를 먼저 처리해라. 그러면 if, else 구문을 한 화면에서 확인 할 수 있다.
  - 더 흥미롭거나 확실한 것을 선호하라.

  위 규칙이 충돌을 일으키는 경우가 있는데 대 부분 확실한 승자가 눈에 들어온다.

  ```
  if (!url.HasQueryParameter("expand_all")) {
    response.Render(items);
    ...
  } else {
    for (int i = 0; i < items.size(); i++) {
      items[i].Expand();
    }
  ...
  }
  ```
  
  expand_all 이 '분홍색 코끼리를 생각하지 마세요' 라는 말을 듣고 난후 분홍색 코끼리와 같다.

  pch comment: 긍정 표현 해도 생각 날거 같은데...




### The ?: Conditional Expression (a.k.a. “Ternary Operator”)
  삼항 연산자가 가독성에 미치는 영향은 논쟁의 대상. 
  옹호하는 측은 여러 줄에 걸쳐서 나타나틑 것을 한줄에 담는 좋은 방법이라 하지만 반대하는 사람은 오히려 읽기 혼란 스럽고 디버깅이 어렵다고 한다.

  ```cpp
    time_str += (hour >= 12) ? "pm" : "am";
  ```

  ```cpp
    if (hour >= 12) {
      time_str += "pm";
    } else {
      time_str += "am";
    }
  ```
  
  이 경우는 삼항연산자 사용이 좋아보인다

  하지만 다음과 같은 표현은 읽기 복잡하다.

  ```cpp
    return exponent >= 0 ? mantissa * (1 << exponent) : mantissa / (1 << -exponent);
  ```

  - KEY IDEA: Instead of minimizing the number of lines, a better metric is to minimize the time needed for someone to understand it.

  이런 코드는 if/else 구문이 코드를 더 자연스럽게 한다.

  ```cpp
    if (exponent >= 0) {
      return mantissa * (1 << exponent);
    } else {
      return mantissa / (1 << -exponent);
    }
  ```

  - ADVICE: By default, use an if/else. The ternary ?: should be used only for the simplest cases.



### Avoid do/while Loops

  ```cpp
    // Search through the list, starting at 'node', for the given 'name'.
    // Don't consider more than 'max_length' nodes.
    public boolean ListHasNode(Node node, String name, int max_length) {
      do {
        if (node.name().equals(name))
          return true;
        node = node.next();
      } while (node != null && --max_length > 0);

      return false;
    }
  ```
  if, while, for 문 과 달리 조건 읽은 방향이 다르고 많은 reader 가 코드를 두번 읽는다.

  do/while 을 제거하려고 중복된 코드를 사용하는건 좋지 않다.

  ```cpp
  // Imitating a do/while — DON'T DO THIS!
  body

  while (condition) {
    body (again)
  }
  ```

  다행히도 대부분의 do/while 이 while 로 쓰여질 수 있다.

  ```cpp
    // Search through the list, starting at 'node', for the given 'name'.
    // Don't consider more than 'max_length' nodes.
    public boolean ListHasNode(Node node, String name, int max_length) {
      while (node != null && max_length-- > 0) {
        if (node.name().equals(name)) return true;
        node = node.next();
      }
      return false;
    }
  ```

  do/while 사용을 피해야하는 다른 이유는 내부의 continue 구분이 혼란을 일으킬 수 있기 때문이다. 다음 코드는 어떻게 동작하는가?

  ```cpp
  do {
    continue;
  } while (false);
  ```

  계속 반복되는가 한 번만 실행되는가? 대부분의 프로그래머는 잠시 멈추고 생각을 해야 할 것이다.
  답은 한번만 실행된다이다.

  c++ 창시자인 Bjarne Stroustrup 가 이렇게 이야기 했다.

  - In my experience, the do-statement is a source of errors and confusion. … I prefer the condition “up front where I can see it.” Consequently, I tend to avoid do-statements.

  [스탭 오버 플로우에서 찾은 full version. From "The C++ Programming Language", 6.3.3:](https://stackoverflow.com/questions/994905/is-there-ever-a-need-for-a-do-while-loop)


### Returning Early from a Function

  어떤 코더는 함수가  multiple return statements 를 가지면 안된다고 믿는다. 이는 nonsense 이다. 
  
  ```
  public boolean Contains(String str, String substr) {
    if (str == null || substr == null) return false;
    if (substr.equals("")) return true;
    ...
  }
  ```

  이 함수를 'guard clauses' 없이 구현하면 매우 부자연스러울 것이다.

  single exit point 를 원하는 동기 중 하나는 함수 끝의 cleanup code 의 호출을 보장하기 위함이다.

  현대 언어는 이를 달성하기 위한 더 정교한 방법을 제공한다.

  |Language|Structured idiom for cleanup code|
  |------|---|
  |C++|destructors|
  |Java, Python|try finally|
  |Python|with|
  |C#|using|

  순수한 c 에서는 위와 같은 방법이 없다.(번역서 있다고 오타.) 
  cleanup 코드가 많은 큰 함수에서는 조기 리턴이 정확히 동작하기 어렵다.
  이런 경우 리팩토링이나 goto cleanup; 의 사용이 고려되어야 한다.


  pch comment: 이전 직장에서 CC 인증 받은 외부 암호화 모듈 사서 사용하는데.. c 기반으로 작성되어서 예제 함수들 마다 single return + goto 문의 향연 이였음...


### The Infamous goto

  goto를 쓰면 코드가 빠르게 손을 벗어서 follow 하기 어렵게 된다.
  하지만 많은 c 프로젝트에서 goto를 볼 수 있다. 특히 linux 커널!.

  goto의 가장 innocent 한 사용은 함수 끝의 signle exit 와 사용하는 것이다. 

  ```cpp
    if (p == NULL) goto exit;
    ...
  exit:
    fclose(file1);
    fclose(file2);
    ...
    return;
  ```

  위와 같은 경우는 큰 문제가 아니다.

  문제는 goto 가 여러개 사용되고 그들의 path 가 교차될때다. 특히 윗방향이면 real spaghetti 코드가 만들어 진다. 그리고 그것들은 반드시 구조적 loops 로 대체될수있다. 대부분, goto는 반드시 피해야한다.


### Minimize Nesting

#### How Nesting Accumulates

#### Removing Nesting by Returning Early

#### Removing Nesting Inside Loops

### Can You Follow the Flow of Execution?

### Summary



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