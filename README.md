# 고급파이썬 과제 1 - 웹사이트 크롤링과 저장·시각화

## 질문
**5점 만점(별 5개) 책의 비율은 얼마나 되는가?**

## 대상 사이트 및 선정 이유
- 사이트: https://books.toscrape.com
- 선정 이유: 웹 크롤링 연습을 위해 공식적으로 운영되는 목(mock) 온라인 서점으로,
  실제 판매 사이트가 아니라 스크래핑 학습용으로 만들어졌다. 로그인, 결제 등
  민감한 기능이 없고 페이지 구조가 단순해 수업에서 배운 requests + BeautifulSoup
  조합으로 접근하기에 적합하다고 판단했다.
- 이용 가능 근거(robots.txt): https://books.toscrape.com/robots.txt 를 확인한 결과
  수집을 막는 Disallow 규칙이 없고, 사이트 자체가 "크롤링 연습용"임을 명시하고 있다.

## 설치 및 실행 방법
```bash
pip install -r requirements.txt
python crawler.py
```
실행하면 `data/result.csv`와 `logs/crawl.log`가 자동으로 생성된다.

## 수집 항목
| 컬럼 | 설명 |
|---|---|
| title | 책 제목 |
| price_gbp | 가격 (파운드, 숫자형으로 변환) |
| rating | 평점 (One ~ Five) |
| availability | 재고 여부 문구 |
| source_page_url | 수집한 목록 페이지 URL |
| crawled_at_utc | 수집 시각(UTC) |

- 분석용 속성: price_gbp, rating, availability (3개 이상 충족)

## 선택자 근거
- 책 1권 단위: `article.product_pod`
- 제목: `h3 a` 태그의 `title` 속성 — 화면 텍스트는 CSS로 말줄임 처리되어 잘려 보이지만,
  `title` 속성에는 전체 제목이 그대로 들어있어 이를 사용했다.
- 가격: `.price_color`
- 평점: `p.star-rating` 태그의 두 번째 class 값 (예: `class="star-rating Three"` → "Three")
- 재고: `.instock.availability`
- 다음 페이지: `li.next a` — 마지막 페이지에는 이 태그가 없어 반복 종료 조건으로 사용

## 수집 과정 요약
- 시작 페이지: `catalogue/page-1.html`
- 최대 5개 목록 페이지(페이지당 20권)를 순회하며 총 3페이지 이상, 100건 내외 수집
- 요청 간 1초 지연(`sleep(1.0)`), timeout(5초 연결/20초 응답) 설정
- 403/429 응답 시 즉시 크롤링 중단, 그 외 요청 오류는 로그를 남기고 해당 페이지만 건너뜀

## 데이터 정제 기준
- 중복 행: `drop_duplicates()`로 완전히 동일한 행 제거
- 결측치: title / price_gbp / rating 중 하나라도 비어 있으면 해당 행 제거
  (셀렉터가 예상과 다르게 매칭되지 않은 경우로 판단)
- 잘못된 행: 파싱 중 예외(AttributeError/KeyError 등)가 발생한 항목은
  수집 단계에서부터 제외하고 로그(`logs/crawl.log`)에 사유를 남김

## 결과 및 해석
- 5점 만점 책 비율: **19%**
- 평점 분포에서 5점은 소수(약 5권 중 1권)에 그쳤고, 나머지는 1~4점에 고르게 분산됨
- (평점별 평균 가격 등 추가 분석 결과는 결과보고서 PDF에서 확장)

## 한계
- 수집 범위를 5페이지(약 100건)로 제한해 전체 1,000권 중 일부만 반영함
- 평점은 별 개수 텍스트로만 표현되어 있어 세부 리뷰 점수(예: 4.3점)는 알 수 없음

## AI 활용 내역
- Claude(Anthropic)를 활용해 requests/BeautifulSoup 기반 크롤링 코드 초안 작성,
  예외 처리·로깅·자료형 변환 로직 보완, 선택자 확인(실제 웹페이지 구조 조사 포함)에 도움을 받음
- 최종 코드 실행과 결과 확인은 직접 수행함
