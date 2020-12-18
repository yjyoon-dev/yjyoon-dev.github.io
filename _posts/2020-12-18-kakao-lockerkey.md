---
layout: post
title: "[2020 카카오 코딩테스트] 자물쇠와 열쇠 문제 풀이 (프로그래머스)"
date: 2020-12-18
categories: kakao
photos: /assets/post_images/kakao/lockerkey.png
tags: [ps,algorithm,kakao,programmers,bruteforce,implementation]
description: "2020 카카오 블라인드 채용 코딩테스트 - 자물쇠와 열쇠 C++ 풀이 (프로그래머스)"
---

<br>

# 문제

고고학자인 튜브는 고대 유적지에서 보물과 유적이 가득할 것으로 추정되는 비밀의 문을 발견하였습니다. 그런데 문을 열려고 살펴보니 특이한 형태의 자물쇠로 잠겨 있었고 문 앞에는 특이한 형태의 열쇠와 함께 자물쇠를 푸는 방법에 대해 다음과 같이 설명해 주는 종이가 발견되었습니다.

잠겨있는 자물쇠는 격자 한 칸의 크기가 `1 x 1` 인 `N x N` 크기의 정사각 격자 형태이고 특이한 모양의 열쇠는 `M x M` 크기인 정사각 격자 형태로 되어 있습니다.

자물쇠에는 홈이 파여 있고 열쇠 또한 홈과 돌기 부분이 있습니다. 열쇠는 회전과 이동이 가능하며 열쇠의 돌기 부분을 자물쇠의 홈 부분에 딱 맞게 채우면 자물쇠가 열리게 되는 구조입니다. 자물쇠 영역을 벗어난 부분에 있는 열쇠의 홈과 돌기는 자물쇠를 여는 데 영향을 주지 않지만, 자물쇠 영역 내에서는 열쇠의 돌기 부분과 자물쇠의 홈 부분이 정확히 일치해야 하며 열쇠의 돌기와 자물쇠의 돌기가 만나서는 안됩니다. 또한 자물쇠의 모든 홈을 채워 비어있는 곳이 없어야 자물쇠를 열 수 있습니다.

열쇠를 나타내는 2차원 배열 key와 자물쇠를 나타내는 2차원 배열 lock이 매개변수로 주어질 때, 열쇠로 자물쇠를 열수 있으면 true를, 열 수 없으면 false를 return 하도록 solution 함수를 완성해주세요.

<br>

# 제한사항

- key는 M x M(3 ≤ M ≤ 20, M은 자연수)크기 2차원 배열입니다.
- lock은 N x N(3 ≤ N ≤ 20, N은 자연수)크기 2차원 배열입니다.
- M은 항상 N 이하입니다.
- key와 lock의 원소는 0 또는 1로 이루어져 있습니다.
    - 0은 홈 부분, 1은 돌기 부분을 나타냅니다.

<br>

# 입출력 예

|key|lock|result|
|-------|-------|-----|
|[[0, 0, 0], [1, 0, 0], [0, 1, 1]]|[[1, 1, 1], [1, 1, 0], [1, 0, 1]]|true|

<br>

# 입출력 예에 대한 설명

![1](https://grepp-programmers.s3.amazonaws.com/files/production/469703690b/79f2f473-5d13-47b9-96e0-a10e17b7d49a.jpg)

key를 시계 방향으로 90도 회전하고, 오른쪽으로 한 칸, 아래로 한 칸 이동하면 lock의 홈 부분을 정확히 모두 채울 수 있습니다.

<br>

# 풀이

이 문제를 보고 처음에는 기발한 아이디어를 떠올려야 한다고 생각할 수 있으나 모든 경우의 수를 확인해보는 `완전탐색` 문제이다.

접근 방식은 자물쇠와 열쇠가 모두 들어가는 충분히 큰 판 위에 자물쇠를 올려두고 열쇠를 모든 위치와 모든 방향에 대하여 놓아보는 것이다. 따라서 이 문제에서 절대적으로 필요한 것은 `구현` 능력이다. 열쇠를 `회전` 시킬 줄 알아야하고, 반복문을 통한 `완전탐색` 코드를 정확히 구현할 수 있어야 한다.

코드의 실행 시간이나 메모리를 자잘하게 절약하는 기법으로 자물쇠와 열쇠를 올려둘 큰 판의 사이즈를 최소화 한다거나 자물쇠의 빈 칸을 미리 세어두거나 하는 등의 여러 방법들이 있지만 필자는 문제가 요구하는 실행 시간 제한과 메모리 제한을 준수하면서 최대한 직관적인 방식의 코드로 구현하였다.

<br>

# 전체 코드

```c++
#include <vector>
using namespace std;

int N,M;
vector<vector<int>> board;

// 잠금 해제 여부 체크
bool check(vector<vector<int>>& key, int y, int x){
	bool ret = true;

	// 보드판에 열쇠 값 적용
	for(int i=y;i<y+M;i++)
		for(int j=x;j<x+M;j++)
			board[i][j]+=key[i-y][j-x];

	// 자물쇠의 모든 좌표 확인
	for(int i=M;i<M+N;i++){
		for(int j=M;j<M+N;j++){
			if(board[i][j]!=1){
				ret=false;
				break;
			}
		}
		if(!ret) break;
	}

	// 보드판에 열쇠 값 해제
	for(int i=y;i<y+M;i++)
		for(int j=x;j<x+M;j++)
			board[i][j]-=key[i-y][j-x];

	return ret;
}

// 열쇠 시계방향 회전
void rotate(vector<vector<int>>& key){
	vector<vector<int>> temp(M,vector<int>(M));

	for(int i=0;i<M;i++)
		for(int j=0;j<M;j++)
			temp[i][j] = key[j][M-i-1];

	for(int i=0;i<M;i++)
		for(int j=0;j<M;j++)
			key[i][j] = temp[i][j];
}

bool solution(vector<vector<int>> key, vector<vector<int>> lock) {
    bool answer = false;

	M = key.size();
	N = lock.size();

	board = vector<vector<int>>(2*M+N,vector<int>(2*M+N));

	// 보드판에 좌물쇠 입력
	for(int i=M;i<M+N;i++)
		for(int j=M;j<M+N;j++)
			board[i][j]=lock[i-M][j-M];
	
	// 보드판의 모든 좌표에서 4방향에 대한 완전탐색
	for(int i=0;i<N+M;i++){
		for(int j=0;j<N+M;j++){
			for(int k=0;k<4;k++){
				rotate(key);
				if(check(key,i,j)){
					answer = true;
					break;
				}
			}
			if(answer) break;
		}
		if(answer) break;
	}

    return answer;
}
```



