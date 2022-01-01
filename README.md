# Rock Paper Scissors & Tic-Tac-Toe - Client and Server model

여러가지 유의사항들
- 기본적으로 모든 전송은 byte[] 타입이다.
- 유저의 갑작스러운 종료를 대비한 코드는 존재하지만, 클라이언트 측에만 대안을 주고, 서버에는 대안이 없으므로 반만 구현된 느낌이...
- 서버 소켓은 9090 번이다.
- 클라이언트는 최대 10명까지 접속할 수 있다. (늘릴 수 있는데, 일단은 10명으로 해두었음)
- ttt게임에서 두 유저간의 동기화는 서버에서 제공하는 610 프로토콜을 사용해 클라이언트에서 구현해야한다.
- 2인 방에서 방장이 나가면 방이 폭파된다.

==============================================
Protocol

서버 >> 클라이언트

로그인 성공: 100
+ byte[] 타입이다.
+ 참고로, 프로토콜: 700과는 별개이므로, 로그인 처음 하면 100하고 700이 순서대로 갈 것이다.


로그인 실패: 105
+ byte[] 타입이다.


회원가입 성공: 200
+ byte[] 타입이다.
+ 회원가입 성공하면, 로그인 창으로 이동해서 로그인을 시도한다.


회원가입 실패_정보가 이미 존재: 205
+ byte[] 타입이다.


회원가입 실패_ID 중복입력 잘못됨: 210
+ byte[] 타입이다.


초대 성공: 300
+ byte[] 타입이다.


초대 실패: 305
+ byte[] 타입이다.


초대 메시지(내가 받는 상황): 400 (나를 초대한 사람의 닉네임)
+ byte[] 타입이다.


게임에서 자신의 선택이 반영되었음: 600
+ byte[] 타입이다.
ex) 틱택토에서 제대로 수를 둔 경우.


게임에서 자신의 선택이 반영되지 않았음: 605
ex) 틱택토에서 이미 둔 자리에 수를 또 둔 경우.
+ byte[] 타입이다.
++ 가위바위보는 따로 위와 같은(600, 605) 프로토콜이 없다. (무조건 성공하기 때문)


게임에서 상대방의 선택이 반영되었음: 610
+ byte[] 타입이다.
>> 클라이언트에서 유저들의 동기화에 사용할 수 있다. (참고로 ttt에서만 쓰인다.)


게임의 초기화가 완료되었음: 650
+ byte[] 타입이다.
>> 게임 시작하기 전에 호출하는 초기화 함수의 결과


게임 입장(메인 대기방 입장): 700 1
+ byte[] 타입이다.
+ roomID는 채팅이나 유저 정보 제공에 사용된다.
ex) 700 1 >> Main room은 무조건 roomID가 1이므로 이는 고정된다.


게임 입장(2인 대기방 입장): 705 (roomID)
+ byte[] 타입이다.
+ roomID는 채팅이나 유저 정보 제공에 사용된다.
ex) 705 10 >> roomID가 10(int type)인 경우


게임 퇴장(메인 대기방 퇴장): 710
+ byte[] 타입이다.


게임 퇴장(2인 대기방 퇴장): 715
+ byte[] 타입이다.


게임에서 승리한 경우: 800
+ byte[] 타입이다.


게임에서 패배한 경우: 801
+ byte[] 타입이다.


게임에서 무승부인 경우: 802
++ 위의 800, 801, 802 프로토콜은 가위바위보와 틱택토의 공통 프로토콜이다.
+ byte[] 타입이다.


게임 전적이 DB에 반영된 경우: 810
+ byte[] 타입이다.


채팅: 850 (채팅 보내는 유저의 닉네임) (내용)
+ 출력 데이터는 byte[] 타입이다.
ex) 800 mynickname Hi!!! I'm new user mynickname!


