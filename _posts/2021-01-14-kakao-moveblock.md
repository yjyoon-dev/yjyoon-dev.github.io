---
layout: post
title: "[2020 카카오 코딩테스트] 블록 이동하기 문제 풀이 (프로그래머스)"
date: 2021-01-14
categories: kakao
photos: /assets/post_images/kakao/moveblock.png
tags: [ps,algorithm,c++,kakao,programmers,bfs,implementation]
description: "2020 카카오 블라인드 채용 코딩테스트 - 블록 이동하기 C++ 풀이 (프로그래머스)"
---

<br>

# 문제 설명

로봇개발자 무지는 한 달 앞으로 다가온 카카오배 로봇경진대회에 출품할 로봇을 준비하고 있습니다. 준비 중인 로봇은 `2 x 1` 크기의 로봇으로 무지는 0과 1로 이루어진 `N x N` 크기의 지도에서 `2 x 1` 크기인 로봇을 움직여 (N, N) 위치까지 이동 할 수 있도록 프로그래밍을 하려고 합니다. 로봇이 이동하는 지도는 가장 왼쪽, 상단의 좌표를 (1, 1)로 하며 지도 내에 표시된 숫자 0은 빈칸을 1은 벽을 나타냅니다. 로봇은 벽이 있는 칸 또는 지도 밖으로는 이동할 수 없습니다. 로봇은 처음에 아래 그림과 같이 좌표 (1, 1) 위치에서 가로방향으로 놓여있는 상태로 시작하며, 앞뒤 구분없이 움직일 수 있습니다.

