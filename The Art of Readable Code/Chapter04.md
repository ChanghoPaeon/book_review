## Chapter 4. Aesthetics
 

### Why Do Aesthetics Matter?
  - 아래 두 코드를 비교해 보자.
  - ```cpp
    class StatsKeeper {
    public:
    // A class for keeping track of a series of doubles
        void Add(double d); // and methods for quick statistics about them
      private: int count; /* how many so far
    */ public:
            double Average();
    private: double minimum;
    list<double>
      past_items
          ;  double maximum;
    };
    ```
  
  - ```cpp
    // A class for keeping track of a series of doubles
    // and methods for quick statistics about them.
    class StatsKeeper {
        public:
            void Add(double d);
            double Average();
        private:
            list<double> past_items;
            int count; // how many so far
            double minimum;
            double maximum;
    };
    ```  
  - 미학적으로 보기 좋은 코드가 사용하기 더 편리한 것은 명백해 보인다.
  
### Rearrange Line Breaks to Be Consistent and Compact
  - Assumption: 한 줄당 80글자. 
  
      ```cpp 
        public class PerformanceTester {
            public static final TcpConnectionSimulator wifi = new TcpConnectionSimulator(
                500, /* Kbps */
                80, /* millisecs latency */
                200, /* jitter */
                1 /* packet loss % */);
        
        public static final TcpConnectionSimulator t3_fiber =    
            new TcpConnectionSimulator(
                45000, /* Kbps */
                10, /* millisecs latency */
                0, /* jitter */
                0 /* packet loss % */);
        
        public static final TcpConnectionSimulator cell = new TcpConnectionSimulator(    
            100, /* Kbps */
            400, /* millisecs latency */
            250, /* jitter */
            5 /* packet loss % */);
        }
      ```
    
      t3_fiber 에서 잠시 눈이 멈춘다. 이유는 비슷칸 코드지만, 비슷하게 보이지 않아서.
    
      ```cpp 
        public class PerformanceTester {
            public static final TcpConnectionSimulator wifi =
                new TcpConnectionSimulator(
                    500, /* Kbps */
                    80, /* millisecs latency */
                    200, /* jitter */
                    1 /* packet loss % */);
            public static final TcpConnectionSimulator t3_fiber =
                new TcpConnectionSimulator(
                    45000, /* Kbps */
                    10, /* millisecs latency */
                    0, /* jitter */
                    0 /* packet loss % */);
            public static final TcpConnectionSimulator cell =
                new TcpConnectionSimulator(
                    100, /* Kbps */
                    400, /* millisecs latency */
                    250, /* jitter */
                    5 /* packet loss % */);
        }
      ```
      추가적인 줄바꿈으로 일관성을 유지할 수 있다. 이는 훑어보기에 (scan to through) 쉽다. 
      pch comment: 보고서 작성의 원리와도 일맥 상통하는 느낌.!
      ```cpp 
        public class PerformanceTester {
            // TcpConnectionSimulator(throughput, latency, jitter, packet_loss)
            // [Kbps] [ms] [ms] [percent]
    
            public static final TcpConnectionSimulator wifi =
                new TcpConnectionSimulator(500,   80,     200,    1);
    
            public static final TcpConnectionSimulator t3_fiber =
                new TcpConnectionSimulator(45000, 10,     0,      0);
    
            public static final TcpConnectionSimulator cell =
                new TcpConnectionSimulator(100,   400,    250,    5);
        }
      ```
      한결 간결하다.
      pch comment: 파라미터 라인 맞추는 노력... ?  -> formatter 적용 가능한가?
      
    
    
