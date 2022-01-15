---
layout: post
title: "[프로그래머스] 거리두기 확인하기 풀이 (2021 카카오 인턴십)"
date: 2021-07-21
categories: kakao
photos: /assets/post_images/kakao/checkdistkeep.png
tags: [ps, kakao, programmers, algorithm, c++, dfs, bfs, implementation]
description: "프로그래머스 - 거리두기 확인하기 C++ 풀이 (2021 카카오 인턴십 코딩테스트)"
---

<br>

# 문제 설명

개발자를 희망하는 죠르디가 카카오에 면접을 보러 왔습니다.

코로나 바이러스 감염 예방을 위해 응시자들은 거리를 둬서 대기를 해야하는데 개발 직군 면접인 만큼
아래와 같은 규칙으로 대기실에 거리를 두고 앉도록 안내하고 있습니다.

> 1. 대기실은 5개이며, 각 대기실은 5x5 크기입니다.
> 2. 거리두기를 위하여 응시자들 끼리는 맨해튼 거리1가 2 이하로 앉지 말아 주세요.
> 3. 단 응시자가 앉아있는 자리 사이가 파티션으로 막혀 있을 경우에는 허용합니다.

예를 들어,

![1](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/8c056cac-ec8f-435c-a49a-8125df055c5e/PXP.png)|![2](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/d611f66e-f9c4-4433-91ce-02887657fe7f/PX_XP.png)|![3](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/ed707158-0511-457b-9e1a-7dbf34a776a5/PX_OP.png)
---|---|---|
위 그림처럼 자리 사이에 파티션이 존재한다면 맨해튼 거리가 2여도 거리두기를 **지킨 것입니다.**|위 그림처럼 파티션을 사이에 두고 앉은 경우도 거리두기를 **지킨 것입니다.**|위 그림처럼 자리 사이가 맨해튼 거리 2이고 사이에 빈 테이블이 있는 경우는 거리두기를 **지키지 않은 것입니다.**

![1](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/4c548421-1c32-4947-af9e-a45c61501bc4/P.png)|![2](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/ce799a38-668a-4038-b32f-c515b8701262/O.png)|![3](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/91e8f98b-baeb-4f81-8cb6-5bafebebdcc7/X.png)
---|---|---|
응시자가 앉아있는 자리(P)를 의미합니다.|빈 테이블(O)을 의미합니다.|파티션(X)을 의미합니다.

5개의 대기실을 본 죠르디는 각 대기실에서 응시자들이 거리두기를 잘 기키고 있는지 알고 싶어졌습니다. 자리에 앉아있는 응시자들의 정보와 대기실 구조를 대기실별로 담은 2차원 문자열 배열 `places`가 매개변수로 주어집니다. 각 대기실별로 거리두기를 지키고 있으면 1을, 한 명이라도 지키지 않고 있으면 0을 배열에 담아 return 하도록 solution 함수를 완성해 주세요.

<br>

# 제한사항

- `places`의 행 길이(대기실 개수) = 5
    - `places`의 각 행은 하나의 대기실 구조를 나타냅니다.
- `places`의 열 길이(대기실 세로 길이) = 5
- `places`의 원소는 `P`,`O`,`X`로 이루어진 문자열입니다.
    - `places` 원소의 길이(대기실 가로 길이) = 5
    - `P`는 응시자가 앉아있는 자리를 의미합니다.
    - `O`는 빈 테이블을 의미합니다.
    - `X`는 파티션을 의미합니다.
- 입력으로 주어지는 5개 대기실의 크기는 모두 5x5 입니다.
- return 값 형식
    - 1차원 정수 배열에 5개의 원소를 담아서 return 합니다.
    - `places`에 담겨 있는 5개 대기실의 순서대로, 거리두기 준수 여부를 차례대로 배열에 담습니다.
    - 각 대기실 별로 모든 응시자가 거리두기를 지키고 있으면 1을, 한 명이라도 지키지 않고 있으면 0을 담습니다.

<br>

# 풀이

기업에 관계없이 코딩테스트에서 가장 흔한 유형인 `그래프 탐색` 및 `구현` 문제이다. **삼성**에서 거의 고정적으로 출제하고 있는 스타일의 문제이며 **카카오**에서는 주로 뒷부분에서 상당히 까다로운 형태로 내는 유형의 문제였는데 이번에는 아주 가벼운 형태로 앞부분에서 등장하였다.

이 문제는 `DFS`와 `BFS` 2가지 방법으로 모두 풀이 가능하며, 제한사항에서 그래프의 크기가 `5x5`로 고정인 것을 감안하였을 때 `시간 초과` 및 `메모리 초과`를 신경 쓸 필요가 없으므로 비교적 코드 길이가 짧고 구현이 간단한 `DFS`를 이용하여 빠르게 푸는 것이 팁이라면 팁이다.

풀이 아이디어는 모든 `P`에 대해서 깊이 `2`를 초과하지 않는 선에서 탐색을 진행하고 도중에 또다른 `P`를 마주칠 경우 거리두기를 지키지 않는다고 판단하는 것이다. 이 때 파티션 `X`는 탐색할 수 없음을 주의한다.

<br>

# 전체 코드

```c++
#include <string>
#include <vector>
#include <cstring>
using namespace std;

bool flag;
bool visited[5][5];
const int dy[]={-1,1,0,0};
const int dx[]={0,0,-1,1};

bool inRange(int y, int x) {
    return y>=0 && y<5 && x>=0 && x<5;
}

void dfs(int y, int x, vector<string>& board, int depth) {
    // base case
    if(flag || depth>2) return;

    // oops
    if(depth>0 && board[y][x]=='P') {
        flag=true;
        return;
    }

    visited[y][x]=true;

    for(int i=0;i<4;i++){
        int ny=y+dy[i];
        int nx=x+dx[i];
        if(!inRange(ny,nx) || visited[ny][nx] || board[ny][nx]=='X')
            continue;
        dfs(ny,nx,board,depth+1);
    }
}

bool solve(vector<string>& board) {
    vector<pair<int,int>> people;
    for(int i=0;i<board.size();i++)
        for(int j=0;j<board[i].size();j++)
            if(board[i][j]=='P') people.push_back({i,j});

    for(auto pii:people){
        flag=false;
        memset(visited,false,sizeof(visited));
        dfs(pii.first,pii.second,board,0);
        if(flag) return false;
    }

    return true;
}

vector<int> solution(vector<vector<string>> places) {
    vector<int> answer;
    for(auto b:places){
        if(solve(b)) answer.push_back(1);
        else answer.push_back(0);
    }
    return answer;
}
```