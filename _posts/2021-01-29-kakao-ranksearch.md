---
layout: post
title: "[프로그래머스] 순위 검색 문제 풀이 (2021 카카오 코딩테스트)"
date: 2021-01-29
categories: kakao
photos: /assets/post_images/kakao/ranksearch.png
tags: [ps,algorithm,c++,kakao,programmers,data_structure,binary_search]
description: "2021 카카오 블라인드 채용 코딩테스트 - 순위 검색 C++ 풀이 (프로그래머스)"
---

<br>

# 문제 설명

카카오는 하반기 경력 개발자 공개채용을 진행 중에 있으며 현재 지원서 접수와 코딩테스트가 종료되었습니다. 이번 채용에서 지원자는 지원서 작성 시 아래와 같이 4가지 항목을 반드시 선택하도록 하였습니다.

코딩테스트 참여 개발언어 항목에 cpp, java, python 중 하나를 선택해야 합니다.
지원 직군 항목에 backend와 frontend 중 하나를 선택해야 합니다.
지원 경력구분 항목에 junior와 senior 중 하나를 선택해야 합니다.
선호하는 소울푸드로 chicken과 pizza 중 하나를 선택해야 합니다.
인재영입팀에 근무하고 있는 니니즈는 코딩테스트 결과를 분석하여 채용에 참여한 개발팀들에 제공하기 위해 지원자들의 지원 조건을 선택하면 해당 조건에 맞는 지원자가 몇 명인 지 쉽게 알 수 있는 도구를 만들고 있습니다.
예를 들어, 개발팀에서 궁금해하는 문의사항은 다음과 같은 형태가 될 수 있습니다.
`코딩테스트에 java로 참여했으며, backend 직군을 선택했고, junior 경력이면서, 소울푸드로 pizza를 선택한 사람 중 코딩테스트 점수를 50점 이상 받은 지원자는 몇 명인가?`

물론 이 외에도 각 개발팀의 상황에 따라 아래와 같이 다양한 형태의 문의가 있을 수 있습니다.

- 코딩테스트에 python으로 참여했으며, frontend 직군을 선택했고, senior 경력이면서, 소울푸드로 chicken을 선택한 사람 중 코딩테스트 점수를 100점 이상 받은 사람은 모두 몇 명인가?
- 코딩테스트에 cpp로 참여했으며, senior 경력이면서, 소울푸드로 pizza를 선택한 사람 중 코딩테스트 점수를 100점 이상 받은 사람은 모두 몇 명인가?
- backend 직군을 선택했고, senior 경력이면서 코딩테스트 점수를 200점 이상 받은 사람은 모두 몇 명인가?
- 소울푸드로 chicken을 선택한 사람 중 코딩테스트 점수를 250점 이상 받은 사람은 모두 몇 명인가?
- 코딩테스트 점수를 150점 이상 받은 사람은 모두 몇 명인가?
즉, 개발팀에서 궁금해하는 내용은 다음과 같은 형태를 갖습니다.

> [조건]을 만족하는 사람 중 코딩테스트 점수를 X점 이상 받은 사람은 모두 몇 명인가?

<br>

# 문제

지원자가 지원서에 입력한 4가지의 정보와 획득한 코딩테스트 점수를 하나의 문자열로 구성한 값의 배열 info, 개발팀이 궁금해하는 문의조건이 문자열 형태로 담긴 배열 query가 매개변수로 주어질 때,
각 문의조건에 해당하는 사람들의 숫자를 순서대로 배열에 담아 return 하도록 solution 함수를 완성해 주세요.

<br>

# 제한 사항