### Use Methods to Clean Up Irregularity
  - ```cpp
    // Turn a partial_name like "Doug Adams" into "Mr. Douglas Adams".
    // If not possible, 'error' is filled with an explanation.
    string ExpandFullName(DatabaseConnection dc, string partial_name, string* error);
    ```
    
    이 함수가 다음과 같은 테스트를 거친다고 하자.
    
    ```cpp
    DatabaseConnection database_connection;
    string error;
    assert(ExpandFullName(database_connection, "Doug Adams", &error)
        == "Mr. Douglas Adams");
    assert(error == "");
    assert(ExpandFullName(database_connection, " Jake Brown ", &error)
        == "Mr. Jacob Brown III");
    assert(error == "");
    assert(ExpandFullName(database_connection, "No Such Guy", &error) == "");
    assert(error == "no match found");
    assert(ExpandFullName(database_connection, "John", &error) == "");
    assert(error == "more than one result");
    ```
    줄이 길어서 다음 줄까지 이어진다.
    아래와 같이 method를 도입하자. 
    
    ```cpp
    CheckFullName("Doug Adams", "Mr. Douglas Adams", "");
    CheckFullName(" Jake Brown ", "Mr. Jake Brown III", "");
    CheckFullName("No Such Guy", "", "no match found");
    CheckFullName("John", "", "more than one result");
    ```    
        
    ```cpp
    void CheckFullName(string partial_name,
                       string expected_full_name,
                       string expected_error) {
        // database_connection is now a class member
        string error;
        string full_name = ExpandFullName(database_connection, partial_name, &error);
        assert(error == expected_error);
        assert(full_name == expected_full_name);
    }
    ```
    미학적 개선에 더불어
      - It eliminates a lot of the duplicated code from before, making the code more compact(?)
      - 이름이나 에러 문자열 같은 테스트의 중요 부분이 한눈에 보인다. 
      - 새로운 테스트 추가가 쉬워 졌다. (2줄 -> 1줄의 의미?)

### Use Column Alignment When Helpful

  - 앞절의 코드 에서 다음과 같이하면 코드를 더 읽기 쉽게 할 수도 있다. (2, 3번째 파라미터 구별이 쉬움.) 
    ```cpp
    CheckFullName():
    
        CheckFullName("Doug Adams"  , "Mr. Douglas Adams" , "");
        CheckFullName(" Jake Brown ", "Mr. Jake Brown III", "");
        CheckFullName("No Such Guy" , ""                  , "no match found");
        CheckFullName("John"        , ""                  , "more than one result");
    ```
    pch comment: ?! HMM...
        
    wget 코드베이스에는 다음과 같이 나열되어 있다. 
    ```cpp
        # Extract POST parameters to local variables
        details = request.POST.get('details')
        location = request.POST.get('location')
        phone = equest.POST.get('phone')
        email = request.POST.get('email')
        url = request.POST.get('url')
    ```   
  
### Should You Use Column Alignment?
  - 열 정렬은 '비슷한 코드는 비슷하게 보여야 한다' 의 원리에 해당하는 좋은 예
  - 열정렬을 위한 노력이 필요하고, 이런 수정은 광범위한 diff 를 유발. 
  - 저자의 경험상 추천. 정 불편하면 그때 그만 둬라. 
      
### Pick a Meaningful Order, and Use It Consistently
  - 코드의 순서가 영향을 미치지 않는 경우가 많지만, 의미 있는 순서대로 나열해라.
    아래는 의미 없는 나열의 예.
      ```javascript
        details = request.POST.get('details')
        location = request.POST.get('location')
        phone = request.POST.get('phone')
        email = request.POST.get('email')
        url = request.POST.get('url')
      ```    
    - 변수의 순서를 HTML 폼의 순서대로 나열
    - '가장 중요한 것' 에서 '가장 덜 중요한 것' 순서대로 나열 (pch: 조기 리턴에도 도움이 될듯)
    - 알파벳 순서대로 나열
  - 어떤 순서를 사용하든 코드 전반에서 일관성을 유지하라.
    ```javascript
        if details: rec.details = details
        if phone: rec.phone = phone # Hey, where did 'location' go?
        if email: rec.email = email
        if url: rec.url = url
        if location: rec.location = location # Why is 'location' down here now?
    ```

