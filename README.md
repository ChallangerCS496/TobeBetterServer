# TobeBetterServerClient - Server API 구성 

Tab1
1. 소속된 그룹들 정보 받아올 때
.../api/recorder/:facebookID (POST)
소속된 그룹에 따른 적절한 기록 UI를 표시하기 위함.
특히 time template의 경우 Wi-Fi list도 받아옴.
Default 값 들은 모두 “-”
첫 로그인 시 users collection에 정보가 없으므로 404 보내기(클라이언트 측에서는 404 받으면 닉네임 설정 팝업 띄우기 해당 팝업에 닉네임 설정 후 확인 누르면 아래 2번 실행)
만약 회원가입은 돼있는데 가입된 그룹이 없는 경우엔 status 400으로 보냄.(nickname은 같이 들어감){nickname: ‘testnickname’}
이 경우엔 ‘가입된 그룹이 없습니다’라는 Textview가 가운데 뜨게 하면 될듯.
요청 방식
: /api/recorder/:facebookID로 보내고 body에는 무의미한 문자열 넣음

각 그룹별 가장 최근 기록 정보가 전송됨.
[
    {
        "index": 1,
        "nickname": “testnickname”,
        "groupID": "testgroup2",
        "template": "time",
        "goal": 100,
        "period_unit": 7,
        "history": 0,
        "unit": "3600000",
        "start": null,
        "time": 0,
        "Wifi": [
            "CS496-3",
            "Welcome_KAIST"
        ],
        "count": 0,
        "date": null
    },
    {
        "index": 1,
        "nickname": “testnickname”,
        "groupID": "testgroup",
        "template": "count",
        "goal": 100,
        "period_unit": 7,
        "history": 0, -----------period_unit기준 누적량(이 경우 7일동안의 누적량)
        "unit": "3600000",
        "start": null,
        "time": 0,
        "Wifi": [],
        "count": 0, -----------당일 누적량
        "date": 2020/07/23
    }
]
 

 
index는 몇주차인지 또는 몇일차인지 구분해주는 데이터
template이 time이어도 count정보가 오긴 함. 걍 안쓰면 됨.
period_unit 은 Number type으로 바뀜. 일: 1 주:7 월:30
count든 time이든 index가 같은 가장 최근의 document정보를 받아옴.
count template의 경우 만약 date 정보가 당일과 다르면 하루의 누적 횟수를 0으로 settext해주면 됨.
같은 날의 document를 받은 경우 count는 이전 입력한 갯수들의 합.
횟수 입력창에  넣은 수치만큼 count에 누적됨.



2. 첫로그인 시(닉네임 등록)
.../api/register/:facebookID/:name/:nickname (POST)
[요청] /api/register/ 나머지는 다 body로 보냄:facebookID/:name/:nickname
닉네임은 앱 내에서 고유한 정보로 멤버 초대시에만 이름과 함께 사용.


3. template이 time인 recorder에서 시작 또는 종료버튼 눌렸을 때
.../api/recorder/start/:facebookID/:groupID (POST)
body : {starttime:”2020/07/23/15:30:25”}
.../api/recorder/end/:facebookID/:groupID/(POST) (time은 누적시간)
body : {starttime: “2020/07/23/15:30:25”, endtime:”2020/07/24/02:10:25”, time :”38400000”} (10시간 40분을 ms로 나타낸 수. 안드로이드에 method 있음)
좀 더 정확히는 start나 end 버튼 눌렀을 때 나오는 pop up에서 ‘네’를 눌렀을 때
start인 경우  starttime에 2020-07-21 18:30:21
end인 경우 endtime에 2020-07-22 04:10:55,  time에 1542000 ( 밀리초단위)

4. template이 count인 recorder에서 기록 버튼 눌렀을 때
.../api/recorder/count/:facebookID/:groupID/:daycount (POST)
body : {count:”20”}
여기서 :daycount는 당일 누적 갯수. 즉 오늘 1시에 턱걸이 20개를 하고나면 다음 번 접속 시엔 20으로 표시됨. 이후 10개를 더하고 나서 10을 입력하면 30으로 표시됨.
*daycount는 그날의 누적 횟수



Tab2
0. Group 생성할 때
초대 가능한 멤버 목록 띄울 때
.../api/create/users/:facebookID(POST)
[{name:”김형규”,nickname:”뿡뿡”},{name:”김경연”,nickname:”삥삥”}]
위 형식으로 갈 거임

