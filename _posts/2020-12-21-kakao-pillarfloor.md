---
layout: post
title: "[2020 카카오 코딩테스트] 기둥과 보 설치 문제 풀이 (프로그래머스)"
date: 2020-12-21
categories: kakao
photos: /assets/post_images/kakao/pillarfloor.png
tags: [ps,algorithm,kakao,programmers,implementation,data_structure,set,c++,python]
description: "2020 카카오 블라인드 채용 코딩테스트 - 기둥과 보 설치 C++ / Python 간단 풀이 (프로그래머스)"
---

<br>

# 문제 설명

빙하가 깨지면서 스노우타운에 떠내려 온 죠르디는 인생 2막을 위해 주택 건축사업에 뛰어들기로 결심하였습니다. 죠르디는 기둥과 보를 이용하여 벽면 구조물을 자동으로 세우는 로봇을 개발할 계획인데, 그에 앞서 로봇의 동작을 시뮬레이션 할 수 있는 프로그램을 만들고 있습니다.
프로그램은 2차원 가상 벽면에 기둥과 보를 이용한 구조물을 설치할 수 있는데, 기둥과 보는 길이가 1인 선분으로 표현되며 다음과 같은 규칙을 가지고 있습니다.

- 기둥은 바닥 위에 있거나 보의 한쪽 끝 부분 위에 있거나, 또는 다른 기둥 위에 있어야 합니다.
- 보는 한쪽 끝 부분이 기둥 위에 있거나, 또는 양쪽 끝 부분이 다른 보와 동시에 연결되어 있어야 합니다.
  
단, 바닥은 벽면의 맨 아래 지면을 말합니다.

2차원 벽면은 `n x n` 크기 정사각 격자 형태이며, 각 격자는 `1 x 1` 크기입니다. 맨 처음 벽면은 비어있는 상태입니다. 기둥과 보는 격자선의 교차점에 걸치지 않고, 격자 칸의 각 변에 정확히 일치하도록 설치할 수 있습니다. 다음은 기둥과 보를 설치해 구조물을 만든 예시입니다.

