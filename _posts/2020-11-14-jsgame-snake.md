---
layout: post
title: "[Project] 바닐라 자바스크립트 게임 제작 02. 뱀 게임"
date: 2020-11-13
categories: project
photos: /assets/post_images/js_game/snake.png
tags: [project,javascript,html,css,game]
description: "바닐라 자바스크립트 게임 제작 프로젝트 - 뱀 게임을 만들어보자"
---

<br>

![snake](/assets/post_images/js_game/ex_snake.png)

<br>

# 개요

`Dummy` 또는 `Snake` 라고 불리는 뱀 게임은 누구나 할만 한 고전적인 미니게임이다. 필자는 초등학생 시절에 전자사전에 내장되어있던 게임으로 접했었다. 이 역시도 간단하게 `javascript` 로 구현하기 용이했다.

<br>

# 제작

## 그래픽

게임의 전체적인 틀은 `2048` 때와 동일하게 `html` 의 `table` 태그로 구성된다. 뱀과 코인 또한 `table` 의 격자를 칠함으로써 나타낸다.

![1](/assets/post_images/js_game/snake_1.png)

<br>

## 뱀 로직

이 게임에서 뱀은 기본적으로 `앞으로 전진` 한다. 그 도중에 방향키를 통해 `왼쪽`과 `오른쪽` 방향 전환이 그낭하다. 따라서 `비동기 프로그래밍` 기법을 이용할 필요가 있었다. `javascript` 에서는 `setinterval` 을 통해 이를 구현할 수 있다.

그리고 코인을 먹을 시 뱀은 한 칸 늘어난다. 움직이고 있는 뱀이 한 칸 늘어난다는 것이 약간 까다롭게 느껴질 수 있지만 백준 문제를 풀던 도중 [이 문제](https://yjyoon-dev.github.io/boj/2020/10/21/boj-3190/)에서 로직에 관한 힌트를 얻었다. 이는 다음과 같다.

- 먼저 뱀은 몸길이를 늘려 머리를 다음칸에 위치시킨다.
- 만약 이동한 칸에 코인이 있다면, 그 칸에 있던 코인 없어지고 꼬리는 움직이지 않는다.
- 만약 이동한 칸에 사과가 없다면, 몸길이를 줄여서 꼬리가 위치한 칸을 비워준다. 즉, 몸길이는 변하지 않는다.

위 조건에 따르자면 우선 몸의 길이를 한 칸 늘리고, 코인을 먹지 못할 시 꼬리를 잘라내는 것이다. 즉 `FIFO` 개념의 `queue` 자료구조를 떠올릴 수 있다. 이 게임에서도 큐 자료구조를 이용해 뱀을 구현하였다. `javascript` 에서는 별도의 큐 자료구조 대신 `array` 자료구조에 `push` 와 `shift` 를 이용하여 큐와 동일한 기능을 표현할 수 있다.

또한 뱀이 자기 자신과 충돌했을 경우 즉시 게임오버 된다.

![2](/assets/post_images/js_game/snake_2.png)

<br>

## 초기 설정과 조작

초기에는 뱀의 위치를 보드판의 가운데에 위치해주고, `setInterval` 을 실행해 뱀의 움직임을 시작해준다. 뱀의 방향전환은 키보드 입력에 따라 해주되, `좌` `우` 방향전환만 가능하므로 방향을 한 번에 180도 바꿀수는 없으므로 이를 따로 신경써주어야 한다.

![3](/assets/post_images/js_game/snake_3.png)

<br>

# 마무리

결과는 간단한 게임인 만큼 만족스러웠으나, 조작감에 대해 좀 더 고찰하는 기회를 가졌다. 뱀은 `setInterval` 을 통해 일정한 간격으로 움직이고 있으나 방향 전환을 이 타이밍에 정확히 맞추지 않는 이상 즉각 방향을 바꾸는 것 처럼은 보이지 않았다. 그렇다고 방향 전환 시마다 `clearInterval` 을 통해 비동기를 초기화하고 다시 이동을 시키자니 첫 틱에 딜레이가 생겼다. 이는 뱀의 이동 간격을 줄임으로써 어느정도 매끄럽게끔 되었지만 좀 더 고민해볼만한 요소인 것 같다.

전체 코드 및 플레이는 `github` 저장소를 참고해주기 바란다.

[yjyoon-dev/vanillaJSGame-snake](https://github.com/yjyoon-dev/vanilla-javascript-game/tree/master/snake)