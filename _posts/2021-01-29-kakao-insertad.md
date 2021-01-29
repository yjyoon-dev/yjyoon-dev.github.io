---
layout: post
title: "[프로그래머스] 광고 삽입 문제 풀이 (2021 카카오 코딩테스트)"
date: 2021-01-29
categories: kakao
photos: /assets/post_images/kakao/insertad.png
tags: [ps,algorithm,c++,kakao,programmers,two_pointer,string,implementation]
description: "2021 카카오 블라인드 채용 코딩테스트 - 광고 삽입 C++ 풀이 (프로그래머스)"
---

<br>

# 문제 설명

카카오TV에서 유명한 크리에이터로 활동 중인 죠르디는 환경 단체로부터 자신의 가장 인기있는 동영상에 지구온난화의 심각성을 알리기 위한 공익광고를 넣어 달라는 요청을 받았습니다. 평소에 환경 문제에 관심을 가지고 있던 죠르디는 요청을 받아들였고 광고효과를 높이기 위해 시청자들이 가장 많이 보는 구간에 공익광고를 넣으려고 합니다. 죠르디는 시청자들이 해당 동영상의 어떤 구간을 재생했는 지 알 수 있는 재생구간 기록을 구했고, 해당 기록을 바탕으로 공익광고가 삽입될 최적의 위치를 고를 수 있었습니다.
참고로 광고는 재생 중인 동영상의 오른쪽 아래에서 원래 영상과 동시에 재생되는 PIP(Picture in Picture) 형태로 제공됩니다.

![2021_kakao_cf_01](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/597ec277-4451-4289-8817-2970be644a69/2021_kakao_cf_01.png)

다음은 죠르디가 공익광고가 삽입될 최적의 위치를 고르는 과정을 그림으로 설명한 것입니다.

![2021_kakao_cf_02](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/e733fafb-1e6b-4d30-bbab-a22f366229e7/2021_kakao_cf_02.png)

<br>

# 문제

죠르디의 동영상 재생시간 길이 play_time, 공익광고의 재생시간 길이 adv_time, 시청자들이 해당 동영상을 재생했던 구간 정보 logs가 매개변수로 주어질 때, 시청자들의 누적 재생시간이 가장 많이 나오는 곳에 공익광고를 삽입하려고 합니다. 이때, 공익광고가 들어갈 시작 시각을 구해서 return 하도록 solution 함수를 완성해주세요. 만약, 시청자들의 누적 재생시간이 가장 많은 곳이 여러 곳이라면, 그 중에서 가장 빠른 시작 시각을 return 하도록 합니다.

<br>

# 제한 사항

- play_time, adv_time은 길이 8로 고정된 문자열입니다.
    - play_time, adv_time은 HH:MM:SS 형식이며, 00:00:01 이상 99:59:59 이하입니다.
    - 즉, 동영상 재생시간과 공익광고 재생시간은 00시간 00분 01초 이상 99시간 59분 59초 이하입니다.
    - 공익광고 재생시간은 동영상 재생시간보다 짧거나 같게 주어집니다.
- logs는 크기가 1 이상 300,000 이하인 문자열 배열입니다.
    - logs 배열의 각 원소는 시청자의 재생 구간을 나타냅니다.
    - logs 배열의 각 원소는 길이가 17로 고정된 문자열입니다.
    - logs 배열의 각 원소는 H1:M1:S1-H2:M2:S2 형식입니다.
        - H1:M1:S1은 동영상이 시작된 시각, H2:M2:S2는 동영상이 종료된 시각을 나타냅니다.
        - H1:M1:S1는 H2:M2:S2보다 1초 이상 이전 시각으로 주어집니다.
        - H1:M1:S1와 H2:M2:S2는 play_time 이내의 시각입니다.
- 시간을 나타내는 HH, H1, H2의 범위는 00~99, 분을 나타내는 MM, M1, M2의 범위는 00~59, 초를 나타내는 SS, S1, S2의 범위는 00~59까지 사용됩니다. 잘못된 시각은 입력으로 주어지지 않습니다. (예: 04:60:24, 11:12:78, 123:12:45 등)

- return 값의 형식
    - 공익광고를 삽입할 시각을 HH:MM:SS 형식의 8자리 문자열로 반환합니다.

<br>

# 입출력 예

play_time|adv_time|logs|result
---|---|---|---
"02:03:55"|"00:14:15"|["01:20:15-01:45:14", "00:40:31-01:00:00", "00:25:50-00:48:29", "01:30:59-01:53:29", "01:37:44-02:02:30"]|"01:30:59"
"99:59:59"|"25:00:00"|["69:59:59-89:59:59", "01:00:00-21:00:00", "79:59:59-99:59:59", "11:00:00-31:00:00"]|"01:00:00"
"50:00:00"|"50:00:00"|["15:36:51-38:21:49", "10:14:18-15:36:51", "38:21:49-42:51:45"]|"00:00:00"

