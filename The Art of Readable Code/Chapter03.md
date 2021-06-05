## Chapter 3. Names That Can’t Be Misconstrued

- key idae: 본인이 지은 이름을 '다른 사람들' 이 '다른 의미'로 해석할 여지가 있는지 철저히 확인해야 한다.

### Example: Filter()
  - ```cpp
    results = Database.all_objects.filter("year <= 2011")
    ``` 
    - "year <= 2011" 인 객체를 선택하는 것인가 혹은 제거하는 것인가? -> select, exclude 가 나은 대안    
    

### Example: Clip(text, length)
  - ```python
    # Cuts off the end of the text, and appends "..."
    def Clip(text, length):
        ...
    ```
    - 문단 끝에서 부터 거꾸로 length 만큼 제거 vs 문단을 처음부터 최대 length 만큼 잘라내기
    - length 보단 max_length 가 나은 선택이다.
    - length 의 기준은 무엇인가? 바이트 수 vs 문자의 수 vs 단어의 수

### Prefer min and max for (Inclusive) Limits
  
  - ```python
    # 의도 10개 초과의 품목을 구매하지 못하게 하는 코드   
    # >= 를 > 로 수정하여 fix 가능
    CART_TOO_BIG_LIMIT = 10

    if shopping_cart.num_items() >= CART_TOO_BIG_LIMIT:
        Error("Too many items in cart.")
    ```
    원인은 'up to' 인지 'up to and including' 인지 불분명한 변수명.

    ```python
    # 변수명을 수정한 코드
    MAX_ITEMS_IN_CART = 10

    if shopping_cart.num_items() > MAX_ITEMS_IN_CART:
        Error("Too many items in cart.")
    ````
  
### Prefer first and last for Inclusive Ranges
  - ```python
    print integer_range(start=2, stop=4)
    # Does this print [2,3] or [2,3,4] (or something else)?
    ```
    경계의 끝점이 포함된다면 다음과 같은 이름짓기가 좋다
    ```python
    set.PrintKeys(first="Bart", last="Maggie")
    ```

### Prefer begin and end for Inclusive/Exclusive Ranges

  - 한쪽 끝이 포함되지만 다른 한쪽긑은 포함되지 않는 범위가 편할때가 있다.
  - 아래 두개 중, 위 표현이 더 편하다.
    ```cpp
    PrintEventsInRange("OCT 16 12:00am", "OCT 17 12:00am")
    ```
    
    ```cpp
    PrintEventsInRange("OCT 16 12:00am", "OCT 16 11:59:59.9999pm")
    ```
  
  - '마지막 값을 방금 지남'을 의미하는 영어 단어가 없지만, c++와 표준라이브러리에서 관습적으로 begin/end 를 사용하므로 이를 따르자.

### Naming Booleans
  - 불리언은 true, fasle 가 각각 무엇을 의미해야하는지 명확해야함
  - 위험한 예
    ```cpp
    bool read_password = true;
    ``` 
    - 여러가지로 해석 가능 : 패스워드를 읽을 필요가 있다 vs 패스워드가 이미 읽혔다
    - is, has, can, should 와 같은 단어를 더하면 의미가 명확해 진다
  - 부정어 피하기
    ```cpp
    bool disable_ssl = false; 
    ``` 
    ```cpp
    bool use_ssl = true; 
    ``` 
  
### Matching Expectations of Users

- Example: get*()
  - 통상적으로 get* 은 가벼운(?) 접근자로 여겨지는 경우가 많다.
    아래는 getMean 보다 computMean 으로 변경하여 시간이 제법 걸릴 수 있음을 명시하여야 한다.
    ```java
    public class StatisticsCollector {
        public void addSample(double x) { ... }
            public double getMean() {
                // Iterate through all samples and return total / num_samples
        }
        ...
    }
    ```  

- Example: list::size()
  - 아래는 list::size() 가 O(n) 임을 모르고 계산된 값을 던져주는 것으로 오해했던 개발자가 작성. -> 해당 프로젝트에서 서버가 기어가다시피 느렸음
    ```cpp
    void ShrinkList(list<Node>& list, int max_size) {
        while (list.size() > max_size) {
            FreeNode(list.back());
            list.pop_back();
        }
    }
    ```  
  - spec을 확인하지 않은 개발자의 잘못이라고 이야기할 수 있지만, 다른 c++ stl 의 size() 가 constant time 임을 감안하면 일관되지 않은 이름짓기로 발생한 것으로도 생각할 수 있음.
  - 최근(이책이 저술된 시점) 에서는 c++ 표준 라이브러이에서 size() 는 O(1) 이어야 하는 규칙을 정함.    


### Example: Evaluating Multiple Name Candidates
  - 방문자가 많은 웹사이트틑 종종 어떤 변화가 비즈니스에 도움되는지 '실험'한다
    아래와 같은 구성 파일은 다른 실험을 위해 복붙되어야하는 상황이 발생한다
    ```json
    experiment_id: 100
    description: "increase font size to 14pt"
    traffic_fraction: 5%
    ...
    ```
    ```json
    experiment_id: 101
    description: "increase font size to 13pt"
    [other lines identical to experiment_id 100]
    ```
    이때 아래와 같은 변수명은 어떻게 짓는 것이 좋은가 ?
    ```json
    experiment_id: 101
    the_other_experiment_id_I_want_to_reuse: 100
    [change any properties as needed]
    ```

    -template
      - 모호성: 다른 template 을 사용 vs 자신이 template 인지 
    -resue
      - 모호성: 100번 실험 값 재사용 vs 100번 재사용 
    -copy
      - 모호성: 100번 복사 vs 100번재 복사물
    -inherit
    
    copy_experiment 나 inherit_from_experiment_id 가 좋은 선택으로 보임

### Summary
  - 의미가 오해되지 않는 이름이 최선의 이름이다.
  - 값의 상하한 혹은 경계값 관련 변수명은 포함/배제 여부가 드러나야 한다.
  - 불리언의 경우 is, has 등을 사용하면 명확해지고, disable_ssl 같은 부정어는 피하라.
  - 사람들이 특정한 단어를 일반적으로 어떻게 생각하는지 유의하라 (get, size)