입력 받은 내용 서버에 전송할 때
.../api/create/:groupID/:template (POST)
body
{members:[“본인 nick”,“nick1”,”nick2”], period_start:”2020/07/23”, period_end:”2020/08/06”, goal:”100”, unit:”1000”, period_unit:”7”, Wifi:[“CS496-3”,”Welcome_KAIST”], goal_unit:”JBG”}
template이 time인 경우 unit 은 ms 기준! 초:1000, 분:60000,시간:3600000
이걸 받으면 Server에선 DB에 
groupinfos collection : 그룹이름(groupID), 멤버들 목록 저장
각 멤버들 collection에는 그룹 정보들 담은 document추가.
1. ‘Tab2’에서 소속된 그룹 리스트 정보 받아올 때
.../api/group/:facebookID (GET)
Tab2에서 그룹 리스트를 표시하기 위함.
소속된 그룹들(Document) 정보 받아와서 몇몇 정보만 Recycler view 로 출력
소속된 그룹이 없으면 400을 보냄 이 경우 소속된 그룹이 없다는 Textview를 visible하게 만들기
[ { members: [ 'testnickname' ],
    Wifi: [],
    _id: 5f1c81d7f5cc400fa45cad7f,
    groupID: 'testgroup',
    template: 'count',
    unit: '3600000',
    goal: 100,
    period_start: 2020-07-23T00:00:00.000Z,
    period_end: 2020-08-06T00:00:00.000Z,
    period_unit: 7,
    __v: 0 },
  { members: [ 'testnickname' ],
    Wifi: [ 'CS496-3', 'Welcome_KAIST' ],
    _id: 5f1c81fff5cc400fa45cad81,
    groupID: 'testgroup2',
    template: 'time',
    unit: '3600000',
    goal: 100,
    period_start: 2020-07-23T00:00:00.000Z,
    period_end: 2020-08-06T00:00:00.000Z,
    period_unit: 7,
    __v: 0 } ]

정확히 뭐 보낼지 몰라서 일단 그룹 정보 다 보냈는데 그냥 그룹 리스트 보여줄 때 필요없는 정보는 안쓰면 됨.


2. ‘Tab 2 Group Click activity’ 에서 개인의 성취 데이터를 받아올 때
.../api/group/:facebookID/:groupID (GET)
각 index별(1주차, 2주차 또는 1개월차 2개월차 등등) 마지막 문서의 history, goal
[
    {
        "_id": 1,
        "published_date": "2020-07-28T19:02:47.319Z",
        "goal": 100,
        "history": 30,
        "unit": "3600000",
        "period_unit": 7
    },
    {
        "_id": 2,
        "published_date": "2020-07-31T19:02:47.319Z",
        "goal": 100,
        "history": 50,
        "unit": "3600000",
        "period_unit": 7
    }
]

 
3. ‘Tab 2 Group Click activity’ 에서 그룹 멤버들의 랭킹 데이터 받아올 때
.../api/group/ranking/:groupID/:standard/:index (GET)
groupid는 tab의 그룹 카드 안의 Textview로 부터 getText로 얻으면 될듯
:standard - 일(1)별 주(7)별 월(30)별 랭킹 요청.
:index - 각각 몇일차, 몇주차, 몇월차 정보를 원하는지도 보내줘야 함.
시작일이 7/25일인 그룹에서 1일차 정보를 요청하면 7/25일에 기록된 그룹 멤버들의 history정보를 정렬해서 리스트로 보내줌. 3주차 정보를 요청하면 8/8~8/14 동안의 멤버들의 history정보를 합산한 후 리스트로 보내줌.
원하는 기간에 데이터 없으면 404 보냄.
[
    {
        "name": "testname",
        "nickname": "testnickname",
        "total": 70
    },
    {
        "name": "testname2",
        "nickname": "testnickname2",
        "total": 35
    }
]


4. ‘Tab 2 Group Click activity’ 에서 나타내 줄 그룹 정보 받아올 때
.../api/groupinfo/:groupID (GET)
1번에서 받은 그룹 정보를 저장해놨다가 intent로 넘겨줄 수도 있긴하지만 따로 저장해놓는 것도 일이니까 일단 구현해놨음. 
{
    "members": [
        "testnickname"
    ],
    "Wifi": [],
    "_id": "5f1c81d7f5cc400fa45cad7f",
    "groupID": "testgroup",
    "template": "count",
    "unit": "3600000",
    "goal": 100,
    "period_start": "2020-07-23T00:00:00.000Z",
    "period_end": "2020-08-06T00:00:00.000Z",
    "period_unit": 7,
    "__v": 0
}
