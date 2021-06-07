## Chapter 1. Code Should Be Eash to Understand

- key idea: Code should be easy to understand

  - 위 코드가 아래 보다 낫다
    ```cpp
    for (Node* node = list->head; node != NULL; node = node->next)
        Print(node->data);
    ```

    ```cpp
    Node* node = list->head;
    if (node == NULL) return;
    
    while (node->next != NULL) {
        Print(node->data);
        node = node->next;
    }
    
    if (node != NULL) Print(node->data);
    ```

- key idea: Code should be written to minimize the time it would take for someone else to understatnd it (이후 가독성의 기본정리로 통칭)
  - time-till-understanding: 평범한 동료 한 사람이 본인이 작성한 코드를 읽고 이해하는데 걸리는 시간. 최소해야하는 것은 이 값이다. 
  - 1인 프로젝트를 한다 하더라도, 6개월 뒤의 자신이 이 그 평범한 동료 한 사람이 될 수 있다. 

- 분량이 적다고 항상 좋은 것은 아니다
  - 위의 예를 보면 분량이 적은 코드가 항상 더 이해하기 좋다고 생각할 수 있지만 이는 아니다.
    ```cpp
        assert((!(bucket = FindBucket(key))) || !bucket->IsOccupied());
    ```
    
    ```cpp
        bucket = FindBucket(key);
        if (bucket != NULL) assert(!bucket->IsOccupied());
    ```

  - 부득이 하다면 주석을 사용하자
    ```cpp
        // Fast version of "hash = (65599 * hash) + c"
        hash = (hash << 6) + (hash << 16) - hash + c;
    ```



  


  
