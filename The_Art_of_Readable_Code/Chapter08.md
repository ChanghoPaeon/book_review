## Chapter 8. Breaking Down Giant Expressions

  최근 연구에 따르면 우리 중 대부분은 한 번에 서너가지 정도만 생각 할 수 있다.

  - KEY IDEA: Break down your giant expressions into more digestible pieces.
    
### Explaining Variables

  표현을 쪼개는 가장 쉬운 방법은 subexpression을 캡쳐하는 추가적인 변수를 사용하는 것이다. 이 추가적인 변수를 때때로 '설명 변수(explaining variable)' 라 부른다

  ```python
    if line.split(':')[0].strip() == "root":
  ```

  
  ```python
    username = line.split(':')[0].strip()
    if username == "root":
  ```

### Summary Variables
  expression이 설명을 필요로 하지 않더라도, 새로운 변수에서 expression을 잡아내는게 유용할수 있다. 큰 크기의 코드를 작은 이름으로 바꾸어 더 쉽게 관리하고 생각하기 위한 목적을 가진 이런 변수를 요약 변수(summary variable) 라고 부르자

  ```javascript
    if (request.user.id == document.owner_id) {
      // user can edit this document...
    }

    ...

    if (request.user.id != document.owner_id) {
      // document is read-only...
    }
  ```

  ```javascript
    final boolean user_owns_document = (request.user.id == document.owner_id);

    if (user_owns_document) {
      // user can edit this document...
    }

    ...

    if (!user_owns_document) {
      // document is read-only...
    }
  ```

  큰 것 처럼 보이지는 않지만, if (user_owns_document)  가 읽기가 쉽고, user_owns_document 가 이 함수를 관통하여 고려되는 개념이란 것을 이야기 해준다.


### Using De Morgan’s Laws

  ```
  1) not (a or b or c) ⇔ (not a) and (not b) and (not c)
  2) not (a and b and c) ⇔ (not a) or (not b) or (not c)
  ```

  드 모르간의 법칙으로 불리언 표현을 간단하게 할 수 있다.

  ``` cpp
    if (!(file_exists && !is_protected)) Error("Sorry, could not read file.");
  ```

  ``` cpp
    if (!file_exists || is_protected) Error("Sorry, could not read file.");
  ```


### Abusing Short-Circuit Logic

  대부분의 프로그래밍 언어에서는 불리언 연산이 short-circuite evlautaion 을 수행한다. 예를 들면 if (a || b) 에서 a가 참이면 b가 true 인지 evaluate 하지 않는다. 이런 동작은 매우 handy 하지만, 때때로 오용 될 수 있다.


  ```cpp
    assert((!(bucket = FindBucket(key))) || !bucket->IsOccupied());
  ```
  ```cpp
    bucket = FindBucket(key);
    if (bucket != NULL) assert(!bucket->IsOccupied());
  ```

  위 두 코드는 동일한 작업을 수행한다. 아래 쪽이 두 줄로 늘어나긴 했지만 훨씬 이해하기 쉽다.

  - KEY IDEA: Beware of “clever” nuggets of code—they’re often confusing when others read the code later.

  이것이 short-circuite 을 피하라는 것을 의미하는가? 아니다. 다음과 같이 깔끔하게 사용되는 경우도 많다.

  ```cpp
    if (object && object->method()) ...
  ```

  여기 언급할만한 가치가 있는 새로운 idiom이 있다: 파이썬, 자바스크립트 그리고 루비 같은 언어에서는 or operator 가 그것의 argements 중 하나를 반환한다.

  ```perl
    x = a || b || c
  ```
  위 와 같은 코드는 first truthy value 를 반환한다.

### Example: Wrestling with Complicated Logic
  
  시작과 끝점을 가지고 있고, 겹치는지 여부를 검사하는 Range 구조체를 구현 예시르라 보자. (끝점은 포함하지 않는 반-개행)

  ```cpp
    struct Range {
      int begin;
      int end;
      // For example, [0,5) overlaps with [3,8)
      bool OverlapsWith(Range other);
    };
  ```

  ```cpp
    bool Range::OverlapsWith(Range other) {
      // Check if 'begin' or 'end' falls inside 'other'.
      return (begin >= other.begin && begin <= other.end) ||
              (end >= other.begin && end <= other.end);
    }
  ```

  단순히 두줄이지만 가능한 경우의 수는 2*2*2 = 8 가지 이다.

  위 코드는 [0, 2), [2, 4) 가 겹친다고 리턴하는 버그가 있다.

  ```cpp
    bool Range::OverlapsWith(Range other) {
      // Check if 'begin' or 'end' falls inside 'other'.
      return (begin >= other.begin && begin < other.end) ||
              (end > other.begin && end <= other.end);
    }
  ```

  위처럼 수정해도 또 버그가 있다. 바로 other 가 완전히 포함되는 경우이다.

  ```cpp
    return (begin >= other.begin && begin < other.end) ||
            (end > other.begin && end <= other.end) ||
            (begin <= other.begin && end >= other.end);
  ```

  너무 복잡 해 졌다.