- info 배열의 크기는 1 이상 50,000 이하입니다.
- info 배열 각 원소의 값은 지원자가 지원서에 입력한 4가지 값과 코딩테스트 점수를 합친 개발언어 직군 경력 소울푸드 점수 형식입니다.
    - 개발언어는 cpp, java, python 중 하나입니다.
    - 직군은 backend, frontend 중 하나입니다.
    - 경력은 junior, senior 중 하나입니다.
    - 소울푸드는 chicken, pizza 중 하나입니다.
    - 점수는 코딩테스트 점수를 의미하며, 1 이상 100,000 이하인 자연수입니다.
    - 각 단어는 공백문자(스페이스 바) 하나로 구분되어 있습니다.
- query 배열의 크기는 1 이상 100,000 이하입니다.
- query의 각 문자열은 [조건] X 형식입니다.
    - [조건]은 개발언어 and 직군 and 경력 and 소울푸드 형식의 문자열입니다.
    - 언어는 cpp, java, python, - 중 하나입니다.
    - 직군은 backend, frontend, - 중 하나입니다.
    - 경력은 junior, senior, - 중 하나입니다.
    - 소울푸드는 chicken, pizza, - 중 하나입니다.
    - '-' 표시는 해당 조건을 고려하지 않겠다는 의미입니다.
    - X는 코딩테스트 점수를 의미하며 조건을 만족하는 사람 중 X점 이상 받은 사람은 모두 몇 명인 지를 의미합니다.
    - 각 단어는 공백문자(스페이스 바) 하나로 구분되어 있습니다.
    - 예를 들면, cpp and - and senior and pizza 500은 cpp로 코딩테스트를 봤으며, 경력은 senior 이면서 소울푸드로 pizza를 선택한 지원자 중 코딩테스트 점수를 500점 이상 받은 사람은 모두 몇 명인가?를 의미합니다.

<br>

# 입출력 예

info|query|result
---|----|----|
["java backend junior pizza 150","python frontend senior chicken 210","python frontend senior chicken 150","cpp backend senior pizza 260","java backend junior chicken 80","python backend senior chicken 50"]|["java and backend and junior and pizza 100","python and frontend and senior and chicken 200","cpp and - and senior and pizza 250","- and backend and senior and - 150","- and - and - and chicken 100","- and - and - and - 150"]|[1,1,1,1,2,4]

<br>

# 풀이

`데이터베이스` 를 구축하고 `쿼리` 를 실행하는 형식의 문제입니다. 우선 사용자의 정보를 저장할 `자료구조` 를 정의해야합니다. 사용자의 정보는 다음과 같습니다.

- 언어: 3가지
- 분야: 2가지
- 경력: 2가지
- 음식: 2가지
- 점수: 정수형

여기서 `점수` 는 **검색**해야하는 정보이고, 나머지 `4가지`는 사용자를 **분류** 하기 위한 정보입니다. 따라서 사용자를 분류하는 기준이 총 `4가지` 이므로 `4차원 배열` 을 만들고, 특정 분류에 여러 사람이 분포할 수 있으므로 배열의 원소는 이 분류에 속한 사람들의 점수를 저장할 `vector` 로 선언해줍니다. 정리하면 다음과 같습니다.

> ***vector< int > db[i][j][k][l]: i언어, j분야, k경력, l음식인 사람들의 점수 목록***

이렇게 하면 `info` 배열로 주어지는 사용자들의 모든 정보는 수월히 저장할 수 있습니다. 다음 해결해야할 문제는 `query` 입니다. 모든 분야가 명확히 주어지는 쿼리는 아주 간단히 구현할 수 있습니다. 문제는 `-` 입니다. 이 경우는 `4중 for문` 을 통해 해결할 수 있습니다. 쿼리를 수행할 때 `-` 가 주어진 분류는 반복문의 범위를 전체 범위로 설정해주고 최종 답을 저장할 변수에 결과값을 누적시켜주면 됩니다.

