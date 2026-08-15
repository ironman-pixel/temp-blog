---
date: 2025-09-11
tags:
  - kanban
  - project
---
[blog: Celery](https://tiaz.dev/Celery/1)

## 작업 큐 (Task Queue) 를 이용한 분산 처리

### 문제
Request를 처리하는데 오래 걸리는 작업이 있다면 
**화면에서는 Response가 올때까지 기다려야 함**

### 해결
Request를 일단 Task Queue에 넣고, **Response를 먼저 보내고**
실제 **Request에 대한 처리는 뒤에서 수행**

### 작업 큐
시간이 오래 걸리거나 Resource를 많이 사용하는 작업을 
Main Application 흐름에서 분리하여 
**Background에서 처리할 수 있게 해주는 시스템**

---

## Celery 란

Python에서 Task Queue를 사용하기 위해 쓰는것
주로 Web Application 에서 Background 작업을 처리하는데 사용되며 
분산 시스템을 구축하는데 매우 유용


## Celery 구성

### Clinet (Producer)
작업을 생성하고 Brocker 에게 전송
일반적으로 Web Application이나 Script 가 Client 가 됨
Celery를 사용하여 작업을 정의하고 실행을 요청함

### Brocker (Task Queue / Message Queue)
Client와 Worker 사이에서 메시지를 중개
작업 메시지를 저장하고 Worker에게 전달하는 역할

### Worker (Consumer)
Broker로 부터 작업을 받아 실제로 실행하는 Proccess
여러 Worker를 동시에 실행하여 작업을 병렬로 처리 가능
작업 결과를 백엔드에 저장 가능


