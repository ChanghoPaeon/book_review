## Chapter 9. Variables and Readability

  이 장에서는 변수의 엉뚱한 사용이 코드를 얼마나 이해하기 어렵게 만드는지 봄.

  1. 더 많은 변수는, 기억하고 추적하기 어렵게 만든다
  2. 더 큰 변수의 scope은 기억하고 추적하는 시간을 더 길게 한다
  3. 더 잦은 변수의 변경은 현재 값을 추적하기 더 어렵게 만든다.

    
### Eliminating Variables

  앞 장에서는 거대한 표현을 쪼개어 가독성에 도움을 주는 변수 도입에 대해 알아보았다.

  이번 장에서는 도움이 되지 않는 변수를 제거하는 법을 알아보자.


#### Useless Temporary Variables
  
  ```python
  now = datetime.datetime.now()
  root_message.last_view_time = now
  ```

  now 변수는 필요가 없다
  - 복잡한 표현을 쪼개지 않음
  - 명확성을 더해 주지 않음
  - 오직 한번만 사용. 즉 중복코드를 제거하지 않음

  ```python
  now =   root_message.last_view_time = now
  ```
#### Eliminating Intermediate Results

  ```javascript
    var remove_one = function (array, value_to_remove) {
      var index_to_remove = null;
      for (var i = 0; i < array.length; i += 1) {
        if (array[i] === value_to_remove) {
          index_to_remove = i;
          break;
        }
      }
      if (index_to_remove !== null) {
        array.splice(index_to_remove, 1);
      }
    };
  ```

  index_to_remove 변수는 값을 holding 하는 용도로만 사용되었다. 이런 변수는 바로 결과를 핸들링하여 제거할 수 있다.

  ```javascript
    var remove_one = function (array, value_to_remove) {
      for (var i = 0; i < array.length; i += 1) {
        if (array[i] === value_to_remove) {
          array.splice(i, 1);
          return;
      }
      }
    };
  ```
  일반적으로 **가능한한 빨리 task를 완료하는 것**이 좋은 전략이다.

#### Eliminating Control Flow Variables

  종종 다음과 같은 패턴을 볼 수 있다.

  ```cpp
    boolean done = false;
      while (/* condition */ && !done) {
        ...
        if (...) {
          done = true;
          continue;
      }
    }
  ```

  done 과 같은 변수를 control flow variables 라 한다. 

  
  ```cpp
    while (/* condition */) {
      ...
      if (...) {
        break;
      }
    }
  ```
  


### Shrink the Scope of Your Variables

  전역 변수를 피하라 라는 충고를 들었응ㄹ 것이다. keep track 하기에 어렵기 때문에 좋은 충고이다. 또한 지역 변수와 충돌이나서 name space 가 polluting 될수 있고, 지역변수를 사요하려고 했는데 전역변수를 수정하는 실수를 할때가 있다. 반대도 마찬가지.

  사실, 전역 변수뿐만 아니라 모든 변수의 scope을 줄이는 것은 좋은 아이디어다.

  - KEY IDEA: Make your variable visible by as few lines of code as possible.

  ```cpp
    class LargeClass {
      
      string str_;

      void Method1() {
        str_ = ...; //처음 str_ 으로 뭔가 하는 곳
        Method2();
      }

      void Method2() {
        // Uses str_
      }

      // Lots of other methods that don't use str_ ...
    };
  ```

  ```cpp
    class LargeClass {
      void Method1() {
        string str = ...;
        Method2(str);
      }

      void Method2(string str) {
        // Uses str
      }

      // Now other methods can't see str.
    };
  ```

  class 멤버 접근을 제한하기 위한 다른 방법은, 가능한한 많은 methods 를 static 으로 만들어라.

  또다른 방법은 큰 class 를 작은 class 들로 나누어라. 이 젖ㅂ근은 작은 클래스들이 서로 isolated 되어있으면 도움이 된다.

  커다란 파일이나 커다란 함수를 작은 것으로 나눌때도 마찬가지다. 이는 데이터간 isolate 하는데 목적이 있다.

  하지만 언어별로 scope 에 대한 서로 다른 rule 을 가지고 있기에 변수 scope 에 연관된 흥미로운 법칙을 짚어 보고자 한다.

#### if Statement Scope in C++
  
  ```cpp
    PaymentInfo* info = database.ReadPaymentInfo();
    if (info) {
      cout << "User paid: " << info->amount() << endl;
    }
  ```

  위 예제를 보면 info 가 if 문 밖에서도 사용될지 궁금해할것.  만약 if문 안에서만 사용되면, c++ 에서는 조건문 안에서 define 할 수 있다.

  ```cpp
    if (PaymentInfo* info = database.ReadPaymentInfo()) {
      cout << "User paid: " << info->amount() << endl;
    }
  ```

#### Creating “Private” Variables in JavaScript
  
  함수 하나에만 사용되는 persistent variable 를 가정하자.

  ```javascript
    submitted = false; // Note: global variable
      var submit_form = function (form_name) {
      if (submitted) {
        return; // don't double-submit the form
      }
      ...
      submitted = true;
    };
  ```

  submitted 는 읽는 사람에게 고민을 안긴다. submit_form 이 해당 변수를 사용하는 유일한 함수로 보이지만 확실히 알 수 없다. 다른 자바스크립트 파일에서 다른 목적으로 submitted 을 사용할 수도 있다. (pch comment: IDE 가 발달하여... 이런일이 생길까요?)

  클로저에 해당 변수를 wrapping 해서 이런 이슈를 막을 수 있다.

  ```javascript
    var submit_form = (function () {
      var submitted = false; // Note: can only be accessed by the function below
        return function (form_name) {
          if (submitted) {
            return; // don't double-submit the form
          }
          ...
      submitted = true;
      };
    }());
  ```

  (이런 테크닉은 JavaScript: The Good Parts by Douglas Crockford [O’Reilly, 2008] 에서 볼 수 있다.)


