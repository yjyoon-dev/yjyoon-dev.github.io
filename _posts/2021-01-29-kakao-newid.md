---
layout: post
title: "[프로그래머스] 신규 아이디 추천 문제 풀이 (2021 카카오 코딩테스트)"
date: 2021-01-29
categories: kakao
photos: /assets/post_images/kakao/newid.png
tags: [ps, algorithm, c++, kakao, programmers, string, implementation]
description: "2021 카카오 블라인드 채용 코딩테스트 - 신규 아이디 추천 C++ 풀이 (프로그래머스)"
---

<br>

# 문제 설명

카카오에 입사한 신입 개발자 `네오`는 카카오계정개발팀에 배치되어, 카카오 서비스에 가입하는 유저들의 아이디를 생성하는 업무를 담당하게 되었습니다. 네오에게 주어진 첫 업무는 새로 가입하는 유저들이 카카오 아이디 규칙에 맞지 않는 아이디를 입력했을 때, 입력된 아이디와 유사하면서 규칙에 맞는 아이디를 추천해주는 프로그램을 개발하는 것입니다.
다음은 카카오 아이디의 규칙입니다.

- 아이디의 길이는 3자 이상 15자 이하여야 합니다.
- 아이디는 알파벳 소문자, 숫자, 빼기(`-`), 밑줄(`_`), 마침표(`.`) 문자만 사용할 수 있습니다.
- 단, 마침표(`.`)는 처음과 끝에 사용할 수 없으며 또한 연속으로 사용할 수 없습니다.

네오는 다음과 같이 7단계의 순차적인 처리 과정을 통해 신규 유저가 입력한 아이디가 카카오 아이디 규칙에 맞는 지 검사하고 규칙에 맞지 않은 경우 규칙에 맞는 새로운 아이디를 추천해 주려고 합니다.
신규 유저가 입력한 아이디가 `new_id` 라고 한다면,

1.  new_id의 모든 대문자를 대응되는 소문자로 치환합니다.
2.  new*id에서 알파벳 소문자, 숫자, 빼기(-), 밑줄(*), 마침표(.)를 제외한 모든 문자를 제거합니다.
3.  new_id에서 마침표(.)가 2번 이상 연속된 부분을 하나의 마침표(.)로 치환합니다.
4.  new_id에서 마침표(.)가 처음이나 끝에 위치한다면 제거합니다.
5.  new_id가 빈 문자열이라면, new_id에 "a"를 대입합니다.
6.  new_id의 길이가 16자 이상이면, new_id의 첫 15개의 문자를 제외한 나머지 문자들을 모두 제거합니다. 만약 제거 후 마침표(.)가 new_id의 끝에 위치한다면 끝에 위치한 마침표(.) 문자를 제거합니다.
7.  new_id의 길이가 2자 이하라면, new_id의 마지막 문자를 new_id의 길이가 3이 될 때까지 반복해서 끝에 붙입니다.

<br>

# 문제

신규 유저가 입력한 아이디를 나타내는 new_id가 매개변수로 주어질 때, 네오가 설계한 7단계의 처리 과정을 거친 후의 추천 아이디를 return 하도록 solution 함수를 완성해 주세요.

<br>

# 제한사항

new*id는 길이 1 이상 1,000 이하인 문자열입니다.
new_id는 알파벳 대문자, 알파벳 소문자, 숫자, 특수문자로 구성되어 있습니다.
new_id에 나타날 수 있는 특수문자는 `-*.~!@#$%^&\*()=+[{]}:?,<>/` 로 한정됩니다.

<br>

# 입출력 예

| no  | new_id                         | result            |
| --- | ------------------------------ | ----------------- |
| 예1 | "...!@BaT#\*..y.abcdefghijklm" | "bat.y.abcdefghi" |
| 예2 | "z-+.^."                       | "z--"             |
| 예3 | "=.="                          | "aaa"             |
| 예4 | "123\_.def"                    | "123\_.def"       |
| 예5 | "abcdefghijklmn.p"             | "abcdefghijklmn"  |

<br>

# 풀이

역시 올해도 1번 문제는 `문자열` 과 `구현` 이 결합된 형태의 문제가 출제되었습니다. 다만 작년의 [**문자열 압축**](https://yjyoon-dev.github.io/kakao/2020/11/05/kakao-strzip/) 문제에 비해 단순히 문제에 표시된 절차에 따라 그대로 구현만 해주면 된다는 점에서 난이도가 낮았습니다. 하지만 본인이 사용하는 언어에서 제공하는 `문자열` 관련 `함수`나 `라이브러리` 사용에 익숙하지 않다면 충분히 삽질을 할 수 있는 문제였습니다. 이 문제 풀이에 어려움을 겪었다면 알고리즘이 아닌 **자신이 사용하는 언어**에 대한 공부가 더 필요합니다.

<br>

# 전체 코드

```c++
#include <string>
#include <vector>
#include <cctype>
using namespace std;

string solution(string new_id) {
    string answer = "";

    // 1단계: 소문자로 변환
    for(char& c:new_id)
        c=tolower(c);

    // 2단계: 특정 문자를 제외하고 모두 삭제
    for(char c:new_id){
        if(c!='-'&&c!='_'&&c!='.'&&!('a'<=c&&c<='z')&&!('0'<=c&&c<='9'))
            continue;
        answer.push_back(c);
    }

    // 3단계: 연속되는 마침표 치환
    string temp;
    for(int i=0;i<answer.size();i++){
        if(answer[i]=='.'){
            temp.push_back('.');
            while(i<answer.size() && answer[i]=='.') i++;
            i--;
        }
        else temp.push_back(answer[i]);
    }
    answer=temp;

    // 4단계: 양 끝 마침표 제거
    if(answer[0]=='.') answer=answer.substr(1);
    if(answer[answer.size()-1]=='.') answer=answer.substr(0,answer.size()-1);

    // 5단계: 빈 문자열일 경우 "a" 로 치환
    if(answer.empty()) answer="a";

    // 6단계: 문자열 15자로 자르기 및 마지막 마침표 제거
    if(answer.size()>15) answer=answer.substr(0,15);
    if(answer[answer.size()-1]=='.') answer=answer.substr(0,answer.size()-1);

    // 7단계: 2자 이하일 경우 끝 글자 늘리기
    while(answer.size()<=2) answer+=answer[answer.size()-1];

    return answer;
}
```
