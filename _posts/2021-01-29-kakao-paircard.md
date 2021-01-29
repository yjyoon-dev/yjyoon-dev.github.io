---
layout: post
title: "[프로그래머스] 카드 짝 맞추기 문제 풀이 (2021 카카오 코딩테스트)"
date: 2021-01-29
categories: kakao
photos: /assets/post_images/kakao/paircard.png
tags: [ps,algorithm,c++,kakao,programmers,bfs,dijkstra,backtracking,bruteforce,implementation]
description: "2021 카카오 블라인드 채용 코딩테스트 - 카드 짝 맞추기 C++ 풀이 (프로그래머스)"
---

<br>

게임 개발자인 베로니는 개발 연습을 위해 다음과 같은 간단한 카드 짝맞추기 보드 게임을 개발해 보려고 합니다.
게임이 시작되면 화면에는 카드 16장이 뒷면을 위로하여 4 x 4 크기의 격자 형태로 표시되어 있습니다. 각 카드의 앞면에는 카카오프렌즈 캐릭터 그림이 그려져 있으며, 8가지의 캐릭터 그림이 그려진 카드가 각기 2장씩 화면에 무작위로 배치되어 있습니다.
유저가 카드를 2장 선택하여 앞면으로 뒤집었을 때 같은 그림이 그려진 카드면 해당 카드는 게임 화면에서 사라지며, 같은 그림이 아니라면 원래 상태로 뒷면이 보이도록 뒤집힙니다. 이와 같은 방법으로 모든 카드를 화면에서 사라지게 하면 게임이 종료됩니다.

게임에서 카드를 선택하는 방법은 다음과 같습니다.

카드는 커서를 이용해서 선택할 수 있습니다.
커서는 4 x 4 화면에서 유저가 선택한 현재 위치를 표시하는 굵고 빨간 테두리 상자를 의미합니다.
커서는 [Ctrl] 키와 방향키에 의해 이동되며 키 조작법은 다음과 같습니다.
방향키 ←, ↑, ↓, → 중 하나를 누르면, 커서가 누른 키 방향으로 1칸 이동합니다.
[Ctrl] 키를 누른 상태에서 방향키 ←, ↑, ↓, → 중 하나를 누르면, 누른 키 방향에 있는 가장 가까운 카드로 한번에 이동합니다.
만약, 해당 방향에 카드가 하나도 없다면 그 방향의 가장 마지막 칸으로 이동합니다.
만약, 누른 키 방향으로 이동 가능한 카드 또는 빈 공간이 없어 이동할 수 없다면 커서는 움직이지 않습니다.
커서가 위치한 카드를 뒤집기 위해서는 [Enter] 키를 입력합니다.
[Enter] 키를 입력해서 카드를 뒤집었을 때
앞면이 보이는 카드가 1장 뿐이라면 그림을 맞출 수 없으므로 두번째 카드를 뒤집을 때 까지 앞면을 유지합니다.
앞면이 보이는 카드가 2장이 된 경우, 두개의 카드에 그려진 그림이 같으면 해당 카드들이 화면에서 사라지며, 그림이 다르다면 두 카드 모두 뒷면이 보이도록 다시 뒤집힙니다.
베로니는 게임 진행 중 카드의 짝을 맞춰 몇 장 제거된 상태에서 카드 앞면의 그림을 알고 있다면, 남은 카드를 모두 제거하는데 필요한 키 조작 횟수의 최솟값을 구해 보려고 합니다. 키 조작 횟수는 방향키와 [Enter] 키를 누르는 동작을 각각 조작 횟수 1로 계산하며, [Ctrl] 키와 방향키를 함께 누르는 동작 또한 조작 횟수 1로 계산합니다.

다음은 카드가 몇 장 제거된 상태의 게임 화면에서 커서를 이동하는 예시입니다.
아래 그림에서 빈 칸은 이미 카드가 제거되어 없어진 칸을 의미하며, 그림이 그려진 칸은 카드 앞 면에 그려진 그림을 나타냅니다.

![2021_kakao_card_01](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/bd1c06b3-6684-480a-85e6-53f1123b0770/2021_kakao_card_01.png)
예시에서 커서는 두번째 행, 첫번째 열 위치에서 시작하였습니다.

![2021_kakao_card_02](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/8d9008a0-a933-44c7-92a8-96b701483d6e/2021_kakao_card_02.png)
[Enter] 입력, ↓ 이동, [Ctrl]+→ 이동, [Enter] 입력 = 키 조작 4회

