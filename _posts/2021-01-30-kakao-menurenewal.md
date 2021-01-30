---
layout: post
title: "[프로그래머스] 메뉴 리뉴얼 문제 풀이 (2021 카카오 코딩테스트)"
date: 2021-01-30
categories: kakao
photos: /assets/post_images/kakao/menurenewal.png
tags: [ps,algorithm,c++,kakao,programmers,bruteforce,string,data_structcure,implementation]
description: "2021 카카오 블라인드 채용 코딩테스트 - 메뉴 리뉴얼 C++ 풀이 (프로그래머스)"
---

<br>

# 문제 설명

레스토랑을 운영하던 스카피는 코로나19로 인한 불경기를 극복하고자 메뉴를 새로 구성하려고 고민하고 있습니다.
기존에는 단품으로만 제공하던 메뉴를 조합해서 코스요리 형태로 재구성해서 새로운 메뉴를 제공하기로 결정했습니다. 어떤 단품메뉴들을 조합해서 코스요리 메뉴로 구성하면 좋을 지 고민하던 스카피는 이전에 각 손님들이 주문할 때 가장 많이 함께 주문한 단품메뉴들을 코스요리 메뉴로 구성하기로 했습니다.
단, 코스요리 메뉴는 최소 2가지 이상의 단품메뉴로 구성하려고 합니다. 또한, 최소 2명 이상의 손님으로부터 주문된 단품메뉴 조합에 대해서만 코스요리 메뉴 후보에 포함하기로 했습니다.

예를 들어, 손님 6명이 주문한 단품메뉴들의 조합이 다음과 같다면,
(각 손님은 단품메뉴를 2개 이상 주문해야 하며, 각 단품메뉴는 A ~ Z의 알파벳 대문자로 표기합니다.)

손님 번호|주문한 단품메뉴 조합
---|---
1번 손님|A, B, C, F, G
2번 손님|A, C
3번 손님|C, D, E
4번 손님|A, C, D, E
5번 손님|B, C, F, G
6번 손님|A, C, D, E, H

가장 많이 함께 주문된 단품메뉴 조합에 따라 스카피가 만들게 될 코스요리 메뉴 구성 후보는 다음과 같습니다.

코스 종류|메뉴 구성|설명
---|---|---
요리 2개 코스|A, C|1번, 2번, 4번, 6번 손님으로부터 총 4번 주문됐습니다.
요리 3개 코스|C, D, E|3번, 4번, 6번 손님으로부터 총 3번 주문됐습니다.
요리 4개 코스|B, C, F, G|1번, 5번 손님으로부터 총 2번 주문됐습니다.
요리 4개 코스|A, C, D, E|4번, 6번 손님으로부터 총 2번 주문됐습니다.

<br>

# 문제

각 손님들이 주문한 단품메뉴들이 문자열 형식으로 담긴 배열 orders, 스카피가 추가하고 싶어하는 코스요리를 구성하는 단품메뉴들의 갯수가 담긴 배열 course가 매개변수로 주어질 때,
스카피가 새로 추가하게 될 코스요리의 메뉴 구성을 문자열 형태로 배열에 담아 return 하도록 solution 함수를 완성해 주세요.

<br>

# 제한사항

- orders 배열의 크기는 2 이상 20 이하입니다.
- orders 배열의 각 원소는 크기가 2 이상 10 이하인 문자열입니다.
  - 각 문자열은 알파벳 대문자로만 이루어져 있습니다.
  - 각 문자열에는 같은 알파벳이 중복해서 들어있지 않습니다.
- course 배열의 크기는 1 이상 10 이하입니다.
  - course 배열의 각 원소는 2 이상 10 이하인 자연수가 오름차순으로 정렬되어 있습니다.
  - course 배열에는 같은 값이 중복해서 들어있지 않습니다.
- 정답은 각 코스요리 메뉴의 구성을 문자열 형식으로 배열에 담아 사전 순으로 오름차순 정렬해서 return 해주세요.
  - 배열의 각 원소에 저장된 문자열 또한 알파벳 오름차순으로 정렬되어야 합니다.
  - 만약 가장 많이 함께 주문된 메뉴 구성이 여러 개라면, 모두 배열에 담아 return 하면 됩니다.
  - orders와 course 매개변수는 return 하는 배열의 길이가 1 이상이 되도록 주어집니다.
  
 <br>
 
 # 입출력 예
 
