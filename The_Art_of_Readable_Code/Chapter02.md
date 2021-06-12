# Part I Surface-Level Improvements 
- 표면적 수준이란, 좋은 이름을 짓고 좋은 을 다고 코드를 보기 좋게 정렬하는 것 등을 의미. 
- 리팩토링이나 동작하는 방식의 변경없이 조금씩 적은 시간으로 변화를 만들어 나갈수 있다.


## Chapter 2. Packing Information into Names
- key idea: Pack information into your names.
  
  
### Choose Specific Words
- 너무 보편적인 단어보단 구체적인 단어를 선택한다
  
  - 보편적인 단어 선택의 예
    ```python
    def GetPage(url):
      ...
    ```
    '어디' 에서 페이지를 가져오는가? 로컬캐시? DB???  만약 인터넷에서 가져오는 것이라면 FetchPage, DownloadPage 가 좀 더 구체적인 명칭이 될 것이다. 

  - 불분명한 의미의 예
    ```cpp
    class BinaryTree {
      int Size();
      ...
    };
    ```
    '어떤' 것의 size를 말하는가? 트리 높이, 노드 갯수, 트리의 메모리 사용량? 
     ```cpp
    class Thread {
      void Stop();
      ...
    };
    ```
    Stop 보단, 수행하는 바에 따라, Kill 또는 Pause 가 좋은 선택이다.
      
  - 더 적합할 수 있는 단어의 예
    |Word|Alternatives|
    |------|---|
    |send|deliver, dispatch, announce, distribute, route|
    |find|search, extract, locate, recover|
    |start|launch, create, begin, open|
    |make|create, set up, build, generate, compose, add, new|

### Avoid Generic Names Like tmp and retval
- tmp, retval, foo 같은 이름이 아닌, 개체의 값이 목적을 정확히 설명하는 이름을 선택해야한다.
  - retval
    ```javascript
    <!-- TMI euclidean_norm = L^2 norm -->
    var euclidean_norm = function (v) {
      var retval = 0.0;
      for (var i = 0; i < v.length; i += 1)
        retval += v[i] * v[i];
      return Math.sqrt(retval);
    };
    ```
    
    실수로 아래와 같이 작성되었다고 하자
    ```javascript
    retval += v[i];    
    ```
    변수 명이 sum_squares 였다면 버그가 명확히 드러났을 것이다.

  - tmp
    - 적절한 예 
      ```cpp
      if (right < left) {
        tmp = right;
        right = left;
        left = tmp;
      }
      ```
  - 게으름의 산물
      ```java
      String tmp = user.name();
      tmp += " " + user.phone_number();
      tmp += " " + user.email();
        ...
      template.set("user_info", tmp);
      ```
      변수 lifecyle 이 위 처럼 짤지만, 임시적 저장소로 국한된 변수는 아니라 user 정보가 조합되어가는 변수이다. 따라서 userInfo 가 더 적당함.
  
  - 일부로써 tmp
      ```java
      tmp_file = tempfile.NamedTemporaryFile()
      ...
      SaveData(tmp_file, ...)
      ```
      만약 위와 같지 않고 아래와 같다면 tmp 가 파일인지, 파일이름인지 혹은 기록될 데이터 인지 구분하기 힘들다.

      ```java
      tmp = tempfile.NamedTemporaryFile()
      ...
      SaveData(tmp, ...)
      ```
  - loop iteration
    ```cpp
    for (int i = 0; i < clubs.size(); i++)
      for (int j = 0; j < clubs[i].members.size(); j++)
        for (int k = 0; k < users.size(); k++)
          if (clubs[i].members[k] == users[j])
            cout << "user[" << j << "] is in club[" << i << "]" << endl;
    ```
    버그를 찾아봅시다.

    ```
    if (clubs[i].members[k] == users[j])
    ```

    (i, j, k) 보다는 club_i, members_i, users_i 혹은 ci, mi, ui  가 더 좋은 선택이다.
    

### Prefer Concrete Names over Abstract Names
  - Example: DISALLOW_EVIL_CONSTRUCTORS
    - cpp 에서 복사생성자, 할당 연산자 생략하면 자동으로 기본값 제공. -> 메모리 누수 등 다른 문제 야기 가능
    - google code base
      ```cpp
      class ClassName {
        private:
          DISALLOW_EVIL_CONSTRUCTORS(ClassName);
        public:
        ...
      };
        
        ...
      #define DISALLOW_EVIL_CONSTRUCTORS(ClassName) \
        ClassName(const ClassName&); \
        void operator=(const ClassName&);
      ```
    - EVIL에 대한 모호성으로 아래가 더 적합함
      ```cpp
      #define DISALLOW_COPY_AND_ASSIGN(ClassName) ...
      ```

  - Example: --run_locally
    - 필자가 참여했던 프로젝트에서 사용했던 '디버깅 정보 출력' flag. 주로 로컬에서 테스트 수행 시 사용되고 원격 서버에서 동작할때는 사용되지 않는 환경하에서 지어진 이름    
    - 문제점 1. 새로 합류한 사람은 무엇을 위한 플래그인지 알수 없다
    - 문제점 2. 원격 서버에서도 디버깅 정보 출력이 필요한 상황 발생
    - 문제점 3. 로컬 에서 성능 테스트 시 이 기능을 사용하지 않아야 함
    결론. 위 상황에선 --extra_logging 이 더 적합.
    
