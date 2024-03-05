# 내부 구조

## 객체

* Git 은 `객체`라는 단위로 버전 관리를 위한 데이터를 관리한다.
* 객체에는 종류나 크기 등의 메타데이터와 실제 내용이 기록된다.
* 객체의 종류로는 아래와 같이 네 종류가 있다.

| name   | description                |
| ------ | -------------------------- |
| blob   | 파일 정보가 들어 있는 객체            |
| tree   | 디렉토리 정보가 들어 있는 객체          |
| commit | 커밋 정보가 들어 있는 객체            |
| tag    | annotated tag 정보가 들어 있는 객체 |

### blob 객체

* 실제 파일의 내용 자체를 기록하고 있는 백업 객체이다.

### tree 객체

* 디렉토리 정보를 보관하는 객체
* 디렉토리에 존재하는 파일과 해당 버전의 blob key(해시값)을 저장한다.
* 각 디렉토리마다 tree 객체가 존재하게 된다.
* 루트 디렉토리에 존재하는 트리 객체로는 리포지토리의 모든 파일에 접근할 수 있다.

### commit 객체

* 리포지토리에 존재하는 루트 디렉토리의 tree 객체의 해시 값(Key), 상위 commit 해시 값, committer와 author 의 타임 스탬프/이름/이메일 주소, 커밋 메시지가 포함된다.
* 이 중 하나라도 변경되면 다른 commit 해시가 되기 때문에 commit은 고유하게 관리되며 변조에 강하다.
* `git add` 시에는 index에 파일을 등록하고, blob 객체를 생성하게 된다.
* `git commit` 시에는 index에 tree 객체를 생성하고 commit 객체를 생성한다. 이후 HEAD를 새로운 commit 해시로 갱신한다.
  * index에서 변경이 발생한 부분만 새로운 blob/tree 객체로 갱신하고, 변경이 발생하지 않은 부분은 기존 참조를 그대로 사용한다.

### tag 객체

* `git tag` 로 light-weight tag 또는 annotated tag 를 생성할 수 있다.
  * light-weight tag는 가장 단순한 태그의 형태로 `git tag {tag name}` 으로 생성할 수 있다.
  * annotated tag는 추가적으로 데이터가 붙은 태그로, `git tag -a {tag name} -m "message"` 로 생성할 수 있다.
* annotated tag를 생성할 때에만 tag 객체가 저장된다.

## index

* 현재 참조하는 버전의 모든 파일에 대한 참조를 갖는다.
* `.git/index` 경로에 기록된다.
* `git ls-files --stage` 를 통해 인덱스의 내용을 확인할 수 있다.

## refs

* 특정 커밋을 가리키는 포인터의 역할을 한다.
* commit hash의 별칭이라고도 할 수 있다.
* tag의 `light-weight tag`, `annotated tag`, branch의 `HEAD`가 refs에 해당한다.

#### 출처

[https://www.mimul.com/blog/git-internal/](https://www.mimul.com/blog/git-internal/)
