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

  코드 중첩이 깊다면 이해하기 힘들다. 각 레벨의 중첩마다 읽는 사람에게 추가 조건이라는 mental stack 을 push 한다.
  닫는 괄호는 만나면 stack 을 pop 하고 기존 조건을 기억하기 힘들다.
  
  다음은 간단한 예제이다.
  ```cpp
    if (user_result == SUCCESS) {
      if (permission_result != SUCCESS) {
        reply.WriteErrors("error reading permissions");
        reply.Done();
        return;
      }
      reply.WriteErrors("");
    } else {
      reply.WriteErrors(user_result);
    }

    reply.Done();
  ```

  첫 번째 닫는 괄호를 보면 당신은 다음과 같이 생각할 것이다.
  ' permission_result != SUCCESS 가 끝났구나!. 이제 permission_result == SUCCESS 이고 아직 user_result == SUCCESS 안이야!'

#### How Nesting Accumulates
  앞의 예를 수정하기 전에, 왜 저렇게 되었는지 이야기 해보자.

  처음엔, 코드는 단순했었다.

```cpp
  if (user_result == SUCCESS) {
    reply.WriteErrors("");
  } else {
    reply.WriteErrors(user_result);
  }
  reply.Done();
```

이 코드는 완벽히 이해 가능하다. 그러나 프로그래머가 두번째 operation을 추가했다.

  ```cpp
    if (user_result == SUCCESS) {
      if (permission_result != SUCCESS) {
        reply.WriteErrors("error reading permissions");
        reply.Done();
        return;
      }
      reply.WriteErrors("");
    } else {
      reply.WriteErrors(user_result);
    }

    reply.Done();
  ```

  이러한 코드는 일리가 있다 - 프로그래머는 넣어야할 새로운 코드 덩어리가 있고, 그것을 넣기 위한 쉬운 곳을 찾았을 뿐이다. 이 새로운 코드는 프로그래머 마음에 bolded 하다. 그리고 이 변화의 차이가 매우 clean 하다. 그래서 이것은 쉬운 change 로 보인다.

  그러나 다은사람이 나중에 이 코드를 보면 모든 context 는 사라져 버린다.

  - KEY IDEA: Look at your code from a fresh perspective when you’re making changes. Step back and look at it as a whole.

#### Removing Nesting by Returning Early

  이것 처럼 중첩된 코드는 실패케이스를 가능한한 빨리 리턴하여 제거할 수 있다.

  ```cpp
    if (user_result != SUCCESS) {
      reply.WriteErrors(user_result);
      reply.Done();
      return;
    }
    if (permission_result != SUCCESS) {
      reply.WriteErrors(permission_result);
      reply.Done();
      return;
    }
    
    reply.WriteErrors("");
    reply.Done();
  ```  
  
  이 코드는 1 level 중첩만 가진다. 그러나 더 중요한 것은 읽는 사람의 mental stack 에서 pop 할일이 없다. - 모든 if block 은 return으로 끝난다.


#### Removing Nesting Inside Loops

  조기 리턴이 항상 적용가능하지 않다. 다음 loop안의 nested code 를 보자.

```cpp
  for (int i = 0; i < results.size(); i++) {
    if (results[i] != NULL) {
      non_null_count++;
      if (results[i]->name != "") {
        cout << "Considering candidate..." << endl;
        ...
      }
    }
  }
```

루프 안에서, 조기 리턴에 대응하는 기술은 continue 이다.

```cpp
  for (int i = 0; i < results.size(); i++) {
    if (results[i] == NULL) continue;
    non_null_count++;

    if (results[i]->name == "") continue;
    cout << "Considering candidate..." << endl;
    
    ...
  }
```

pch comment: continue 내리는게 더 나은가? 한 줄이면 괄호 빼먹었다고 오해할수도 있으니 옆으로 쓰는 것이 나은가??

if (...) return; 과 같은 방식으로  if (...) continue; 가 loop 안에서 동작한다.


cotniue가 goto 처럼 bounce 하기 때문에 혼란스러운수 있다. 하지만 이 경우 loop의 각 iteration이 독립적이고, continue를 'skip over this item' 를 의미하는 것을 쉽게 알 수 있다.

### Can You Follow the Flow of Execution?

  이 챕터는 low-level 제어 흐름에 관한 것이다.: loops, condtion 그리고 다른 jumps 를 어떻게 읽게 쉽게 만드는 지. 하지만 당신의 프로그램의 흐름을 high-level 에서 생각해봐야만 한다. 이는 프로그램의 전체 실행 경로를 쉽게 따라갈수있게 하기 위해서다 - main에서 시작하여 끝날때 까지 마음속으로 코드를 밟아 나가야 한다.

  하지만 , 실제로, 프로그래밍 언어와 라이브러리는 코드가 'behinde the scenes' 에서 동작하도록 하여 흐름을 따르기 어렵게 만든다.

  |Programming construct|How high-level program flow gets obscured|
  |------|---|
  |threading|It’s unclear what code is executed when.|
  |signal/interrupt handlers|Certain code might be executed at any time.|
  |exceptions|Execution can bubble up through multiple function calls.|
  |function pointers & anonymous functions|It’s hard to know exactly what code is going to run because that isn’t known
at compile time|
  |virtual methods|object.virtualMethod() might invoke code of an unknown subclass|

  이러한 구조 중 몇몇은 매우 유용하다. 코드를 좀더 가독성 있게 만들고 덜 중복되게 한다. 그러나 우리는 때때로  나중에 코드를 볼 사람이 얼마나 어렵게 읽을 지 모른채 과도하게 이런 구조를 사용한다. 또한 이런 구조는 버그 추적을 매우 어렵게 한다.

  key 는 이런 구조의 비율이 너무 크지 않아야 한다. 이런 특성을 과용하면 코드를 추적하는 일을 Three-Card Monte 게임처럼 만들어 버린다.


### Summary
  코드 흐름을 읽기 쉽게 만드는 몇가지 방법이 있다.

  비교 구문을 쓸때에는 변하는 값을 왼쪽에, 좀더 stable 한 값을 오른쪽에 놓아라

  if/else block의 순서를 바꿀수 도 있다. 일반적으로 positive/easier/interesting case를 처음에 놓아라. 때때로 이런 criteria가 충돌이 일어나지만, 일반적으로 좋은 규칙이다.

  삼항 연산자, do/while 그리고 goto 같은 특정 구조는 종종 가독성이 떨어지는 결과를 낳는다. 거의 항상 clear 한 대안이 있으므로 사용하지 않는 것이 좋다.

  중첩된 코드는 따라가기 위해 더 많은 집중을 요구한다. 각 새로운 중첩은 mental stack에 push 될 좀 더 많은 context 를 만든다. 대신에, 이를 피하기 위해선형적인 코드를 추구하라.

  조기 리턴은 중첩을 제거하고 코드를 clean up 한다.  Guard statements 은 매우 유용하다
