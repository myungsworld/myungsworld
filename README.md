## 빗썸 open api 활용 , 자동매매 프로그램(BackEnd)

### 기획

- **빗썸 api 정리 및 리팩토링**
    - [Info](https://github.com/myungsworld/myungsworld/tree/master/api/bithumb/Info) : 거래대기,사용가능한돈,캔들스틱,코인별잔고,코인별지갑주소,코인시세
    - [Transaction](https://github.com/myungsworld/myungsworld/tree/master/api/bithumb/transaction) : 매도예약,매수예약,시장가매도,시장가매수,예약가매수,원화출금
- **요구사항**
    - 코인마다 매수,매도 최소 수량이 다름 (api 쓸때 구분 필요) -> 소수점 리팩토링
    - 초단위의 모니터링 및 10분 단위로 변수 초기화 -> 10분 데이터를 디비에 저장할지는 미정
    - 잔여 코인이 없을시에 대한 에러 핸들링
    - 데이터베이스에 매수,매도 내역 기록
    - 모든 매도 매수에 대한 정보 카카오톡 or 문자 폰으로 전송
    
- **핸들러**
  - 코인별 비동기 큐 생성
    - 폭락방지 
      - 10분마다 현재가 , 1초마다 시장가를 가져옴 -> 
      - 10분 기준으로 시장가와 가격 차이가 -3% 가 나면 가진 코인의 절반을 매도 ->
      - 대기열에 들어와서 위의 알고리즘은 대기
      - 10분을 추가로 더 본 다음 처음 현재가와의 차이가 -5%가 나면 남은 코인의 절반을 또 다시 매도 ->
      - 초기화 시킨 후 다시 비동기 큐로 돌아감
    - 폭등감지덕지 키득키득
      - 짜고난후 기록
  
### 기술스택

- MySQL
- GORM
- Go

### USAGE