#### JavaScript Global Scope
  
  var 를 생략하면 해당 변수는 global scope 에 들어가서, 모든 자바스크립트 파일과 <script> 블록에서 접근 가능하다.

  ```javascript
  <script>
    var f = function () {
        // DANGER: 'i' is not declared with 'var'!
        for (i = 0; i < 10; i += 1) ...
    };

    f();
  </script>
  ```
  
  이 코드는 i 를 global scope 에 넣고 있다. 그래서 아래와 같은 블록에서 접근 가능하다.

  ```javascript
  <script>
    alert(i); // Alerts '10'. 'i' is a global variable!
  </script>
  ```

  이러한 사실을 모로는 프로그래머는 그의 컴퓨터가 해킹당했거나 RAM이 고장났다고 결론 내릴 수 있다.

  일반적인 'best practice' 는 **항상 var 키워드와 변수 정의하기**이다. 이는 변수의 범위를 innermost 하게 제한한다.


#### No Nested Scope in Python and JavaScript
  
  c++ 나 java 같은 언어는 block scpoe 을 가진다.

  ```cpp
    if (...) {
      int x = 1;
    }

    x++; // Compile-error! 'x' is undefined.  
  ```

  하지만 파이썬이나 자바스크림트 같이 block 안에서 정의된 변수가 전체 함수로 흘러 나온다

  ```python
    # No use of example_value up to this point.
    if request:
      for value in request.values:
        if value > 0:
          example_value = value
          break
    
    for logger in debug.loggers:
      logger.log("Example:", example_value) 
  ```


  위 예는 심지어 버그도 있다. (example_value 가 for 문 안에서 설정되지 않으면 logger 에서 "NameError: ‘example_value’ is not defined". 를 일으킴) 

  ```python
    example_value = None

    if request:
      for value in request.values:
        if value > 0:
          example_value = value
          break

    if example_value:
      for logger in debug.loggers:
        logger.log("Example:", example_value)
  ```

  위와 같이 버그를 없앤 코드에, '중간 결과 삭제하기' 를 적용하면 ,

  ```python
    def LogExample(value):
      for logger in debug.loggers:
        logger.log("Example:", value)

    if request:
      for value in request.values:
        if value > 0:
          LogExample(value) # deal with 'value' immediately
          break
  ```

#### Moving Definitions Down


```python
  def ViewFilteredReplies(original_id):
    filtered_replies = []
    root_message = Messages.objects.get(original_id)
    all_replies = Messages.objects.select(root_id=original_id)

    root_message.view_count += 1
    root_message.last_view_time = datetime.datetime.now()
    root_message.save()

    for reply in all_replies:
      if reply.spam_votes <= MAX_SPAM_VOTES:
        filtered_replies.append(reply)
    return filtered_replies
```

```python
  def ViewFilteredReplies(original_id):
    root_message = Messages.objects.get(original_id)
    root_message.view_count += 1
    root_message.last_view_time = datetime.datetime.now()
    root_message.save()

    all_replies = Messages.objects.select(root_id=original_id)
    filtered_replies = []
    for reply in all_replies:
      if reply.spam_votes <= MAX_SPAM_VOTES:
        filtered_replies.append(reply)
    
    return filtered_replies
```


### Prefer Write-Once Variables

  수많은 변수가 사용되면 프로그램을 읽기가 어려워 지는 것에 대해 논의하였다. 끊임없이 변하는 변수는 생각을 더욱 어렵게 한다. 그런 변수를 추적하는 것은 추가적인 어려움을 더한다.

  이 문제를 해결하기 위해, 이상한 것처럼 들리는 것을 제안한다.: 한번만 할당되는 변수를 선호하라,

  영원히 고정된 변수는 생각하기 쉽다. 다음과 같은 변수는 코드를 읽는 사람에게 추가적인 생각을 요구하지 않는다.
  
  ```
    static const int NUM_THREADS = 10;
  ```

  변수의 값을 한 번만 할당하게 할 수 없더라도, 변수 변경이 적은 곳에서 일어나는 것은 도움이 된다.

  - KEY IDEA: The more places a variable is manipulated, the harder it is to reason about its current value

### A Final Example

  ```javascript
  <input type="text" id="input1" value="Dustin">
  <input type="text" id="input2" value="Trevor">
  <input type="text" id="input3" value="">
  <input type="text" id="input4" value="Melissa">
  ...
  ```

  ```javascript
    var setFirstEmptyInput = function (new_value) {

      var found = false;
      var i = 1;
      var elem = document.getElementById('input' + i);

      while (elem !== null) {
        if (elem.value === '') {
          found = true;
          break;
        }

        i++;
        elem = document.getElementById('input' + i);
      }

      if (found) elem.value = new_value;

      return elem;
    };
  ```

  ```javascript
    var setFirstEmptyInput = function (new_value) {
      for (var i = 1; true; i++) {
        var elem = document.getElementById('input' + i);
        if (elem === null)
          return null; // Search Failed. No empty input found.

        if (elem.value === '') {
          elem.value = new_value;
          return elem;
        }
      }
    };
  ```


### Summary

  - 변수 제거하기. 특히 결과를 즉시 처리하여 중간 결과를 제거하는 것을 알아보았다.
  - 가능한한 각 변수의 범위를 줄이기. 각 변수를 코드의 제잉ㄹ 적은 line이 볼 수 있는 곳으로 옮겨라.
  - 오직 한번만 write 되는 변수를 선호하기. 한 번만 할당되는 변수는 코드를 쉽게 이해하게 한다.