![1](https://grepp-programmers.s3.amazonaws.com/files/production/33f5c19ba6/052d3514-5fca-4b85-82aa-0f9eaefae0a3.jpg)

로봇이 움직일 때는 현재 놓여있는 상태를 유지하면서 이동합니다. 예를 들어, 위 그림에서 오른쪽으로 한 칸 이동한다면 (1, 2), (1, 3) 두 칸을 차지하게 되며, 아래로 이동한다면 (2, 1), (2, 2) 두 칸을 차지하게 됩니다. 로봇이 차지하는 두 칸 중 어느 한 칸이라도 (N, N) 위치에 도착하면 됩니다.

로봇은 다음과 같이 조건에 따라 회전이 가능합니다.

![2](https://grepp-programmers.s3.amazonaws.com/files/production/edfcdf57d3/f87055df-91e5-4f47-b99a-400c54bfdf3a.jpg)

위 그림과 같이 로봇은 90도씩 회전할 수 있습니다. 단, 로봇이 차지하는 두 칸 중, 어느 칸이든 축이 될 수 있지만, 회전하는 방향(축이 되는 칸으로부터 대각선 방향에 있는 칸)에는 벽이 없어야 합니다. 로봇이 한 칸 이동하거나 90도 회전하는 데는 걸리는 시간은 정확히 1초 입니다.

0과 1로 이루어진 지도인 board가 주어질 때, 로봇이 (N, N) 위치까지 이동하는데 필요한 최소 시간을 return 하도록 solution 함수를 완성해주세요.

<br>

# 제한사항

- board의 한 변의 길이는 5 이상 100 이하입니다.
- board의 원소는 0 또는 1입니다.
- 로봇이 처음에 놓여 있는 칸 (1, 1), (1, 2)는 항상 0으로 주어집니다.
- 로봇이 항상 목적지에 도착할 수 있는 경우만 입력으로 주어집니다.

<br>

# 입출력 예

|board|result|
|---|---|
|[[0, 0, 0, 1, 1],[0, 0, 0, 1, 0],[0, 1, 0, 1, 1],[1, 1, 0, 0, 1],[0, 0, 0, 0, 0]]|7|

<br>

# 입출력 예에 대한 설명

문제에 주어진 예시와 같습니다.
로봇이 오른쪽으로 한 칸 이동 후, (1, 3) 칸을 축으로 반시계 방향으로 90도 회전합니다. 다시, 아래쪽으로 3칸 이동하면 로봇은 (4, 3), (5, 3) 두 칸을 차지하게 됩니다. 이제 (5, 3)을 축으로 시계 방향으로 90도 회전 후, 오른쪽으로 한 칸 이동하면 (N, N)에 도착합니다. 따라서 목적지에 도달하기까지 최소 7초가 걸립니다.

<br>

# 풀이

이 문제는 `2020 카카오 코딩테스트` 에서 순수 구현시간이 가장 오래걸리는 문제이다. 그렇기 때문에 처음 풀 때 정확하고 꼼꼼하게 풀어야 한다. 그렇지 않으면 디버깅에 막대한 시간을 뺏길 수 밖에 없다.

이 문제가 단순한 `BFS` 문제보다 까다로운 이유는 바로 **로봇이 2칸을 차지**한다는 점과 **로봇의 회전** 때문이다. 이 2가지를 고려하여 얼마나 직관적이고 깔끔하게 구현하냐에 따라 문제풀이 시간이 달라진다.

필자는 우선 **로봇 구조체**를 선언하고 로봇이 차지하는 2칸의 좌표를 4개의 변수로 표현하였다. 그리고 구조체 안에 다음과 같은 함수들을 구현했다.

- 방향 D에 대한 평행이동
- 방향 D에 대한 회전
- 로봇의 유효성 검사(범위검사 및 벽 검사)
- 도착 여부 검사

이외에 이 구조체를 `map` 의 `key` 로 사용하고 상태에 대한 중복 검사를 하기 위해 **비교연산자**에 대한 `오버라이딩`을 하였다.

이처럼 구현의 핵심적인 부분들을 미리 `구조체` 안에 구현해두면 `BFS` 함수 부분이 쓸데없이 복잡해지는 것을 방지하고 직관적인 코드를 작성하여 실수를 줄일 수 있다.

<br>

다음은 **로봇의 회전** 이다. 이는 모든 경우에 대하여 하나하나 일일이 구현할 경우 코드가 굉장히 지저분해지므로 실수가 발생하고 추후 디버깅에 애를 먹을 수 있다. 여기서 로봇의 회전에 대하여 약간의 고찰을 하면 다음과 같은 사실을 알 수 있다.

> **로봇이 방향 D로 평행이동이 가능할 경우 방향 D에 대한 2가지 회전 모두 가능하다.**

즉 로봇의 상태가 수직이건 수평이건 관계 없이 위와 같은 동일한 방법으로 모든 회전 가능한 상태를 얻어낼 수 있다. 이 때 로봇의 평행이동과 유효성 검사를 `구조체`에서 이미 구현해두었으므로 로봇의 회전 상태도 아주 간결한 코드로 얻어낼 수 있다.

<br>

# 전체 코드

```c++
#include <iostream>
#include <vector>
#include <tuple>
#include <map>
#include <queue>
using namespace std;

int N;
vector<vector<int>> board;
const int dy[] = {-1,1,0,0};
const int dx[] = {0,0,-1,1};

struct Robot{
    // 로봇의 좌표
    int y1,x1,y2,x2;
    Robot(int y1, int x1, int y2, int x2): y1(y1), x1(x1), y2(y2), x2(x2){}

    // 방향 D에 대한 평행이동
    Robot move(int d){
        return Robot(y1+dy[d], x1+dx[d], y2+dy[d], x2+dx[d]);
    }

    // 방향 D에 대한 회전
    pair<Robot,Robot> rotate(int d){
        return {Robot(y1, x1, y1+dy[d], x1+dx[d]), Robot(y2, x2, y2+dy[d], x2+dx[d])};
    }

    // 유효성 검사
    bool isValid(){
        return y1>=0 && x1>=0 && y2>=0 && x2>=0 &&
                y1<N && x1<N && y2<N && x2<N &&
                !board[y1][x1] && !board[y2][x2];
    }

    // 종료 검사
    bool isFinished(){
        return y1==N-1&&x1==N-1 || y2==N-1&&x2==N-1;
    }
};

// Robot 구조체를 map의 원소로 사용하기 위한 오버라이딩
bool operator < (Robot a, Robot b){
    return tie(a.y1, a.x1, a.y2, a.x2) < tie(b.y1, b.x1, b.y2, b.x2);
}

// map 안에서의 Robot 중복 처리를 위한 오버라이딩
bool operator == (Robot a, Robot b){
     return tie(a.y1, a.x1, a.y2, a.x2)==tie(b.y1, b.x1, b.y2, b.x2)
    || tie(a.y1, a.x1, a.y2, a.x2)==tie(b.y2, b.x2, b.y1, b.x1);
}

map<Robot, int> dist;

// 로봇의 평행이동 반환
vector<Robot> getMoved(Robot& r){
    vector<Robot> ret;
    for(int i=0;i<4;i++)
        ret.push_back(r.move(i));
    return ret;
}

// 로봇의 회전 반환
vector<Robot> getRotated(Robot& r){
    vector<Robot> ret;
    for(int i=0;i<4;i++){
        // 방향 D에 대하여 평행이동이 가능하면
        // 방향 D에 대한 회전 2가지 모두 가능
        if(r.move(i).isValid()){
            pair<Robot,Robot> res = r.rotate(i);
            ret.push_back(res.first);
            ret.push_back(res.second);
        }
    }
    return ret;
}

int solve(Robot init){
    dist[init] = 0;

    queue<Robot> q;
    q.push(init);

    while(!q.empty()){
        Robot cur = q.front();
        q.pop();

        int curDist = dist[cur];
        if(cur.isFinished()) return curDist; // 목적지 도착

        // 다음 상태: 1) 평행이동, 2) 회전
        vector<Robot> next[2] = {getMoved(cur), getRotated(cur)};
        for(int i=0;i<2;i++){
            for(Robot r: next[i]){
                // 이 상태가 방문한 적이 없고 유효한 상태일 경우
                if(dist.find(r)==dist.end() && r.isValid()){
                    dist[r] = curDist+1;
                    q.push(r);
                }
            }
        }
    }
    return -1; // 목적지 도달 실패 (문제에서는 주어지지 않는 상황)
}

int solution(vector<vector<int>> b){
    board = b;
    N = board.size(); // 편리를 위한 전역 선언

    int answer = solve(Robot(0,0,0,1));

    return answer;
}
```