# 로그정의서 작성 매뉴얼
* 로그정의서는 기획자, 개발자, 분석가들이 로그 데이터를 활용할 수 있도록 협업을 위한 도구를 제공합니다
* 로그정의서는 크게 3개의 요소로 이루어져 있습니다
 * 테이블정의서
 * 배치정의서
 * 코드정의서

# 테이블정의서
* 테이블 정의서는 적재되는 로그를 담기위한 형식을 지정합니다
* 아래는 테이블 정의서의 예제 입니다<br />

  | 로그키 | 구분 | 이름 | 타입 | 설명 | 빈값가능 | 암호화 | 자동수집 | 검증룰 | 파라미터 |
  |------|-----|-----|-----|-----|-------|------|--------|------|--------|
  | | header | log_time | string | 로그시간 | true | false | x | datetime | YYYYMMDDHHmmssSSS |
  | | header | log_vision | string | 로그 버전(정의서) | false | false | v | | |
  | | header | hostname | string | 로그전송 서버 | false | false | x | | |
  | v | header | action | string | 로그 ID,액션지정 | false | false | x | | |
  | | header | level | string | 로그 레벨 | false | false | x | code | action |
  | | header | session | string | 브라우저 세션 | true | false | x | | |
  | | header | user | string | 행동 수행자 | true | false | x | | |
  | | header | resource | string | 영향받는 리소스 | true | false | x | | |
  | | header | verb | string | HTTP Method | false | false | x | | |
  | | header | url | string | 요청 URL | true | false | x | | |
  | | header | res_status | int | HTTP 응답 상태코드 | false | false | x | | |
  | | body | res_message | string | human readable message | true | false | x | | |
  | | body | resource_id | string | 요청 리소스 식별자 | true | false | x | | |
  | | body | resource_body | string | 요청 리소스 내용 | true | false | x | | |


* Header & Body 모델
 * Header는 모든 로그에 남는 정보를 기입합니다
 * Body는 로그의 문맥에 따라 달라지는 데이터의 종류를 기술 합니다
 * 로그가 입수되기 시작하면 Body는 변경 가능하지만 Header 정보는 추가, 삭제 및 변경이 불가능합니다
* 테이블 정의서에서 구분이 Header인 한줄의 행은 하이브 테이블 1개의 컬럼에 연결됩니다
* 테이블 정의서에서 구분이 Body인 전체는 하이브테이블 1개의 컬럼에 연결되고 데이터는 JSON의 형식을 갖습니다
* 예제를 보면 Header는 11행, Body는 3행으로 구성되어있기 때문에 하이브 테이블은 총 12개의 컬럼을 갖게 됩니다
* 하이브에서 JSON 컬럼을 조회하기 위해서 [get_json_object UDF](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF#LanguageManualUDF-get_json_object) 를 사용합니다

# 테이블정의서 - 용어설명
* 로그키 - 로그문맥을 구분할 수 있는 유일한값을 갖는 조합을 선택합니다. 일반적으로 page_id, action_id 조합이 사용됩니다
* 구분 - 로든 로그에 남는 정보라면 Header, 로그키의 문맥에 따라 변경이 된다면 Body를 선택합니다
* 이름 - 하이브 테이블의 컬럼이름을 기술합니다. 추후 HQL(Hive Query Language)에 사용됩니다
* 타입 - 하이브 테이블 컬럼의 데이터타입을 기술합니다. 개발자에게 필요한 정보입니다
* 설명 - 행이 나타내는 의미를 기술합니다. 이름으로는 모든것을 구분하기 어렵기 때문에 커뮤니케이션을 위해 충실히 적어주는것이 좋습니다
* 빈값가능 - 해당 값이 반드시 입력되야 하는지 입력되지 않아도 되는지를 설정합니다. 추후 검증단계에서 의도적으로 값을 남기지 않은 것인지 실수한것인지 발견할 수 있습니다
* 암호화 - 암호화가 필요한지 아닌지를 결정합니다. 유저의 개인정보가 포함된 경우라면 반드시 암호화를 해야합니다
* 자동수집 - Rake를 이용할 경우 자동으로 수집되는 필드인지를 나타냅니다. 개발자가 별다른 설정을 하지 않아도 자동으로 해당 값이 입력되어 수집됩니다
* 검증룰 - 원하는 데이터 형식으로 수집이 되고있는지 검증단계에서 확인할 수 있도록 검증규칙을 설정합니다
  * ip - 127.0.0.1 같은 IP 형식인지를 검증합니다
  * url - http://naver.com 같은 웹 URL 형식인지를 검증합니다
  * mdn - 010-1234-5678 같은 MDN 형식인지를 검증합니다
  * resolution - 1920\*1024 같은 화면해상도 형식인지를 검증합니다
  * datetime - 20170119174753292 같은 시간 형식인지를 검증합니다
  * code - 코드정의서에 지정된 코드인지를 검증합니다
  * regex - 정규식을 사용하여 검증합니다. 개발자가 사용합니다
  * function - Javascript 함수를 사용하여 검증합니다. 개발자가 사용합니다

# 배치정의서
* 테이블정의서에서 선틱된 로그키를 이용하여 로그의 문맥에 따라 남게될 Body를 기술합니다<br />

   | page_id | action_id | | | | |
   | --------|-----------|----------|----------|----------|---------|
   | search_product | search_btn_touch | query | sort_by | page_num | per_page |

* /search\_product 페이지의 search\_btn\_touch 액션을 했을때 검색어, 정렬옵션, 페이지번호, 한페이지당 보여지는 아이템수가 필요하다는 것을 알 수 있습니다

# 코드정의서
* key, value, description 으로 정의합니다
* 주로 배치정의서에서 page\_id, action\_id 의 내용이 정상적으로 입력되었는지 검증하기위해 사용됩니다<br />

  key | value | description
  -----|-------|-------------
  page_id | /search_product | 상품검색 페이지
  action_id | search_btn_click | 상품 검색 아이콘 클릭

* 예를 들어 위와같이 정의 했을 때 배치정의서는 action_id 에 search_btn_touch 로 정의했지만 코드정의서에는 search_btn_click 으로 정의 되었기 때문에 검증단계에서 오류가 발견됩니다.