### Attaching Extra Information to a Name

  ```cpp
  string id; // Example: "af84ef845cd8"
  ```
  id 내용을 기억해야하거나, id 에 다양한 포맷이 있다면 hex 를 붙이는 것이 좋은 선택

  - 단위를 가지는 값
    ```javascript
    var start = (new Date()).getTime(); // top of the page
    ...
    var elapsed = (new Date()).getTime() - start; // bottom of the page
    document.writeln("Load time was: " + elapsed + " seconds");
    ```

    ```javascript
    var start_ms = (new Date()).getTime(); // top of the page
    ...
    var elapsed_ms = (new Date()).getTime() - start_ms; // bottom of the page
    document.writeln("Load time was: " + elapsed_ms / 1000 + " seconds");
    ```

    |함수|인수단위를 포함하게 renaming|
    |------|---|
    |Start(int delay )|delay → delay_secs|
    |CreateCache(int size )| size → size_mb |
    |ThrottleDownload(float limit)| limit → max_kbps|
    |Rotate(float angle)| angle → degrees_cw |


  - 다른 중요한 속성 포함하기
    |상황|변수명|더 나은 변수명|
    |------|---|---|
    |A password is in “plaintext” and should be encrypted before further processing|password|paintext_password|
    |A user-provided comment that needs escaping before being displayed|comment | unescaped_comment|
    |Bytes of html have been converted to UTF-8|html|html_utf8|
    |Incoming data has been “url encoded”| data|data_urlenc|

### How Long Should a Name Be?
  ```java
  newNavigationControllerWrappingViewControllerForDataSourceOfClass
  ```
  이름이 길면 기억하기 어렵고, 다음줄로 코드가 넘어갈수도 있다. 하지만 너무 짧으면 의미를 담지 못한다. 
  d, days, 혹은 days_since_last_update 중 우리는 어떤 변수명을 선택해야 하는가?
  
  - 좁은 범위에서는 짧은 이름도 좋은 선택
    ```cpp
    if (debug) {
      map<string,int> m;
      LookUpNamesNumbers(&m);
      Print(m);
    }
    ``` 
    m은 아무런 정보를 담고 있지 않지만 해당 블록이 작아서 쉽게 확인이 가능하다
    
  - 긴 이름이 더 이상 문제되지 않는다
    IDE 발달로 변수 자동 완성기능이 잘 되어 있다. 
  
  - 약어와 축약형
    - BackEndManger vs BEManager: 특정 프로젝트에 국한된 의미를 가진 약어 사용은 좋지 않다. 새로 합류한 인원에게 바로 이해가능한 코드가 되지 못 함. 시간이 흐르면 본인도 그 의미를 다시 생각해내야 한다.
    - 규칙: 팀에 새로 합류한 사람이 이름이 의미하는 바를 바로 이해하면 좋은 이름.      
  - 불필요한 단어 제거
    - 경우에 따라서는 생략을 해도 실질적 정보가 없어지지 않는다
    - ConvertToString() vs ToString()
    - DoServerLoop() vs ServerLoop() 
    - pch comment. 함수명은 동사로 시작.. 어느 범위까지...???
  
### Use Name Formatting to Convey Meaning
  - 밑줄, 대시, 그리고 대문자를 이용하면 이름에 더 많은 정보를 담을 수 있다.

    구글 오픈소스 프로젝트 포매팅 컨벤션을 따른 소스
    ```cpp
    static const int kMaxOpenFiles = 100; // CONSTANT_NAME 같이 쓰는 경우 대비,  매크로 함수와 구분이 쉬움
    class LogReader {
      public:
        void OpenFile(string local_file);
      private:
        int offset_; //_가 멤버변수 구분을 용이하게함. 
        DISALLOW_COPY_AND_ASSIGN(LogReader);
    };
    ```

    이런 관습은 아래와 같은 코드를 만날대, 클래스의 내부 상태를 변경하지 않았음을 쉽게 알 수 있다.
    ```cpp
    stats.clear();
    ```

    - pch comment. 코드 리뷰 시 naming 규약을 잘 지켰는지 확인을 잘 해야할 듯. 아니면 파국! 
    
  


  - 다른 포맷팅 관습
    - 프로젝트나 언어에 따라서 이름에 더 많은 정보를 담을 수 있는 포매팅 관습이 있을수 있다
    Douglas Crockford 의 제안: 생성자는 대문자, 나머지는 소문자
    ```javascript
    var x = new DatePicker(); // DatePicker() is a "constructor" function
    var y = pageHeight(); // pageHeight() is an ordinary function
    ``` 

    jQuery 의 관습
    ```javascript
    var $all_images = $("img"); // $all_images is a jQuery object
    var height = 250; // height is not
    ``` 

    ex: 밑줄로 id 안의 단어 구분, dash 로 클래스 안의 단어 구분
    ```html
    <div id="middle_column" class="main-content"> ...
    ```

### Summary
  - 상황에 따른 특정한 단어 사용
  - 보편적 이름 사용 피하기
  - 구체적인 이름 사용
  - 변수명에 중요한 세부 정보 표시
  - 사용 범위가 넓으면 긴 이름을 사용
  - 대문자나 밑줄등을 의미있는 방식으로 사용

  
