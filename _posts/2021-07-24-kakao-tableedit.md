---
layout: post
title: "[프로그래머스] 표 편집 풀이 (2021 카카오 인턴십)"
date: 2021-07-24
categories: kakao
photos: /assets/post_images/kakao/tableedit.png
tags: [ps, kakao, programmers, algorithm, c++, string, linked_list, set, implementation]
description: "프로그래머스 - 표 편집 C++ 풀이 (2021 카카오 인턴십 코딩테스트)"
---

<br>

# 문제 설명

업무용 소프트웨어를 개발하는 니니즈웍스의 인턴인 앙몬드는 명령어 기반으로 표의 행을 선택, 삭제, 복구하는 프로그램을 작성하는 과제를 맡았습니다. 세부 요구 사항은 다음과 같습니다

![0](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/d8e89054-53ba-4222-a485-dc56893f45e4/table_1.png)

위 그림에서 파란색으로 칠해진 칸은 현재 **선택된 행**을 나타냅니다. 단, 한 번에 한 행만 선택할 수 있으며, 표의 범위(0행 ~ 마지막 행)를 벗어날 수 없습니다. 이때, 다음과 같은 명령어를 이용하여 표를 편집합니다.

- `"U X"`: 현재 선택된 행에서 X칸 위에 있는 행을 선택합니다.
- `"D X"`: 현재 선택된 행에서 X칸 아래에 있는 행을 선택합니다.
- `"C"` : 현재 선택된 행을 삭제한 후, 바로 아래 행을 선택합니다. 단, 삭제된 행이 가장 마지막 행인 경우 바로 윗 행을 선택합니다.
- `"Z"` : 가장 최근에 삭제된 행을 원래대로 복구합니다. **단, 현재 선택된 행은 바뀌지 않습니다.**

예를 들어 위 표에서 `"D 2"`를 수행할 경우 아래 그림의 왼쪽처럼 4행이 선택되며, `"C"`를 수행하면 선택된 행을 삭제하고, 바로 아래 행이었던 "네오"가 적힌 행을 선택합니다(4행이 삭제되면서 아래 있던 행들이 하나씩 밀려 올라오고, 수정된 표에서 다시 4행을 선택하는 것과 동일합니다).

처음 표의 행 개수를 나타내는 정수 n, 처음에 선택된 행의 위치를 나타내는 정수 k, 수행한 명령어들이 담긴 문자열 배열 cmd가 매개변수로 주어질 때, 모든 명령어를 수행한 후 표의 상태와 처음 주어진 표의 상태를 비교하여 삭제되지 않은 행은 O, 삭제된 행은 X로 표시하여 문자열 형태로 return 하도록 solution 함수를 완성해주세요.

<br>

# 제한사항

- 5 ≤ `n` ≤ 1,000,000
- 0 ≤ `k` < `n`
- 1 ≤ `cmd`의 원소 개수 ≤ 200,000
    - `cmd`의 각 원소는 `"U X"`, `"D X"`, `"C"`, `"Z"` 중 하나입니다.
    - X는 1 이상 300,000 이하인 자연수이며 0으로 시작하지 않습니다.
    - X가 나타내는 자연수에 ',' 는 주어지지 않습니다. 예를 들어 123,456의 경우 123456으로 주어집니다.
    - `cmd`에 등장하는 모든 X들의 값을 합친 결과가 1,000,000 이하인 경우만 입력으로 주어집니다.
    - 표의 모든 행을 제거하여, 행이 하나도 남지 않는 경우는 입력으로 주어지지 않습니다.
    - 본문에서 각 행이 제거되고 복구되는 과정을 보다 자연스럽게 보이기 위해 `"이름"` 열을 사용하였으나, `"이름"`열의 내용이 실제 문제를 푸는 과정에 필요하지는 않습니다. `"이름"`열에는 서로 다른 이름들이 중복없이 채워져 있다고 가정하고 문제를 해결해 주세요.
