## 빗썸 open api 활용 , 자동매매 프로그램(BackEnd)

### 기획

- **빗썸 api 정리 및 리팩토링**
    - [Info](https://github.com/myungsworld/myungsworld/tree/master/api/bithumb/Info) : 거래대기,사용가능한돈,캔들스틱,코인별잔고,코인별지갑주소,코인시세
    - [Transaction](https://github.com/myungsworld/myungsworld/tree/master/api/bithumb/transaction) : 매도예약,매수예약,시장가매도,시장가매수,예약가매수,원화출금
- **요구사항**
    - 코인마다 매수,매도 최소 수량이 다름 (api 쓸때 구분 필요) -> 소수점 리팩토링
    - 초단위의 모니터링 및 cycle 변수 단위로 변수 초기화 -> 데이터를 디비에 저장할지는 미정
    - 잔여 코인이 없을시에 대한 에러 핸들링
    - 데이터베이스에 매수,매도 내역 기록
    - 모든 매도 매수에 대한 정보 카카오톡 or 문자 폰으로 전송

- **핸들러**
  - 코인별 비동기 큐 생성
    - 폭락방지
      - 무한루프 생성 cycle 초기화 -> 
      - 가진 코인의 금액이 5천원 이상이면 가진 코인의 절반을 매도 -> 
      - 새로운 10분 루프 생성 ,변동률이 -5퍼가 될시 남은 금액의 절반을 다시 매도 후 무한루프 복귀 ->   
      - 10분동안 -5퍼가 되지 않으면 다시 처음의 10분 무한루프로 복귀 ->
      - 데이터베이스에 트랜잭션 기록
    - 폭등감지
      - 무한루프 생성 10분마다 초기화 ->
      - 10분 사이 변동률이 3퍼 이상인 코인 50000원 매수 ->
      - 새로운 20분 루프 생성 , 변동률이 10퍼 이상일 경우 매수 금액의 20퍼 매도 ->
      - 20퍼 이상일 경우 매수 금액의 40퍼 매도 , 30퍼 이상일 경우 나머지의 전액을 매도 ->
      - 위의 변동률에 못미치면서 20분 지나면 다시 첫번째 무한루프로 복귀 ->
      - 매수와 매도는 데이터베이스에 기록
  
### 기술스택

- MySQL
- GORM
- Go

- 문제점 ( around 2021. 09  ~ 10 )
    - 티커별 소수점 정리 
    - 폭등매수후 마이너스가 될때의 핸들링
    - 30퍼 먹었을때 매도 하는거 트랜잭션 체크
    - 마지막 매도 때린다음 3퍼가 다시 오르면 또 매수를 하기 때문에 시간을 둬야함

- 개선된 코드 ( 2021 . 11 . 2 )  
    - 모든 코인 티커 소수점 4자리까지 Default 로 수정
    - 코인별로 세부적인 설정 가능하게 변경
    - ~~폭등매수후 마이너스가 된다고 팔면 그게 투자니?~~
    - 폭등감지후 매수후 마지막 매도할때 에러 버그 픽스
        - 원인 : balance = fmt.Sprintf("%.4f", EA) 를 해올때 소수점 5째 자리수를 반올림 했었기 때문에 총 수량보다 높게 측정되서 팔지 못함
        - 해결 : balance = balance - 0.00005
    - 폭등감지 마지막 매도후 한시간 쉬고 다시 대기열에 들어가게 함
