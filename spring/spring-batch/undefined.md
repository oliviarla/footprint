---
description: 배치 실행 과정에서 일어나는 상태를 DB에 저장하기 위한 방법을 알아본다.
---

# 메타데이터 저장

## 메타데이터 테이블

* 다음 사항들을 디스크에 저장해두기 위한 테이블 구성이 필요하다.
  * 이전에 실행한 Job 목록
  * 최근 실패한 Batch Parameter, 성공한 Job 목록
  * 다시 실행한다면 어디서 부터 시작하면 될지
  * 어떤 Job에 어떤 Step들이 있었고, Step들 중 성공한 Step과 실패한 Step들은 어떤것들이 있는지
* Job Instance
  *
* Job Execution
  *

## ExecutionContext