- 표의 범위를 벗어나는 이동은 입력으로 주어지지 않습니다.
- 원래대로 복구할 행이 없을 때(즉, 삭제된 행이 없을 때) "Z"가 명령어로 주어지는 경우는 없습니다.
- 정답은 표의 0행부터 n - 1행까지에 해당되는 O, X를 순서대로 이어붙인 문자열 형태로 return 해주세요.

<br>

# 풀이

`구현`이나 `문자열` 실력에 부족함이 없다면 **정확성 테스트**는 쉽게 통과할 수 있는 문제였다. 하지만 **효율성 테스트**에서 다양한 풀이 방법을 떠올리며 삽질을 할 수 있다. 필자도 `유니온 파인드`의 `경로 압축`을 통해 커서를 이동하거나 `세그먼트 트리`를 이용해 특정 구간에서 삭제된 행의 `누적 합`을 계산하는 등... 참 멀리 돌아갔었지만 이 문제는 `리스트`에서 `삭제`와 `복원`을 한다는 점에 착안해서 `연결 리스트`를 떠올려야 한다. 정확히 말하자면 양쪽으로 탐색이 모두 가능한 `양방향 연결 리스트`이다.

<br>

## 연결 리스트 풀이
  
`삭제`에서 `연결 리스트`를 떠올리는 사람은 많지만 `복원`에서 `연결 리스트`를 떠올리는 사람은 상대적으로 적을 수 있다. `복원`이 `연결 리스트`와 찰떡인 이유는 다음과 같다.

![1](/assets/post_images/kakao/tableedit_1.png)

> ***연결 리스트에서 삭제된 노드는 자신의 prev 노드와 next 노드를 기억하고 있다!***

`연결 리스트`의 노드를 삭제하는 과정에서 해당 노드의 정보를 직접 건들이지 않고 그 노드와 양쪽으로 연결된 노드들만의 정보를 건들이기 때문에 삭제된 노드는 자신이 누구와 연결되어 있었는지에 대한 정보를 온전히 보존하고 있다. 따라서 이 정보를 이용해 `prev` 노드와 `next` 노드에 접근하여 다시 자신을 가리키게 할 수 있다.

<br>

## set 자료구조 풀이

`연결 리스트`에 비해 코드 길이가 **2 ~ 3배** 짧은 풀이가 존재한다. 바로 집합 자료구조 `set`을 이용한 풀이다. 이는 `C++`의 `STL`에 깊은 이해가 있는 사람만이 떠올릴 수 있는 풀이법이다. 자신이 다음과 같은 내용을 알고있는지 점검해보자.

- `set` 자료구조는 내부적으로 `이진 트리` 구조를 갖추고 있다.
    - 삽입 / 삭제의 시간 복잡도가 **O(logN)** 이다.
- `iterator`를 통해 `set`의 원소에 순차적으로 접근할 수 있다.
    - `next()`, `prev()` 를 통해 현재 가리키는 `iterator`의 양 옆을 탐색할 수 있다.
- `set` 자료구조의 원소는 삽입과 동시에 **정렬된다.**

위 내용을 이해했다면 `set` 자료구조를 선언만 하면 자동으로 본 문제에서 요구하는 **표** 시스템이 구축됨을 알 수 있다! 기능에 따른 구현은 다음과 같이 할 수 있다.

- 커서 이동: `next(set<int>::iterator)`, `prev(set<int>::iterator)`
- 행 삭제: `set<int>::erase(set<int>::iterator)`
- 행 복원: `set<int>::insert(int)`

심지어 `erase()`의 같은 경우, 원소 삭제 후 해당 원소의 다음 원소를 가리키는 `iterator`를 반환하므로 별 다른 구현없이 문제의 요구사항을 만족시킴을 알 수 있다. 삭제한 원소가 마지막 원소여서 `iterator`가 `set<int>::end()` 를 가리키게 될 경우에만 `prev()` 를 통해 이전 원소를 가리키게끔 예외 처리를 해주면 된다.

