## Chapter 5. Knowing What to Comment

  - KEY IDEA: The purpose of commenting is to help the reader know as much as the writer did.
    - 설명하지 말아야 하는 것.
    - 코딩을 수행하면서 머릿속에 있는 정보를 기록하지
    - 코드를 읽는 사람의 입장에서 필요한 정보가 무엇인지 유추하기
    
### What NOT to Comment
  주석을 단다면 반드시 이유가 있어야 한다. 다음음 아무런 가치가 없다 
  ```cpp
    // The class definition for Account
    class Account {
        public:
            // Constructor
            Account();

            // Set the profit member to a new value
            void SetProfit(double profit);

            // Return the profit from this Account
            double GetProfit();
    };
  ```
KEY IDEA: Don’t comment on facts that can be derived quickly from the code itself.

  아래 코드의 주석에는 아무런 정보가 없음
  ```python    
  # remove everything after the second '*'
  name = '*'.join(line.split('*')[:2])
  ```

  - Don’t Comment Just for the Sake of Commenting
     
    모든  함수에 주석을 달아야 한다고 요구 받을때 아래와 같이 적는 경우가 있는데 이는 무가치 하다.(이미 함수명을 통해 유추가능한 것.)
    ```cpp
    // Find the Node in the given subtree, with the given name, using the given depth.
    Node* FindNodeInSubtree(Node* subtree, string name, int depth);
    ```
     
    아래와 같이 중요한 세부 사항에 대한 주석들 적는 것이 낫다.
    ```cpp
    // Find a Node with the given 'name' or return NULL.
    // If depth <= 0, only 'subtree' is inspected.
    // If depth == N, only 'subtree' and N levels below are inspected.
    Node* FindNodeInSubtree(Node* subtree, string name, int depth);
    ```
    
    pch comment: 위와 같은 주석은 로직 변경에 의한 꾸준한 주석 update 또한 요구된다. 
  
  - Don’t Comment Bad Names—Fix the Names Instead
    ```
    // Enforce limits on the Reply as stated in the Request,
    // such as the number of items returned, or total byte size, etc.
    void CleanReply(Request request, Reply reply);
    ```
    
    ```
    // Make sure 'reply' meets the count/byte/etc. limits from the 'request'
    void EnforceLimitsFromRequest(Request request, Reply reply);
    ```
    
    ```
    // Releases the handle for this key. This doesn't modify the actual registry.
    void DeleteRegistry(RegistryKey* key);
    ```
    
    ```
    void ReleaseRegistryHandle(RegistryKey* key);
    ```
  

### Recording Your Thoughts
  좋은 주석ㄷ은 자신의 생각을 기록하는 것만으로도 탄생할 수 있다. 
  - Include “Director Commentary”
    - 중요한 통찰을 기록한 주석을 코드에 포함시켜라. 
      다음 주석은 코드를 최적화하느라 시간을 허비하지 않게 도와준다. 
    ```
    // Surprisingly, a binary tree was 40% faster than a hash table for this data.
    // The cost of computing a hash was more than the left/right comparisons.
    ```
    다음과 같은 주석은 코드에 무언가 버그가 있다고 생각하고 수정, TC 작성에 허비하는 것을 방지한다(?)
    ```
    // This heuristic might miss a few words. That's OK; solving this 100% is hard.
    ```
    pch comment: 앞의 내용에 따르면 'TODO' 혹은 'XXX' 등을 달아야 할거 같은데 ? 아래 예제도 마찬가지
    
    ```
    // This class is getting messy. Maybe we should create a 'ResourceNode' subclass to
    // help organize things.
    ```
    
  - Comment the Flaws in Your Code
    코드는 지속적으로 진화하며, 그런 과정 중에 버그를 가지게 될 수 밖에 없다. 결함 혹은 미비한 점을 설명하는 주석을 부끄러워 하지 말자. 
    ```cpp
    // TODO: use a faster algorithm
    ```
    
    ```cpp
    // TODO(dustin): handle other image formats besides JPEG
    ```
    
    
  아래는 프로그래머 사이에 널리 사용되는 몇가지 표시이다.
    
    
  |Marker|Typical meaning|
  |------|---|
  |TODO:|Stuff I haven’t gotten around to yet|
  |FIXME:|Known-broken code here|
  |HACK:|Admittedly inelegant solution to a problem|
  |XXX:|Danger! major problem here|
    
  - Comment on Your Constants
    -  상수를 정의할땐 그 상수가 왜 그 값을 가지게 되었는지 '사연'을 적어주자
    
    ```
    NUM_THREADS = 8
    ```
    vs
    ```
    NUM_THREADS = 8 # as long as it's >= 2 * num_processors, that's good enough.
    ```
        
    - 아무런 의미를 가지지 않는 경우라도 그런 사실을 알려주라 
    
    ```
    // Impose a reasonable limit - no human can read that much anyway.
    const int MAX_RSS_SUBSCRIPTIONS = 1000;
    ``` 
    
    - 상수값이 tuned 되어 변경하지 않는 게 좋은 경우 
    ```
    image_quality = 0.72; // users thought 0.72 gave the best size/quality tradeoff
    ``` 