![1](https://grepp-programmers.s3.amazonaws.com/files/production/c453630fa0/834b86e5-6fd0-4d3c-8023-7f853ea4301f.jpg)

예를 들어, 위 그림은 다음 순서에 따라 구조물을 만들었습니다.

1. (1, 0)에서 위쪽으로 기둥을 하나 설치 후, (1, 1)에서 오른쪽으로 보를 하나 만듭니다.
2. (2, 1)에서 위쪽으로 기둥을 하나 설치 후, (2, 2)에서 오른쪽으로 보를 하나 만듭니다.
3. (5, 0)에서 위쪽으로 기둥을 하나 설치 후, (5, 1)에서 위쪽으로 기둥을 하나 더 설치합니다.
4. (4, 2)에서 오른쪽으로 보를 설치 후, (3, 2)에서 오른쪽으로 보를 설치합니다.

만약 (4, 2)에서 오른쪽으로 보를 먼저 설치하지 않고, (3, 2)에서 오른쪽으로 보를 설치하려 한다면 2번 규칙에 맞지 않으므로 설치가 되지 않습니다. 기둥과 보를 삭제하는 기능도 있는데 기둥과 보를 삭제한 후에 남은 기둥과 보들 또한 위 규칙을 만족해야 합니다. 만약, 작업을 수행한 결과가 조건을 만족하지 않는다면 해당 작업은 무시됩니다.

벽면의 크기 n, 기둥과 보를 설치하거나 삭제하는 작업이 순서대로 담긴 2차원 배열 build_frame이 매개변수로 주어질 때, 모든 명령어를 수행한 후 구조물의 상태를 return 하도록 solution 함수를 완성해주세요.

<br>

# 제한사항

- n은 5 이상 100 이하인 자연수입니다.
- build_frame의 세로(행) 길이는 1 이상 1,000 이하입니다.
- build_frame의 가로(열) 길이는 4입니다.
- build_frame의 원소는 [x, y, a, b]형태입니다.
    - x, y는 기둥, 보를 설치 또는 삭제할 교차점의 좌표이며, [가로 좌표, 세로 좌표] 형태입니다.
    - a는 설치 또는 삭제할 구조물의 종류를 나타내며, 0은 기둥, 1은 보를 나타냅니다.
    - b는 구조물을 설치할 지, 혹은 삭제할 지를 나타내며 0은 삭제, 1은 설치를 나타냅니다.
    - 벽면을 벗어나게 기둥, 보를 설치하는 경우는 없습니다.
    - 바닥에 보를 설치 하는 경우는 없습니다.
- 구조물은 교차점 좌표를 기준으로 보는 오른쪽, 기둥은 위쪽 방향으로 설치 또는 삭제합니다.
- 구조물이 겹치도록 설치하는 경우와, 없는 구조물을 삭제하는 경우는 입력으로 주어지지 않습니다.
- 최종 구조물의 상태는 아래 규칙에 맞춰 return 해주세요.
    - return 하는 배열은 가로(열) 길이가 3인 2차원 배열로, 각 구조물의 좌표를 담고있어야 합니다.
    - return 하는 배열의 원소는 [x, y, a] 형식입니다.
    - x, y는 기둥, 보의 교차점 좌표이며, [가로 좌표, 세로 좌표] 형태입니다.
    - 기둥, 보는 교차점 좌표를 기준으로 오른쪽, 또는 위쪽 방향으로 설치되어 있음을 나타냅니다.
    - a는 구조물의 종류를 나타내며, 0은 기둥, 1은 보를 나타냅니다.
    - return 하는 배열은 x좌표 기준으로 오름차순 정렬하며, x좌표가 같을 경우 y좌표 기준으로 오름차순 정렬해주세요.
    - x, y좌표가 모두 같은 경우 기둥이 보보다 앞에 오면 됩니다.

<br>

# 풀이

처음 문제를 접하고 풀기 시작했을 때는 멘탈이 나갈 뻔 했다. 단순히 엄청난 양의 예외 처리가 필요한 빡구현 문제인 줄 알았다. 물론 그렇게도 풀 수 있겠지만 `이차원 배열` 의 선언 조차 없이 간단히 푸는 방법이 있었다. 아이디어는 다음과 같다.

- 구조물을 추가하는 경우
    - 구조물을 추가한 후 모든 구조물에 대하여 문제에서의 조건을 만족하는지 확인한다.
    - 만족하지 않는다면 추가한 구조물을 다시 제거한다.
- 구조물을 제거하는 경우
    - 구조물을 제거한 후 모든 구조물에 대하여 문제에서의 조건을 만족하는지 확인한다.
    - 만족하지 않는다면 제거한 구조물을 다시 추가한다.

위 아이디어에서 **문제의 조건**이란 다음과 같다.

- 기둥은 바닥 위에 있거나 보의 한쪽 끝 부분 위에 있거나, 또는 다른 기둥 위에 있어야 합니다.
- 보는 한쪽 끝 부분이 기둥 위에 있거나, 또는 양쪽 끝 부분이 다른 보와 동시에 연결되어 있어야 합니다.

즉, 건물에 변동이 있을 때마다 모든 구조물에 대하여 위 두가지 조건을 만족하는지 확인하는 것이다. 만족하지 않는다면 다시 롤백시킨다는 개념이다. 어떻게 생각하면 매우 직관적이고 단순한 아이디어지만 쉽게 떠올리고 적용할 수 없었던 이유는 바로 **너무 비효율적** 이라는 생각이 들기 때문일 것이다. 분명 매 구조물마다 건물 벡터를 일일이 뒤지는 것은 비효율적이지만 구현이 압도적으로 간단하고 위 문제와 같은 경우 `build_frame` 의 길이가 최대 `1000` 이기 때문에 충분히 빨리 돌아간다.

시간을 단축하는 방법으로 가장 먼저 떠오르는 것은 구조물을 저장하는 자료구조를 `vector` 가 아닌 `set` 을 사용함으로서 원소의 탐색 및 삭제 속도를 높이는 것이다. 그리고 이 방법을 사용하면 문제에서 제시한 정렬 기준 또한 `set` 의 특징으로 인해 별도의 추가 작업 없이 만족할 수 있다. 실제로 `set` 을 사용했을 때가 `vector` 를 사용했을 때보다 최악의 테스트 케이스에서 `10배` 정도 빠른 것을 확인할 수 있었다.

또 이 문제는 `python` 으로 푸는 것이 많이 유리한 유형의 문제라고 생각하기 때문에 `c++` 풀이와 `python` 풀이 두가지를 모두 작성했다.

<br>

# 전체 코드

## C++

```c++
#include <set>
#include <vector>
#define has(a) (building.find(pred[(a)])!=building.end())
using namespace std;

bool isValid(const set<vector<int>>& building){
	for(auto build:building){
		int x=build[0], y=build[1], a=build[2];

		if(a==0){
			vector<int> pred[3]={{x,y-1,0},{x,y,1},{x-1,y,1}};
			if(y==0 || has(0) || has(1) || has(2)) continue;
			return false;
		}
		else{
			vector<int> pred[4]={{x,y-1,0},{x+1,y-1,0},{x+1,y,1},{x-1,y,1}};
			if(has(0) || has(1) || (has(2)&&has(3))) continue;
			return false;
		}
	}

	return true;
}

vector<vector<int>> solution(int n, vector<vector<int>> build_frame) {
	vector<vector<int>> answer;
    set<vector<int>> building;

	for(auto task:build_frame){
		vector<int> v = {task[0],task[1],task[2]};
		if(task[3]==0){
			building.erase(v);
			if(!isValid(building)) building.insert(v);
		}
		else{
			building.insert(v);
			if(!isValid(building)) building.erase(v);
		}
	}

	for(auto build:building)
		answer.push_back(build);

    return answer;
}
```

## Python 3

```python
def isValid(answer):
    for x,y,a in answer:
        if a==0:
            if (x,y-1,0) in answer or (x-1,y,1) in answer or (x,y,1) in answer or y==0:
                continue
            else:
                return False
        if a==1:
            if (x,y-1,0) in answer or (x+1,y-1,0) in answer or ((x-1,y,1) in answer and (x+1,y,1) in answer):
                continue
            else:
                return False
    return True

def solution(n, build_frame):
    answer = set()
    for x,y,a,b in build_frame:
        if b==0:
            answer.remove((x,y,a))
            if not isValid(answer):
                answer.add((x,y,a))
        else:
            answer.add((x,y,a))
            if not isValid(answer):
                answer.remove((x,y,a))

    answer = [list(i) for i in answer]
    answer.sort(key=lambda x:(x[0],x[1],x[2]))
    return answer
```