#### Finding a More Elegant Approach
  이제 우리는 다른 접근 방법을 찾아야 한다. 간단한 문제에서 출발해 놀랍도록 꼬인 로직 조각을 얻었다. 보통 이것은 더 쉬운 길이 반드시 있다는 것의 신호다.

  하지만 좀더 elegant 한 해법은 창의성을 요구한다. 어떻게 할 수 있는가?? 한가지 기술은 이 문제를 '반대' 방향에서 풀수 있다는 것이다. 당신이 처한 상황에 빗대어 말하면, 배열을 역으로 순회하거나, 데이터 구조를 채우기 위해 forward 하는 것이 아니라 backward 하는 것을 의미한다.

  여기서 OverlapsWith 의 반대는 겹치지 않는것!

  1. 다른 범위가 현재 범위 시작보다 전에 끝난다
  2. 다른 범위가 현재 범위 끝난 후에 시작된다.

  훨씬 쉬운 문제로 바뀌었다.

  ```cpp
    bool Range::OverlapsWith(Range other) {
      if (other.end <= begin) return false; // They end before we begin
      if (other.begin >= end) return false; // They begin after we end
      return true; // Only possibility left: they overlap
    }
  ```

### Breaking Down Giant Statements


  ```javascript
  var update_highlight = function (message_num) {
    if ($("#vote_value" + message_num).html() === "Up") {
      $("#thumbs_up" + message_num).addClass("highlighted");
      $("#thumbs_down" + message_num).removeClass("highlighted");
    } else if ($("#vote_value" + message_num).html() === "Down") {
      $("#thumbs_up" + message_num).removeClass("highlighted");
      $("#thumbs_down" + message_num).addClass("highlighted");
    } else {
      $("#thumbs_up" + message_num).removeClass("highighted");
      $("#thumbs_down" + message_num).removeClass("highlighted");
    }
  };
```

개별 표현은 크지 않지만 모두 같이 있어서 하나의 커다란 statement 를 만든다.

다행히, 많은 부분이 동일하다. 이는 summary 변수를 extract 할 수 있음을 의미한다. (이것은 또한 DRY - Don't Repeat Yourself 원리를 보여준다 )

  ```javascript
    var update_highlight = function (message_num) {
      var thumbs_up = $("#thumbs_up" + message_num);
      var thumbs_down = $("#thumbs_down" + message_num);
      var vote_value = $("#vote_value" + message_num).html();
      var hi = "highlighted";

      
      if (vote_value === "Up") {
        thumbs_up.addClass(hi);
        thumbs_down.removeClass(hi);
      } else if (vote_value === "Down") {
        thumbs_up.removeClass(hi);
        thumbs_down.addClass(hi);
      } else {
        thumbs_up.removeClass(hi);
        thumbs_down.removeClass(hi);
      }
    };
  ```

  반드시 hi 변수를 만들 필요는 없지만, 그렇지 않으면 6번의 copy가 발생한다. 다음은 compelling benefits 다.

  - 타이핑 실수를 방지한다. (사실 수정전 예제에서 highighted라는 잘못된 철자를 사용되었었다.) 
  - 라인 너비를 좀 더 줄여 준다. 코드를 scan 하는게 쉽도록.
  - 클래스 명 변경이 필요하면 한 곳만 바꾸면 된다.

### Another Creative Way to Simplify Expressions

  ```cpp
  void AddStats(const Stats& add_from, Stats* add_to) {
    add_to->set_total_memory(add_from.total_memory() + add_to->total_memory());
    add_to->set_free_memory(add_from.free_memory() + add_to->free_memory());
    add_to->set_swap_memory(add_from.swap_memory() + add_to->swap_memory());
    add_to->set_status_string(add_from.status_string() + add_to->status_string());
    add_to->set_num_processes(add_from.num_processes() + add_to->num_processes());
    ...
  }
  ```

  코드가 길고 유사해 보이겠지만 실제로 완전이 같지 않다. 10초간 집중하면 각 라인이 다른 필드로 같은 일을 하는 것을 알 수 있다.

  매크로를 정의하여 다음과 같이 구현 할 수 있다

  ```cpp
    void AddStats(const Stats& add_from, Stats* add_to) {
      #define ADD_FIELD(field) add_to->set_##field(add_from.field() + add_to->field())
      ADD_FIELD(total_memory);
      ADD_FIELD(free_memory);
      ADD_FIELD(swap_memory);
      ADD_FIELD(status_string);
      ADD_FIELD(num_processes);
      ...
      #undef ADD_FIELD
    }
  ```

  매크로를 권장하는게 아님에 유의해라. 대개 매크로를 피하는데, 코드를 혼란스럽게 하고 subtle bugs 를 야기할 수 있기 때무이다. 그러나 때때로, 이 예제처럼, 가독성에 분명한 이점을 제공한다.

### Summary
  거대한 코드는 생각하기 어렵다.  이 챕터에서는 큰 코드를 작게 쪼개는 몇가지 방법을 알아보았다.

  그중 하나는 '설명 변수' 의 도입이다. 이 접근은 다음 세가지 이점을 가진다.
  - 거대한 표현을 쪼갠다
  - subexpression 을 간결한 이름으로 서술하여 코드를 문서화한다.
  - 코드의 중요 concept 을 identify 하는데 도움을 준다.

  또다른 방법은 드모르간 법칙을 사용하는 것이다.
  
  복잡한 논리 조건을 작은 statement로 나누는 것도 살펴 보았다. 이 방법이 항상 가능한 것이 아니다. 그럴땐 문제를 'negating' 하거나 목적의 'opposite' 를 고려하라.

  마지막으로 개별 표혀을 쪼개는 방법은 더큰 코드 블록을 쪼개는되에도 사용될 수 있다.


