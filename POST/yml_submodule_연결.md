# `yml`파일 `Submodule`로 분리

`yml` 파일에는 중요한 정보가 들어있을 수 있기 때문에 `github`에 올리면 좋지 않다.
이때, `submodule`을 사용하면 `clone`하거나 업데이트 시에 편하게 `yml` 파일을 관리할 수 있다.

1. 프로젝트의 `.gitignore` 에 `*.yml` 추가
2. `github`에 `private repository`를 만들고 `yml` 파일을 넣어준다.
3. 프로젝트 레포에 `submodule`를 추가해준다.
```terminal
git submodule add {private repository address}
```

이렇게 되면 `submodule` 연결은 끝

추후 `gradle` 설정을 해주면 `build`할때 자동으로 `yml`파일을 `submodule` 폴더에서 복사해올 수 있다.

```gradle
task copyPrivate(type: Copy) {
copy {
		from '../../SsaMuSoConfig/main'
		include "*.yml"
		into 'src/main/resources'
	}
copy {
		from '../../SsaMuSoConfig/test'
		include "*.yml"
		into 'src/test/resources'
	}
}
```