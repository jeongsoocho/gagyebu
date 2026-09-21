# 가계부 🐷

브라우저에서 바로 쓰는 단일 파일 가계부입니다. 내역은 **Supabase** 데이터베이스에 저장되어, 폰·노트북 어디서 열어도 똑같이 보입니다. 로그인은 없습니다.

## 기능

- **기기 간 동기화** — 내역이 Supabase에 저장되고, 탭으로 돌아올 때마다 최신 상태를 다시 읽어옵니다. 화면 아래 배지로 `동기화됨 / 저장 중… / 동기화 실패` 를 확인할 수 있어요
- **달력 뷰** — 한 달을 한눈에. 날짜별 수입/지출 금액이 칸 안에 표시되고, 날짜를 누르면 그날 내역만 따로 보여줍니다 (더블클릭하면 그 날짜로 바로 입력창이 열려요)
- **월 이동** — `‹ ›` 버튼 또는 키보드 좌우 화살표, `오늘` 버튼으로 이동
- **수입 / 지출 입력** — 금액·분류·날짜·메모, 이모지 분류 칩과 +1천/+1만 같은 빠른 금액 버튼
- **내역 수정 / 삭제** — 내역을 누르면 수정창, `✕`로 바로 삭제
- **10만원 이상 지출 경고 ⚠️** — 금액 입력 중 실시간 경고 배너, 저장할 때 "정말 기록할까요?" 확인창, 저장 후 경고 토스트. 달력 칸과 내역에는 ⚠️ 표시, 상단에는 이번 달 큰 지출 요약 배너
- **월별 집계** — 수입 · 지출 · 남은 돈 카운트업 애니메이션, 수입 대비 지출 비율 바
- **지출 분석** — 분류별 금액·비율 막대 그래프 (상위 6개)
- **전체 내역** — 전체 / 수입 / 지출 필터
- **귀여운 애니메이션** — 돼지 마스코트(눌러보세요!), 컨페티, 팝인·슬라이드 전환, 몽글몽글 배경

> 접근성: 시스템의 "동작 줄이기(reduce motion)" 설정을 켜면 애니메이션이 자동으로 꺼집니다.

## 사용법

https://jeongsoocho.github.io/gagyebu/ 에서 바로 사용할 수 있습니다. 로그인 없이 열면 바로 내역이 뜹니다.

## 동작 방식

- 저장은 낙관적 갱신입니다. 화면을 먼저 바꾸고 DB 요청은 뒤따라가며, 실패하면 되돌린 뒤 사유를 토스트로 알려줍니다
- 예전 localStorage 내역이 남아 있으면 처음 열 때 DB로 한 번 옮기고, 원본은 `gagyebu_entries_backup` 키에 사본으로 남겨둡니다 (DB에 이미 내역이 있으면 중복 방지를 위해 건너뜁니다)

## ⚠️ 보안

이 앱에는 **로그인이 없습니다.** `public.entries` 테이블의 RLS 정책이 `anon` 롤에 조회·추가·수정·삭제를 모두 열어두고 있고, publishable 키는 HTML 안에 들어 있습니다.

따라서 **앱 주소를 아는 사람은 누구나 내역을 보고 고치고 지울 수 있습니다.** 공개된 주소로 개인 가계부를 쓰는 셈이니 감안하고 사용하세요. 나중에 막고 싶다면 Supabase Auth(이메일 매직링크 등)를 붙이고 `entries`에 `user_id` 컬럼과 `auth.uid() = user_id` 정책을 되살리면 됩니다.

> `service_role` 키는 절대 HTML에 넣지 마세요. 노출되면 RLS가 통째로 우회됩니다.

## 스키마

```sql
create table public.entries (
  id         bigint generated always as identity primary key,
  type       text not null check (type in ('income', 'expense')),
  date       date not null,
  amount     bigint not null check (amount > 0),
  category   text not null,
  memo       text not null default '',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

## 기술

HTML / CSS / 바닐라 JavaScript 단일 파일 + [supabase-js](https://github.com/supabase/supabase-js) (CDN).