orders|course|result
---|---|---
["ABCFG", "AC", "CDE", "ACDE", "BCFG", "ACDEH"]|[2,3,4]|["AC", "ACDE", "BCFG", "CDE"]
["ABCDE", "AB", "CD", "ADE", "XYZ", "XYZ", "ACD"]|[2,3,5]|["ACD", "AD", "ADE", "CD", "XYZ"]
["XYZ", "XWY", "WXA"]|[2,3,4]|["WX", "XY"]

<br>

# 풀이

시간을 재고 2021 카카오 코딩테스트 전체 문제를 풀었을 때 총 7문제 중 2문제를 못 풀었는데 그 중 하나가 바로 이 문제였다. (나머지 하나는 마지막 문제였다.)
처음 이 문제를 보고 당연히 `그래프` 문제인 줄 알고 `그래프 탐색` 으로 이것저것 삽질을 하다가 결국 뒤 문제로 넘어가고 시간이 부족해 다시 돌아오지 못했다.
이 문제의 첫걸음은 바로 `입력 제한사항` 을 보고 `브루트포스` 방식으로 충분히 해결 가능하다는 것을 깨닫는 것이다. 이 무식하면서도 가장 강력한 방법을 간과하면 안된다.

문제를 말 그대로 받아들이고 가장 직관적으로 접근하는 것이다. `order` 배열에서 가능한 모든 조합을 구하고 `unordered_map` 해시 자료구조를 이용해 조합을 카운팅한다.
이 때, 문자 길이별로 가장 큰 카운팅을 지속적으로 갱신한다. 또한 문자의 길이와 카운팅으로 분류하여 모든 조합을 `vector` 에 저장해둔다. 아래 전체 코드를 확인하면 좀 더
수월하게 이해할 수 있을 것이다. 이 작업을 모든 조합에 대하여 수행하면 `course` 배열 쿼리를 아주 간단하고 상수 시간내에 수행할 수 있다.
다만 다른 방법보다 메모리를 많이 차지하지만 입력의 규모가 압도적으로 작아 메모리 사용량은 최대 `4MB` 를 넘지 않았다.

`order` 배열에서 모든 조합을 구하는 과정은 `재귀`를 통한 `백트래킹` 을 이용했다.

<br>

# 전체 코드

```c++
#include <string>
#include <vector>
#include <unordered_map>
#include <algorithm>
using namespace std;

int cnt[27]; // cnt[i]=n : 길이가 i인 조합 중 최대 주문 횟수는 n번
unordered_map<string,int> um; // um[str]=n : 조합 str의 주문 횟수는 n번
vector<string> menu[27][21]; // menu[i][j] : 길이가 i이고 j번 주문된 조합들의 목록

void comb(string s, int idx, string made){
    if(made.size()>1){ // 2번 이상 주문된 것만
        um[made]++; // 해당 조합 주문 횟수 누적
        cnt[made.size()]=max(cnt[made.size()],um[made]); // 조합 길이 별 최대 주문 횟수 갱신
        menu[made.size()][um[made]].push_back(made); // 분류별 조합 목록 추가
    }
    // 백트래킹
    for(int i=idx+1;i<s.size();i++){
        made.push_back(s[i]);
        comb(s,i,made);
        made.pop_back();
    }
}
vector<string> solution(vector<string> orders, vector<int> course) {
    vector<string> answer;
    
    // order 의 모든 조합 전처리
    for(string& s:orders){
        sort(s.begin(),s.end());
        comb(s,-1,"");
    }
    
    // 쿼리 수행
    for(int i:course)
        if(cnt[i]>1) // 길이가 i인 조합의 최대 주문 횟수가 1 이상인 경우만
            for(string s:menu[i][cnt[i]])
                answer.push_back(s);
    
    sort(answer.begin(),answer.end());
    
    return answer;
}
```