![2021_kakao_card_03](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/89b256d7-b8a8-4fb1-a1f4-84407a029d03/2021_kakao_card_03.png)
[Ctrl]+↑ 이동, [Enter] 입력, [Ctrl]+← 이동, [Ctrl]+↓ 이동, [Enter] 입력 = 키 조작 5회

![2021_kakao_card_04](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/96b37dbd-bba1-47e0-89e5-7a3e518eab24/2021_kakao_card_04.png)
[Ctrl]+→ 이동, [Enter] 입력, [Ctrl]+↑ 이동, [Ctrl]+← 이동, [Enter] 입력 = 키 조작 5회

위와 같은 방법으로 커서를 이동하여 카드를 선택하고 그림을 맞추어 카드를 모두 제거하기 위해서는 총 14번(방향 이동 8번, [Enter] 키 입력 6번)의 키 조작 횟수가 필요합니다.

<br>

# 문제

현재 카드가 놓인 상태를 나타내는 2차원 배열 board와 커서의 처음 위치 r, c가 매개변수로 주어질 때, 모든 카드를 제거하기 위한 키 조작 횟수의 최솟값을 return 하도록 solution 함수를 완성해 주세요.

<br>

# 제한 사항

- board는 4 x 4 크기의 2차원 배열입니다.
- board 배열의 각 원소는 0 이상 6 이하인 자연수입니다.
  - 0은 카드가 제거된 빈 칸을 나타냅니다.
  - 1 부터 6까지의 자연수는 2개씩 들어있으며 같은 숫자는 같은 그림의 카드를 의미합니다.
  - 뒤집을 카드가 없는 경우(board의 모든 원소가 0인 경우)는 입력으로 주어지지 않습니다.
- r은 커서의 최초 세로(행) 위치를 의미합니다.
- c는 커서의 최초 가로(열) 위치를 의미합니다.
- r과 c는 0 이상 3 이하인 정수입니다.
- 게임 화면의 좌측 상단이 (0, 0), 우측 하단이 (3, 3) 입니다.

<br>

# 입출력 예

board|r|c|result
---|---|---|---
[[1,0,0,3],[2,0,0,0],[0,0,0,2],[3,0,1,0]]|1|0|14
[[3,0,0,2],[0,0,1,0],[0,1,0,0],[2,0,0,3]]|0|1|16

<br>

# 풀이

`카카오 블라인드 테스트` 중후반 쯤에서 항상 등장하는 `최단거리` 와 `백트래킹` 이 결합된 `구현` 스타일의 문제입니다. `삼성 SW 역량테스트` 기출들과 비슷한 느낌의 문제입니다.

문제 풀이를 다음과 같이 2단계로 나눌 수 있습니다.

- 보드판의 카드를 뒤집는 모든 순서를 방문하는 `백트래킹` 구현
- 보드판의 칸을 이동할 때 필요한 `최소 키보드 입력 횟수` 계산

여기서 코드 제출 시에 `테스트케이스` 몇몇개에서만 `실패`가 뜰 경우 2번째 `최소 키보드 입력 횟수` 계산에 빈틈이 있을 확률이 큽니다. ~~제가 그랬거든요~~

한 번 구현을 실패하면 디버깅 시간이 엄청나게 걸리는 스타일의 문제기 때문에 문제를 잘 읽고 원큐에 정확히 푸는게 중요한 문제입니다. 제가 원큐를 실패해서 실제 시간 맞추고 풀 때 이 문제에 너무 많은 시간을 소비했습니다.

`백트래킹` 함수는 다음과 같이 정의할 수 있습니다.

> solve(board, y, x) : board[y][x] 의 위치에서 시작해 모든 종류의 카드 뒤집어보기

따라서 `solve` 함수가 한 번 수행될 때마다 임의의 카드 한 쌍을 뒤집고 두번째로 뒤집은 카드의 좌표에서 다시 `solve` 함수를 호출합니다. 카드 쌍을 뒤집을 때 둘 중 어떤 카드를 나중에 뒤집냐에 따라 다음 `solve` 함수의 매개변수가 달라지기 때문에 두 가지 경우를 모두 탐색해주어야 합니다. 

다음은 `최소 키보드 입력 횟수` 계산입니다. 고려해야 할 사항은 다음과 같습니다.