유저 정보 출력(메인 대기방): 900 (닉네임) (win_rcp) (lose_rcp) (total_game_rcp) (win_T3) (lose_T3) (total_game_T3)
+ 출력 데이터는 byte[] 타입이다. 전체 데이터는 String >> byte[]로 변환한 것이므로 String으로 분할하여 그대로 사용하거나, String을 int로 변환하여 사용하면 된다.
ex) 900 mynickname 100 10 120 120 110 300


방장이 된 경우: 1000
+ 출력 데이터는 byte[] 타입이다.


방의 이름이 변경된 경우: 1005 (새로운 방 이름)
+ 출력 데이터는 byte[] 타입이다.
ex) 1005 New room 140


방의 이름을 반환하는 경우: 1010 (기존의 방 이름)
+ 출력 데이터는 byte[] 타입이다.
ex) 1010 Room_14


잘못된 프로토콜의 사용: 2000
=====================================================

클라이언트 >> 서버

로그인: 100 (ID) (PW)
+ byte[] 타입이다.


회원가입: 105 (이름) (닉네임) (ID) (ID_중복) (PW) (이메일)
+ byte[] 타입이다.


채팅(메인, 2인방 공통): 200 (현재 방 ID) (보낼 데이터)
+ byte[] 타입이다.
+ 보낼 데이터는 모든 프로토콜을 합쳐서 1500byte까지이다.


유저 정보 제공받기(메인, 2인방 공통): 300 (현재 방 ID) 
+ byte[] 타입이다.
+ 만약 roomID가 1이라면(mainRoom에 있다면) 모든 유저의 정보를 제공받을 수 있다. >> 필요할때 활용해보자.


2인 대기방에서 메인 대기방으로 이동: 400 (현재 방 ID)
+ byte[] 타입이다.
+ 만약 roomID가 1이라면(이미 mainRoom에 있다면) 아무런 동작을 하지 않는다.
+ 참고로, 만약 사용한 사람이 owner일 경우, 나머지 1명은 자동으로 나가진다.////


메인 대기방에서 누군가를 초대할때: 500 (내 닉네임) (초대할 대상의 닉네임)
+ byte[] 타입이다.
+ 초대한 사람이 owner이다.


초대에 답할때: 505 (flag) (나를 초대한 사람의 닉네임) (내 닉네임)
+ byte[] 타입이다.
+ flag: 0 >> 수락, flag: 1 >> 거절


현재 방의 이름을 원할때: 600 (현재 방 ID)
+ byte[] 타입이다.
+ 메인 대기방에서도 사용 가능하다.


게임방의 이름을 변경할 때: 605 (현재 방 ID) (새 이름)
+ byte[] 타입이다.
+ 자신이 owner가 아니라면 아무런 반응을 하지 않는다.
+ 메인 대기방에서도 사용 가능하긴 한데, 어차피 owner가 더미 유저라서 의미 없다.


게임을 시작할때(초기화 함수): 700 (현재 방 ID) (내 닉네임) (상대 닉네임)
+ byte[] 타입이다.
+ 중복 처리되기는 하는데, 귀찮으니 그냥 냅두었다.


T3 게임을 진행할때: 701 (현재 방 ID) (내 닉네임) (상대 닉네임) (x 좌표) (y 좌표) 
+ byte[] 타입이다.
+ x행 y열에 자신의 표식을 둘 수 있다.
+ x, y는 0 ~ 2의 숫자이다.


RCP 게임을 진행할때: 702 (현재 방 ID) (내 닉네임) (상대 닉네임) (나의 수)
+ byte[] 타입이다.
+ 나의 수는 가위 바위 보이다.
+ 가위: 0, 바위: 1, 보: 2 로 주면 된다.


메인 대기방에서 나갈때(게임을 종료할때): 808 (현재 방 ID)
+ byte[] 타입이다.
+ 현재 방 ID는 메인 대기방에서만 실행 가능함을 보장하기 위해 필요하다!
