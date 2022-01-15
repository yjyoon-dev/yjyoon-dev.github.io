---
layout: post
title: "[프로그래머스] 숫자 문자열과 영단어 풀이 (2021 카카오 인턴십)"
date: 2021-07-12
categories: kakao
photos: /assets/post_images/kakao/numstrengword.png
tags: [ps, kakao, programmers, algorithm, c++, dynamic_programming]
description: "프로그래머스 - 숫자 문자열과 영단어 C++ 풀이 (2021 카카오 인턴십 코딩테스트)"
---

<br>

# 문제 설명

![0](https://grepp-programmers.s3.ap-northeast-2.amazonaws.com/files/production/d31cb063-4025-4412-8cbc-6ac6909cf93e/img1.png)

네오와 프로도가 숫자놀이를 하고 있습니다. 네오가 프로도에게 숫자를 건넬 때 일부 자릿수를 영단어로 바꾼 카드를 건네주면 프로도는 원래 숫자를 찾는 게임입니다.

다음은 숫자의 일부 자릿수를 영단어로 바꾸는 예시입니다.

- 1478 → "one4seveneight"
- 234567 → "23four5six7"
- 10203 → "1zerotwozero3"

이렇게 숫자의 일부 자릿수가 영단어로 바뀌어졌거나, 혹은 바뀌지 않고 그대로인 문자열 `s`가 매개변수로 주어집니다. `s`가 의미하는 원래 숫자를 return 하도록 solution 함수를 완성해주세요.

참고로 각 숫자에 대응되는 영단어는 다음 표와 같습니다.

숫자 | 영단어
--|-----
0 | zero
1 | one
2 | two
3 | three
4 | four
5 | five
6 | six
7 | seven
8 | eight
9 | nine

<br>

# 제한사항

- 1 ≤ `s`의 길이 ≤ 50
- `s`가 "zero" 또는 "0"으로 시작하는 경우는 주어지지 않습니다.
- return 값이 1 이상 2,000,000,000 이하의 정수가 되는 올바른 입력만 `s`로 주어집니다.

<br>

# 풀이

이번에도 1번 문제는 어김없이 `문자열` 문제로 시작했다. 문제 자체가 워낙 단순하기 때문에 알고리즘이나 풀이법을 떠올리기보단 **최대한 빨리 정확하게** `구현`해내는 것이 관건이었다. 이런 문제를 풀 때 한가지 주의해야 할 것은 다음과 같다.

> 괜히 코드를 최대한 깔끔하게 작성하려고 하다가 오답이 나오거나 시간이 지체될 수 있다.

이러한 문제의 특징은 코드가 결국 더러워질 수 밖에 없다는 것이다. 수많은 `if`문과 더불어 상당히 많은 양의 코드가 `중복`될 수 있다. 이것을 참지 못하고 문제를 품과 동시에 코드 설계까지 생각하려는 경우가 있는데 이렇게 하면 빠뜨리는 경우의 수가 생기거나 시간이 지체될 수 밖에 없다. 이러한 노가다 문제의 경우 어떻게든 풀기만 하면 된다는 마인드로 최대한 빠르게 기계적으로 코딩하고 다음 문제로 넘어가야 한다.

풀이 자체는 부분 문자열 함수인 `substr` 하나로 해결된다.

<br>

# 전체 코드

```c++
#include <string>
using namespace std;

bool isNum(char c) {
    return '0'<=c && c<='9';
}

int solution(string s) {
    int answer = 0;
    
    string str = "";
    for(int i = 0; i < s.size(); i++) {
        if(isNum(s[i])) {
            str.push_back(s[i]);
            continue;
        }
        if(s.size()-i>=5 && s.substr(i,5)=="three") str.push_back('3');
        else if(s.size()-i>=5 && s.substr(i,5)=="seven") str.push_back('7');
        else if(s.size()-i>=5 && s.substr(i,5)=="eight") str.push_back('8');
        else if(s.size()-i>=4 && s.substr(i,4)=="zero") str.push_back('0');
        else if(s.size()-i>=4 && s.substr(i,4)=="four") str.push_back('4');
        else if(s.size()-i>=4 && s.substr(i,4)=="five") str.push_back('5');
        else if(s.size()-i>=4 && s.substr(i,4)=="nine") str.push_back('9');
        else if(s.size()-i>=3 && s.substr(i,3)=="one") str.push_back('1');
        else if(s.size()-i>=3 && s.substr(i,3)=="two") str.push_back('2');
        else if(s.size()-i>=3 && s.substr(i,3)=="six") str.push_back('6');            
    }
    
    answer = stoi(str);
    return answer;
}
```