- 아무리 다음 카드가 멀리 있더라도 경로에 다른 카드가 없다면 한 번에 이동할 수 있다.
    - Ctrl + 방향키
    - 따라서 최단거리가 카드 간의 떨어진 칸 수에 비례하지 않는다 → `다익스트라` 필요
  
- Ctrl 키를 사용하지 않고 일일히 한 칸씩 이동하는 경우가 최단거리 일때도 있다.

- Enter 키는 카드 한 쌍 당 항상 2번씩 눌린다.

여기까지 정리가 되면 나머지는 `구현`입니다. 예외 없는 꼼꼼한 구현과 더불어 `다익스트라` 알고리즘에 대한 경험과 익숙함이 필요합니다.

<br>

# 전체 코드

```c++
#include <string>
#include <vector>
#include <queue>
using namespace std;

int const dy[]={-1,1,0,0};
int const dx[]={0,0,-1,1};

struct Point{
	int d,y,x;
	Point(int d,int y, int x):d(d),y(y),x(x){}
	
};

// 우선순위 큐 비교 연산자 오버라이딩
bool operator < (Point a, Point b){
	return a.d > b.d;
}

bool isFinished(vector<vector<int>>& board){
	for(auto v:board) for(int i:v) if(i!=0) return false;
	return true;
}

bool inRange(int y, int x){
	return y>=0 && x>=0 && y<4 && x<4;
}

// 다익스트라를 이용한 최소 키보드 입력 횟수 반환
int getDist(vector<vector<int>>& board, int y1, int x1, int y2, int x2){

    priority_queue<Point> q;
	q.push(Point(0,y1,x1));
	
    int dist[4][4];
	for(int i=0;i<4;i++)
		for(int j=0;j<4;j++)
			dist[i][j]=1e9;
	dist[y1][x1]=0;

	while(!q.empty()){
		Point cur = q.top();
		q.pop();

		int curDist = cur.d;

		if(dist[cur.y][cur.x]<cur.d) continue;

		if(cur.y==y2 && cur.x==x2) return curDist; // 완료

		for(int i=0;i<4;i++){
            int cnt=0;
			int ny=cur.y, nx=cur.x;

        // 한 칸씩 i방향으로 옮겨가며 최단거리 계산
			while(inRange(ny+dy[i],nx+dx[i])){
				cnt++;
				ny+=dy[i]; nx+=dx[i];

				if(board[ny][nx]!=0) break; // 카드 마주침

				if(dist[ny][nx]>curDist+cnt){
					dist[ny][nx]=curDist+cnt;
					q.push(Point(curDist+cnt,ny,nx));
				}
			}

        // 카드 또는 벽을 마주친 경우 Ctrl 키를 이용해 1번만에 이동 가능
			if(dist[ny][nx]>curDist+1){
					dist[ny][nx]=curDist+1;
					q.push(Point(curDist+1,ny,nx));
			}
		}
	}
}

// 백트래킹
int solve(vector<vector<int>>& board, int y, int x){
	if(isFinished(board)) return 0; // 전부 뒤집음

	int ret=1e9;

    // 카드 k 뒤집기
	for(int k=1;k<=6;k++){
			vector<pair<int,int>> point;
			for(int i=0;i<4;i++)
				for(int j=0;j<4;j++)
					if(board[i][j]==k)
						point.push_back(make_pair(i,j));

			if(point.empty()) continue;

        // 앞에꺼를 먼저 뒤집음
			int cand1 = getDist(board,y,x,point[0].first,point[0].second)
			+ getDist(board,point[0].first,point[0].second,point[1].first,point[1].second)
			+ 2;

        // 뒤에꺼를 먼저 뒤집음
			int cand2 = getDist(board,y,x,point[1].first,point[1].second)
			+ getDist(board,point[1].first,point[1].second,point[0].first,point[0].second)
			+ 2;
			
        // dfs
			board[point[0].first][point[0].second]=0;
			board[point[1].first][point[1].second]=0;

			ret=min(ret,min(cand1+solve(board,point[1].first,point[1].second),cand2+solve(board,point[0].first,point[0].second)));


			board[point[0].first][point[0].second]=k;
			board[point[1].first][point[1].second]=k;
	}
	return ret;
}

int solution(vector<vector<int>> board, int r, int c) {
    int answer = solve(board,r,c);
    return answer;
}
```

