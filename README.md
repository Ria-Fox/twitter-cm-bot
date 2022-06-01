# twitter-cm-bot

해당 레포는 파이썬 및 개발 경험이 없는 사용자를 대상으로 작성되었습니다.

실행을 위한 가장 기본적인 설명을 포함하고 있으며, 사용자가 겪을 오류에 대한 대처 방법은 각 항목에 기술된 Notion 문서에서 설명할 예정입니다.

> (Modified by Riafox)

- 원본 레포 : [OtterNuts/twitter-cm-bot](https://github.com/OtterNuts/twitter-cm-bot)
- 원 기술 블로그 : [otternut.log](https://velog.io/@otternut/%ED%8A%B8%EC%9C%84%ED%84%B0%EB%A1%9C-trpg%EA%B2%8C%EC%9E%84%EC%9A%A9-%EB%B4%87%EC%9D%84-%EB%A7%8C%EB%93%A4%EC%96%B4%EB%B3%B4%EC%9E%90-0-%EB%93%A4%EC%96%B4%EA%B0%80%EA%B8%B0)


## Index

0. [사용될 프로그램 및 툴](##사용될-프로그램-및-툴)
1. [환경 구성(Python 설치 및 가상 환경 구성)](##Python-및-가상-환경-구성)
2. [기본 패키지 설치(Poetry 사용법 및 의존성 관리)](##기본-패키지-설치)
3. [트위터 인증용 키/토큰 생성과 추가](##트위터-인증용-키/토큰-생성과-추가)
4. [키워드 등록 및 새로운 기능 추가](##키워드-등록-및-새로운-기능-추가)
5. [구글 스프레드 시트 관리](##구글-스프레드-시트-관리)
- [사용된 이미지에 관해](##사용된-이미지에-관해)

## 사용될 프로그램 및 툴

해당 ReadMe는 다음 프로그램 및 툴을 사용한다고 가정하고 작성되었습니다. 각 항목은 해당 프로그램의 간단한 사용법 링크로 연결됩니다.

- [PowerShell (기본 Windows CLI Tool)](https://appia.tistory.com/409)
- [(선택) Visual Studio Code (코드 변경을 위한 IDE)](https://m.blog.naver.com/cjsksk3113/222360517100)

## Python 및 가상 환경 구성

해당 자동봇의 구동을 위해서는 Python의 설치가 필요합니다. 아래 항목에서는 Python 설치 방법 및 자동봇을 위한 가상 환경 구성에 대해 설명합니다.

### Python 설치

본 프로젝트에서 사용될 Python 버전은 3.8.10입니다. 3.8 버전 중 Windows 인스톨러를 제공하는 가장 마지막 안정적인 버전으로, [파이썬 공식 홈페이지](https://www.python.org/downloads/release/python-3810/)에서 설치하실 수 있습니다. (Files의 가장 아래 Windows installer (64-bit))

Python3.8 설치가 완료된 후, PowerShell에서 ```python```을 입력하였을 때 정상적으로 실행되어야 합니다. 만약 오류가 발생할 경우(Python이 실행 가능한 파일 또는 프로그램이 아니라는 오류) 다음 노션 문서를 참조하여 환경 변수를 추가해 주세요.

[Python 실행 오류 트러블슈팅](https://www.notion.so/Python-f396637ef5b04e909720a4d6e6bfed48)

### 가상 환경 구성

작성 예정

## 기본 패키지 설치

본 프로젝트는 poetry 로 라이브러리 의존성 관리를 하고 있습니다. 다음의 command를 입력 후 사용해주시기 바랍니다.

```
> curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python
> poetry install
```

## 트위터 인증용 키/토큰 생성과 추가

트위터와 구글 스프레드 시트 api를 사용하고 있습니다.

### 트위터 인증
src/auth/twitter.py

```
self.consumer_key = "여기에"
self.consumer_secret = "키와 시크릿,"
self.access_token = "토큰과 시크릿을"
self.access_token_secret = "입력하세요"

```

### 구글 인증
auth 폴더에 구글드라이브의 인증 json 파일을 넣은 후 아래 코드에 경로를 입력하세요.
src/auth/google_drive.py

```
self.keyfile = "src/auth" + "json 파일명"
```

## 키워드 등록 및 새로운 기능 추가
src/main
```
activity_list = ["오늘의운세", "[사냥]", "[요리]", "[낚시]", "[장비뽑기]", "[로또뽑기]", "[일괄판매]"]

```
에 새로운 키워드를 추가한 후 src/tweetBot/generate_reply 함수의 아래 부분에 elif 구문으로 기능을 추가해서 사용하세요

```
        elif task_name == "[일괄판매]":
            print("일괄판매 시작")
            num_c_equip = sheet_data["플레이어"][user_id].C급장비개수
            num_b_equip = sheet_data["플레이어"][user_id].B급장비개수
            if int(num_b_equip) + int(num_c_equip) == 0:
                reply_comment = "장비가 없습니다. 인벤토리를 확인해주세요."
            else:
                sell_total = 5000 * int(num_b_equip) + 1000 * int(num_c_equip)
                sheet_data["플레이어"][user_id].골드 += sell_total
                sheet_data["플레이어"][user_id].C급장비개수 = 0
                sheet_data["플레이어"][user_id].B급장비개수 = 0
                self.google_api.update_user_data("플레이어", "test", sheet_data["플레이어"])
                reply_comment = "@%s" % user_id + f"[장비판매]\nB급장비개수: {num_b_equip}\nC급장비개수: {num_c_equip}\n총가격: {str(sell_total)}"

        ##### 새로운 기능을 추가하길 바랄 경우 여기에 elif 구문으로 코드를 추가하세요 #####

        else:
            reply_comment = "@%s" % user_id + "봇 오류입니다. 캡쳐와 함께 총괄계에 문의 부탁드립니다."

        return [{"reply_image": reply_image, "reply_comment": reply_comment}]
```

결과값은 reply_comment에 할당하고, 이미지를 추가하고 싶을 경우 reply_image에 이미지파일명을 확장자와 함께 기입하세요.

```
reply_comment = "@%s" % user_id + "봇이 명령어를 읽었습니다."
reply_image = "photo.png"
```

## 구글 스프레드 시트 관리
예시용 구글 스프레드 시트: https://docs.google.com/spreadsheets/d/15HMKanZCVymE3XsnkhjgYZ_ddQTmfZRTrclwD3S9Cts/edit#gid=1956526250

사용할 시트에 구글 api 계정을 추가한 후 src/dataProcessors/from_google_spread_sheet.py 최상단 상수 SHEET_NAME에 sheet이름을 추가하세요
예시)

```
SHEET_NAME = "bot-data-example"

```

새로운 데이터 시트를 추가 후 불러올 경우 src/dataProcessors/from_google_spread_sheet.py/DataProcessingService 에 아래의 형식을 지켜 새로운 데이터 시트를 추가하세요.

```
example_raw_data = google_api.get_all_data_from_sheet(SHEET_NAME, "시트이름")
```
시트 데이터에 맞춰 src/dataProcessors/models/items에 dataclass를 추가하고 데이터를 가공할 함수를 생성하세요.
raw_data를 가공하여 sheet_data에 넘기세요.

## 사용된 이미지에 관해
오픈소스 이미지들을 넣어두었습니다. 가챠 기능을 그대로 이용하실 경우 normal.png, special.png의 이미지는 커뮤 분위기에 맞는 이미지로 교체하시는 것을 추천드립니다.
이외에 이미지를 넣을 경우 구글 스프레드시트의 이미지란에 이미지이름을 적어두시고 src/tweetBot/images 에 이미지를 첨부하면 됩니다.