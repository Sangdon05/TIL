최소한의 캡슐화 설정

database와 service를 추가하여 데이터 관리와 비지니스 로직 분리 및 DI 구성
- 이로인해 unit test로 기능 구현 및 동작 확인이 수월해짐

Error 정의로 예외 처리를 체계적으로 하려하였으나 불필요한 확장으로 판단되어 ValueError로 통일