마지막으로 **점수가 X점 이상인 사람들의 카운트** 입니다. 단순히 배열의 모든 원소를 하나씩 비교하여 카운트 하면 모든 사람을 비교하여 **O(N)** 의 시간복잡도가 나옵니다. 이 때 가능한 사람의 최대 수는 `50000`, 쿼리의 최대 수는 `100000` 이므로 최악의 경우 **50000번의 비교를 100000번** 하게 될 수도 있습니다. 따라서 `효율성 테스트`를 통과하지 못합니다. 어느 배열에서 **자신보다 크거나 같은 값의 개수**를 효율적으로 찾는 방법은 다음과 같은 아이디어를 통해 얻을 수 있습니다.

> 배열을 정렬한 후 자신이 배열의 몇번 째인지만 파악하면...?

예를들어 오름차순 정렬된 배열에서 `100점` 이상인 사람의 수를 셀 때 배열의 왼쪽에서 가장 처음으로 등장하는 `100점` 의 인덱스만 파악하면 알 수 있습니다. 위 말을 `알고리즘`으로 그대로 옮기면 바로 `이진 탐색`이 되겠죠.

따라서 이 문제는 적절한 `자료구조의 선언`과 `효율적인 탐색` 방법이 핵심이었습니다.

<br>

# 전체 코드

```c++
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

vector<int> db[3][2][2][2];

// query 문자열 쪼개기 함수
vector<string> querySplit(string s){
  vector<string> ret;
  string temp;
  for(char c:s){
    if(c==' '){
      ret.push_back(temp);
      temp="";
    }
    else temp+=c;
  }
  ret.push_back(temp);
  return ret;
}

// info 문자열 쪼개기 함수
vector<int> infoSplit(string s){
  vector<int> ret;
  string temp;
  int n;
  for(char c:s){
    if(c==' '){
      if(temp=="cpp" || temp=="backend" || temp=="junior" || temp=="chicken") n=0;
      else if(temp=="python") n=2;
      else n=1;
      ret.push_back(n);
      temp="";
    }
    else temp+=c;
  }
  ret.push_back(stoi(temp));
  return ret;
}

vector<int> solution(vector<string> info, vector<string> query) {
   vector<int> answer;

  // info 저장
  for(string s:info){
    vector<int> v = infoSplit(s);
    db[v[0]][v[1]][v[2]][v[3]].push_back(v[4]);
  }

    // 추후 이분 탐색을 위한 모든 배열 정렬
  for(int i=0;i<3;i++)
    for(int j=0;j<2;j++)
      for(int k=0;k<2;k++)
        for(int l=0;l<2;l++)
          sort(db[i][j][k][l].begin(),db[i][j][k][l].end());

    // 쿼리 수행
  for(string s:query){
    vector<string> v = querySplit(s);

    // 쿼리 조건에 따른 구간 정의
    int ai,bi,aj,bj,ak,bk,al,bl;

    if(v[0]=="cpp") ai=bi=0;
    else if(v[0]=="java") ai=bi=1;
    else if(v[0]=="python") ai=bi=2;
    else {ai=0; bi=2;} // '-'

    if(v[2]=="backend") aj=bj=0;
    else if(v[2]=="frontend") aj=bj=1;
    else {aj=0; bj=1;};

    if(v[4]=="junior") ak=bk=0;
    else if(v[4]=="senior") ak=bk=1;
    else {ak=0; bk=1;} // '-'

    if(v[6]=="chicken") al=bl=0;
    else if(v[6]=="pizza") al=bl=1;
    else {al=0; bl=1;} // '-'

    int score = stoi(v[7]);

    // 점수가 X점 이상인 사람의 수 계산
    int ans=0;
    for(int i=ai;i<=bi;i++){
      for(int j=aj;j<=bj;j++){
        for(int k=ak;k<=bk;k++){
          for(int l=al;l<=bl;l++){
            int n = db[i][j][k][l].size();
            if(n==0) continue;

            // 이분 탐색
            auto iter = lower_bound(db[i][j][k][l].begin(),db[i][j][k][l].end(),score);

            if(iter == db[i][j][k][l].end()) continue;
            ans += n-(iter-db[i][j][k][l].begin());
          }
        }
      }
    }
    answer.push_back(ans);
  }

  return answer;
}
```