<br>

# 풀이

문제가 요구하는 것은 간단하지만 `카카오`의 전매특허인 `시간 문자열` 이 나와서 읽기만해도 어질어질해지는 문제였습니다. 실제로 문제를 시간재고 풀 때 대충 훑어보고 가장 마지막에 풀었던 문제이기도 합니다. 하지만 막상 풀면 문제 자체는 크게 어렵지 않습니다.

문제를 단순화 하면 `전체 구간에서 특정 길이 만큼의 구간합이 가장 큰 구간을 구하는 문제`입니다.

우선 `HH:MM:SS` 형식의 문자열을 모두 초 단위로 변환하여 통일시켜주어야 합니다. 최대 입력값이 `99:59:59` 이므로 **100시간 = 360000초** 임을 보았을 때 초 단위 사이즈로 배열을 잡는데 무리가 없음을 알 수 있습니다.

> int ad[360000]

다음엔 배열에서 시청자들이 각각 광고를 재생하는 구간마다 `+1` 을 해줍니다. 그러면 다음이 성립합니다.

> ad[i] = n 이면 i시간 때 광고를 재생하는 시청자 수가 n명

여기까지 하면 이제 남은 것은 구간합 구하기 밖에 없습니다. 이 때 구간의 길이가 `adv_time` 으로 정해져있습니다. 단순히 `완전탐색` 을 이용할 경우 전체 구간의 길이를 `N`, 광고의 길이를 `M` 이라고 할 때 **O(N*M)** 의 시간복잡도가 걸립니다. 따라서 이 방법으로는 `효율성 테스트` 를 통과할 수 없습니다.

하지만 **구간의 길이가 항상 일정**하다는 점에 착안하면 **O(N)** 의 시간복잡도로 문제를 해결할 수 있습니다. 전체 구간에서 `0`번째 인덱스부터 `M-1` 인덱스까지 첫 `M` 개 원소 만큼의 구간합을 구했다고 합시다. 그 다음 구간의 구간합을 구할 때 굳이 `1`번째 인덱스부터 `M` 번째 인덱스까지 다시 구해야 할까요?

> 전에 구한 구간합에서 가장 첫 원소를 빼고 다음 원소 하나를 더해주면 다음 구간합과 같다.

즉, 구간을 한 칸씩 옮긴다고 생각하면 됩니다.

**가장 첫 원소를 뺀다** 라는 점에서 자연스레 `큐` 자료구조를 떠올릴 수 있습니다.

그리고 포인터를 구간의 양 끝에 2개를 유지한다는 점에서 이러한 알고리즘을 `투포인터` 알고리즘이라고 합니다.

`완전탐색` 대신 `투포인터` 알고리즘을 통해 구간합이 최대인 구간의 첫 인덱스를 갱신하면 답을 구할 수 있습니다.

<br>

# 전체 코드

```c++
#include <string>
#include <vector>
#include <queue>
using namespace std;

int ad[360000];

// 문자열 → 초 변환
int strToSec(string s){
	int ret=0;

	string h = s.substr(0,2);
	string m = s.substr(3,2);
	string c = s.substr(6,2);

	ret+=stoi(h)*60*60;
	ret+=stoi(m)*60;
	ret+=stoi(c);
	return ret;
}

// 초 → 문자열 변환
string secToStr(int n){
	string ret="";

 	int s = n%60; n/=60;
	int m = n%60; n/=60;
	int h = n;

	if(h<10) ret+="0";
	ret+=to_string(h);
	ret+=":";

	if(m<10) ret+="0";
	ret+=to_string(m);
	ret+=":";

	if(s<10) ret+="0";
	ret+=to_string(s);

    return ret;
}

string solution(string play_time, string adv_time, vector<string> logs) {
	string answer = "";

	for(string s:logs){
		int start = strToSec(s.substr(0,8));
		int finish = strToSec(s.substr(9,8));
		for(int i=start;i<finish;i++) ad[i]++; // 시청자 수 누적
	}

	int N = strToSec(play_time); // 전체 구간 길이
	int len = strToSec(adv_time); // 광고 구간 길이

	int idx=0;
	long long sum=0;
	long long maxSum=0;
	
	queue<int> q;

	for(int i=0;i<len;i++){
		sum+=ad[i];
		q.push(ad[i]);
	}
	maxSum=sum;

    // 큐를 이용한 투 포인터
	for(int i=len;i<N;i++){
		sum += ad[i];
		q.push(ad[i]);
		sum -= q.front();
		q.pop();
		if(sum > maxSum){
			idx = i-len+1;
			maxSum = sum;
		}
	}
	
    answer = secToStr(idx);
    return answer;
}
```




