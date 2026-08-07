
| 명령어                                        | 설명                        |
| ------------------------------------------ | ------------------------- |
| `ssh-keygen -t ed25519 -C <email address>` | 유니크 키를 이용하여 ssh 키 생성      |
| `ssh -T git@github.com`                    | github.com으로 보안 인증 테스트 진행 |


---

## 개인 학습 사항 정리

> 기존 사용하는 ssh 키 설정이 있고 별도의 ssh 키를 생성하여 과리하여야 할 경우

ssh 키를 지정하여 생성

```
kant
kant.pub
```

ssh config에 host 정보 추가

```
Host github.com-kant
	HostName github.com
	User git
	IdentityFile ~/.ssh/kant
```

ssh 인증 테스트 진행

```
ssh -T git@github.com-kant
```

이후 github에서 ssh로 clone 시 다음과 같이 사용한다.

```
git@github.com:Sangdon05/ax_study.git
-> 아래와 수정하여 같이 사용
git@github.com-kant:Sangdon05/ax_study.git
```
