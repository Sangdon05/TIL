#supabase

### 원인
- Supabase는 보안상 `auth` 스키마와 `public` 스키마 간의 직접적인 외래 키 조인이나 스키마 간 쿼리를 기본적으로 제한합니다.
- 클라이언트(프론트엔드) 코드에서 `.from('users').select('*, auth.users(*)')`와 같은 방식으로 직접 조인하면 권한 문제나 에러가 발생합니다.

아래와 같이 postprogreSQL view를 public으로 생성하여 활용할수있다. (선능 이슈 발생 가능)

```sql
create view user_profiles as
select id, email, created_at
from auth.users;
```

trigger를 통해 새로운 유저, 업데이트 등이 일어났을때 업데이트하는 방식으로 전환 필요