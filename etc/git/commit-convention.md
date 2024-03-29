# Commit Convention

### 커밋 메시지 규칙

* 문장의 끝에 `.` 를 붙이지 말기
* 이슈 번호를 커밋 메시지에 붙여 연관된 이슈로 바로 연결되도록 할 수 있다.
* 형식
  * `[타입]: [내용] [이슈 번호]`
  * 현재 직장에서는 \[타입]: \[내용] 형태로 커밋 메시지를 작성한다. 타입은 대문자로, 내용은 소문자로 작성하고 있다. 커밋 메시지는 소속된 곳에 따라 맞춰가면 될 것 같다.
  * 개인적으로는 전부 소문자로 쓰는게 편하긴 한데, 문장 첫 글자를 대문자로 안쓰면 맞춤법에 맞진 않는다고 한다.
  * ex) `docs: OO메소드 관련 설명 주석 #3`, `ENHANCE: improve add() logic in reservation system [#6]`
* 타입 종류
  * chore : 간단한 수정
  * feature : 새로운 기능 추가
  * fix : 버그 수정
  * refactor : 코드 수정 / 리팩터링
  * enhance: 성능 향상
  * test : 테스트 추가
  * docs : 문서 작성