### Organize Declarations into Blocks
  - 우리는 그룹과 계층에 익숙하다. 따라서 아래보단
    ```cpp
    class FrontendServer {
        public:
            FrontendServer();
            void ViewProfile(HttpRequest* request);
            void OpenDatabase(string location, string user);
            void SaveProfile(HttpRequest* request);
            string ExtractQueryParam(HttpRequest* request, string param);
            void ReplyOK(HttpRequest* request, string html);
            void FindFriends(HttpRequest* request);
            void ReplyNotFound(HttpRequest* request, string error);
            void CloseDatabase(string location);
            ~FrontendServer();
    };
    ```
    
    다음이 쉽다.
    
    ```cpp
    class FrontendServer {
        public:
            FrontendServer();
            ~FrontendServer();
    
            // Handlers
            void ViewProfile(HttpRequest* request);
            void SaveProfile(HttpRequest* request);
            void FindFriends(HttpRequest* request);
    
            // Request/Reply Utilities
            string ExtractQueryParam(HttpRequest* request, string param);
            void ReplyOK(HttpRequest* request, string html);
            void ReplyNotFound(HttpRequest* request, string error);
            // Database Helpers
    
            void OpenDatabase(string location, string user);
            void CloseDatabase(string location);
    };
    ```
    

### Break Code into “Paragraphs”
  - 우리가 글을 쓸때 여러 문단으로 나누는 이유는 여러가지가 있다.
    - 비슷한 생각을 하나로 묶어서, 성격이 다른 생각과 구분
    - 문단은 시각적 디딤돌 역할. 문단이 없으면 하나의 페이지 안에서 읽던 부분을 놓치기 슆다
    - 하나의 문단에서 다른 문단으로의 navigation을 쉽게 한다.
    
    따라서 우리는 
    
    ```javascript
    # Import the user's email contacts, and match them to users in our system.
    # Then display a list of those users that he/she isn't already friends with.
    def suggest_new_friends(user, email_password):
        friends = user.friends()
        friend_emails = set(f.email for f in friends)
        contacts = import_contacts(user.email, email_password)
        contact_emails = set(c.email for c in contacts)
        non_friend_emails = contact_emails - friend_emails
        suggested_friends = User.objects.select(email__in=non_friend_emails)
        display['user'] = user
        display['friends'] = friends
        display['suggested_friends'] = suggested_friends
        return render("suggested_friends.html", display)
    ```
    
    ```javascript
    def suggest_new_friends(user, email_password):
        # Get the user's friends' email addresses.
        friends = user.friends()
        friend_emails = set(f.email for f in friends)
        
        # Import all email addresses from this user's email account.
        contacts = import_contacts(user.email, email_password)
        contact_emails = set(c.email for c in contacts)
        
        # Find matching users that they aren't already friends with.
        non_friend_emails = contact_emails - friend_emails
        suggested_friends = User.objects.select(email__in=non_friend_emails)
        
        # Display these lists on the page.
        display['user'] = user
        display['friends'] = friends
        display['suggested_friends'] = suggested_friends
        
        return render("suggested_friends.html", display)
    ```

### Personal Style versus Consistency
  - '미학'적인 문제는 개인의 스타일 문제로 귀결되는 사안이 있다. 특히 block opening.
    ```cpp
    class Logger {
        ...
    };
    ```
    
    ```cpp
    class Logger 
    {
        ...
    };
    ```
    무엇을 선택하든 가독성에 영향은 없다. 섞어쓰지만 마라.
  - key idea: Consistent style is more important than the “right” style.
  
### Summary
  - 여러 블록에 담긴 코드가 비슷한 일을 수행하면, 실루엣이 동일하게 보이도록 하라
  - 코드 열 맞춤을 하면 훑어 보기 수월하다 (easy to skim through)
  - 순서르 바꾸지 마라. 한 곳에서, A, B, C 로 언급되는데 다른 곳에서 B, C, A 와 같은 식으로 언급하지 마라.
  - 빈 줄을 이용하여 커다란 블록을 논리적인 '문단'으로 나누어라