### Put Yourself in the Reader’s Shoes
  이 책은 '코드를 처음으로 읽는 외부인의 입장에 자기 자신을 놓는 기법' 에 관한 것. 
  - Anticipating Likely Questions
      - 다음과 같은 코드를 보자 
        ```cpp
        struct Recorder {
            vector<float> data;
                ...
                void Clear() {
                    vector<float>().swap(data); // Huh? Why not just data.clear()?
            }
        };
        ```
        - 왜 clear를 호출하지 않고 data와 빈 벡터를 교체하는가? 
          - 이것은 vector가 메모리 할당자에게 자신의 메모리를 실제로 반납하게 하는 유일한 방법.. (pch comment: 다시 한번 이야기 하지만 이 책은 2011년 책)
          - 이 코드는 아래와 같은 주석이 있어야 했다.
              ```cpp
              // Force vector to relinquish its memory (look up "STL swap trick")
              vector<float>().swap(data);
              ```     
        
  - Advertising Likely Pitfalls
    -  다른 사람이 코드를 사용하다가 만날지도 모르는 문제들을 미리 예측하자.
      - 
### Final Thoughts - Getting Over Writer’s Block
  - 좋은 주석을 달기 위해서 글 쓰는 걸 두려워 하지 마라. 일단 쓰고 고쳐라.
    ```
    // 이런 제길 이 리스트 안에 중복된 항목이 있으면 이건 복잡해 지잖아
    ```
    - '이런 제길' = '주의: 주의릴 기울여야 할 내용'
    - '이건' = '입력을 다루는 코드'
    - '복잡해지잖아' = '구현하기 어려워진다'
    
    ```
    // 주의: 이 코드는 리스트안에 있는중복된 항목을 다루지 않는다. 그렇게 하는 것이 어렵기 때문이다. 
    ```
  - 주석 쓰기의 단계
    1. 마음에 떠오르는 생각을 무조건 적기.
    2. 주석을 읽고 무엇이 개선되어야 하는지 (그런 부분이 있다면) 확인하다.
    3. 개선한다.  
    
### Summary
  - 설명하지 말아야 할 것
    - 코드 자체에서 빠르게 도출되는 것
    - 나쁘게 작성된 코드를 보정하려고 애쓰는 주석. 주석을 달지 말고 코드를 수정해라
  - 기록해야 하는 생각
    - 코드가 특정한 방식으로 작성된 이유
    - 코드에 담긴 결함, TODO: 혹은 XXX: 와 같은 표시를 사용.
    - 어떤 상수가 특정한 값을 가지게 된 '사연'
  - 자신을 코드를 '읽는' 입장에 놓고 보아라.
    - 코드를 읽는 사람이 코드를 보고 '뭐라고?' 라는 생각을 할지 예측해 보고 그 부분에 주석을 넣어라
    - 평범한 사람이 예상하지 못할 특이한 동작을 기록하라
    - 파일이나 클래스 수준 주석에서 '큰 그림'을 설명하고 각 조각이 어떻게 맞추어 지는지 설명
    - 코드에 블록별로 주석을 달아 나무만 보다가 숲을 못 보게 하는 실수를 저지르지 마라.