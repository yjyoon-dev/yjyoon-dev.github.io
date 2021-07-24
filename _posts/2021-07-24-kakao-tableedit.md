---
layout: post
title: "[프로그래머스] 표 편집 풀이 (2021 카카오 인턴십)"
date: 2021-07-24
categories: kakao
photos: /assets/post_images/kakao/tableedit.png
tags: [ps, boj, algorithm, c++, string, linked_list, set, implementation]
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

`연결 리스트`의 노드를 삭제하는 과정에서 해당 노드의 정보를 직접 건들이지 않고 그 노드와 양쪽으로 연결된 노드들만의 정보를 건들이기 때문에 삭제된 노드는 자신이 누구와 연결되어 있었는지에 대한 정보를 온전히 보존하고 있다. 따라서 이 정보를 이용해 `prev` 노드와 `next` 노드에 접근에 다시 자신을 가리키게 할 수 있다.

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

# 전체 코드

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

<br>

- 연결 리스트 풀이

```c++
#include <string>
#include <vector>
#include <stack>
using namespace std;

int cur;
stack<int> st;

// 연결 리스트 node
struct Node {
    int num;
    Node* prev;
    Node* next;
    Node(int num):num(num),prev(NULL),next(NULL){};
};

vector<Node*> v;

void solve(vector<string>& cmd){
    for(string s:cmd){
        if(s[0]=='D' || s[0]=='U'){
            int x = stoi(s.substr(2));
            if(s[0]=='D') while(x--) cur = v[cur]->next->num;
            else while(x--) cur = v[cur]->prev->num;
        }
        else if(s[0]=='C'){
            st.push(cur);
            if(v[cur]->prev != NULL)
                v[cur]->prev->next = v[cur]->next;
            if(v[cur]->next != NULL){
                v[cur]->next->prev = v[cur]->prev;
                cur = v[cur]->next->num;
            }
            else cur = v[cur]->prev->num;
        }
        else if(s[0]=='Z'){
            int r = st.top(); st.pop();
            if(v[r]->prev != NULL)
                v[r]->prev->next = v[r];
            if(v[r]->next != NULL)
            v[r]->next->prev = v[r];
        }
        else return; //oops
    }
}

string solution(int n, int k, vector<string> cmd) {
    string answer(n,'X');
    
    // 연결리스트 생성 및 연결
    v = vector<Node*>(n);
    
    for(int i=0;i<n;i++)
        v[i] = new Node(i);
    
    v[0]->next = v[1];
    v[n-1]->prev = v[n-2];
    
    for(int i=1;i<n-1;i++){
        v[i]->next = v[i+1];
        v[i]->prev = v[i-1];
    }
    
    // cmd 수행
    cur = k;
    solve(cmd);
    
    // 삭제 여부 체크
    int leftCheck, rightCheck;
    leftCheck = rightCheck = cur;
    
    answer[cur] = 'O';
    
    // 현재 커서 기준 왼쪽 체크
    while(v[leftCheck]->prev != NULL){
        leftCheck = v[leftCheck]->prev->num;
        answer[leftCheck] = 'O';
    }
    
    // 현재 커서 기준 오른쪽 체크
    while(v[rightCheck]->next != NULL){
        rightCheck = v[rightCheck]->next->num;
        answer[rightCheck] = 'O';
    }

    return answer;
}
```