풀이만 봐도 감이 오겠지만 이렇게 구현하면 코드가 상당히 짧다. 다만 `연결 리스트` 풀이에 비해 실행 시간은 `5 ~ 6배` 정도 오래걸리는 것으로 나온다. 하지만 문제에서 요구하는 **효율성 테스트**는 충분히 통과할 수 있을 정도의 속도이고, 구현이 간단하여 `연결 리스트` 풀이에 비해 실수할 요소가 확연히 적어지므로 실전에서 본 풀이를 떠올린다면 **시간적으로 큰 이득**을 얻을 수 있을 것이다.

<br>

> 문제를 푼지 반년만에 다시 풀이해 본 결과, set 자료구조를 이용한 풀이는 더 이상 프로그래머스에서 **효율성 테스트**를 통과하지 못하는 것으로 확인했습니다. 아무래도 본 풀이를 카카오의 의도에 맞지 않는 풀이로 취급하여 테스트 케이스에 데이터 **삽입 및 삭제**를 대량으로 수행하는 케이스를 추가한 것으로 보입니다. set 자료구조를 이용한 풀이는 참고만 해주시고, 프로그래머스 상에서 정답 처리를 받기 위해서는 **연결 리스트 풀이**를 사용하셔야 함을 알려드립니다.

<br>

# 전체 코드

- 연결 리스트 풀이

```c++
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int n;
    Node* prev;
    Node* next;
    Node(int n, Node* prev, Node* next) : n(n), prev(prev), next(next) {}
};

string solution(int n, int k, vector<string> cmd) {
    string answer(n, 'X');
    
    // 테이블 생성
    Node* cursor = new Node(0, NULL, NULL);
    for(int i=1;i<n;i++) {
        cursor->next = new Node(i, cursor, NULL);
        cursor = cursor->next;
    }
    
    // k로 커서 이동
    for(int i=0;i<n-k-1;i++) cursor = cursor->prev;
    
    // cmd 수행
    stack<Node*> del;
    for(string str : cmd) {
        if(str[0] == 'U' || str[0] == 'D') {
            int x = stoi(str.substr(2));
            if(str[0] == 'U') while(x--) cursor = cursor->prev;
            else while(x--) cursor = cursor->next;
        }
        else if(str[0] == 'C') {
            del.push(cursor);
            if(cursor->prev != NULL) cursor->prev->next = cursor->next;
            if(cursor->next != NULL) cursor->next->prev = cursor->prev;
            if(cursor->next == NULL) cursor = cursor->prev;
            else cursor = cursor->next;
        }
        else {
            Node* r = del.top(); del.pop();
            if(r->prev != NULL) r->prev->next = r;
            if(r->next != NULL) r->next->prev = r;
        }
    }
    
    // 삭제 여부 검사
    // (다소 비효율적인 방법이지만 코드의 간결성을 위해)
    answer[cursor->n] = 'O';
    while(cursor->prev != NULL) {
        answer[cursor->prev->n] = 'O';
        cursor = cursor->prev;
    }
    while(cursor->next != NULL) {
        answer[cursor->next->n] = 'O';
        cursor = cursor->next;
    }
    
    return answer;
}
```

<br>

- set 자료구조 풀이

```c++
#include <set>
#include <iterator>
#include <string>
#include <vector>
#include <stack>
using namespace std;

string solution(int n, int k, vector<string> cmd) {
    string answer(n,'X');

    set<int> table;
    stack<int> hist;
    for(int i=0;i<n;i++) table.insert(i);

    auto cur = table.find(k);

    for(string s:cmd){
        if(s[0]=='U' || s[0]=='D'){
            int x = stoi(s.substr(2));
            if(s[0]=='U') while(x--) cur = prev(cur);
            else while(x--) cur = next(cur);
        }
        else if(s[0]=='C'){
            hist.push(*cur);
            cur = table.erase(cur);
            if(cur==table.end()) cur = prev(cur);
        }
        else if(s[0]=='Z'){
            table.insert(hist.top());
            hist.pop();
        }
        else return ""; // oops
    }

    for(int i:table) answer[i]='O';

    return answer;
}
```