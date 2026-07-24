---
title: 영어단어 외우기
description: 너가 지켜야 할 것들
---

# TOEIC 단어장 앱 — 이어가기 노트 (핸드오프)

새 채팅방에서 이 파일을 붙여넣으면 맥락 없이도 이어서 작업할 수 있습니다.

## 무엇을 만드는가
사용자(한국인 영어 학습자, 비개발자)가 쓰는 **오프라인 TOEIC 단어 암기 웹앱**.
단일 HTML 파일이며 빌드/서버 없음. PDF 단어장을 넣으면 단어 카드로 만들어 발음·뜻·예문과 함께 공부.

## 파일 & Git
- 메인 파일(= 소스): `output/toeic_offline_vocabulary.html` (약 7.4MB, 대부분 base64 내장 발음 오디오)
- `index.html` = 메인 파일의 **복사본** (GitHub Pages 업로드용). **메인을 고치면 반드시 `cp output/toeic_offline_vocabulary.html index.html` 로 동기화 후 커밋.**
- git repo: `/Users/iyongho/Documents/English`, 브랜치 `main`. 최근 커밋: `eab1205`(무료 뜻·예문 채우기), `28a5bcd`(동기화+index.html), `72565b7`(최초).
- 7.4MB 단일 라인(오디오 blob)이 있어 `grep`/`node`가 종종 실패함 → 편집은 Edit 도구, 텍스트 처리는 `perl`/`sed`/`awk` 사용(grep은 그 라인을 통째로 날림).

## ⚠️ 최우선 제약: 유료 API 절대 안 씀
사용자가 크레딧 결제를 안 하므로 **Anthropic API는 완전히 제거됨**(키 입력·모델선택 없음, `api.anthropic.com` 호출 0). 앞으로도 유료 API 쓰지 말 것.
대신 **무료·키없음·CORS 되는 공개 API**만 사용:
- **MyMemory** `https://api.mymemory.translated.net/get?q=...&langpair=en|ko` → 한국어 뜻/문장 번역 (CORS `*`, 익명 하루 한도 있음, `quotaFinished`/"MYMEMORY WARNING"로 한도 감지)
- **dictionaryapi.dev** `https://api.dictionaryapi.dev/api/v2/entries/en/{word}` → 영어 예문 (CORS `*`, 구/숙어는 404)
- **jsonblob** `https://jsonblob.com/api/jsonBlob` → 기기 간 동기화 저장소 (CORS `*`, POST가 `Location`/`X-jsonblob-id` 헤더 노출, 계정 불필요, ~30일 미사용 시 만료)
- **claude.ai 링크**: 🌱 어원 / 다른 뜻 예문 버튼은 `claude.ai/new?q=...`를 새 탭으로 열어 무료로 물어봄(앱으로 데이터 회수는 안 됨).
- Tatoeba는 CORS 헤더 없어서 **브라우저에서 못 씀**(확인함).

## 현재 기능
- 기본 내장 240단어(unit13/14/part6) + 발음(base64 오디오 → 없으면 기기 TTS)
- 검색 / 단원 필터 / **완료함 폴더**(완료 체크한 단어는 다른 뷰에서 숨고 "✅ 완료함"에만; 상태 드롭다운은 제거됨)
- **한 단어씩 뜻 가리기**(👁, `state.hidden` Set) + 상단 "뜻 가리기"로 현재 목록 일괄
- **다른 뜻 예문**(＋ 버튼): PDF의 senses를 순회, 없으면 claude.ai 링크. 최대 2개 표시
- **품사 배지 / senses**(PDF 항목에 있으면)
- **페이지네이션**: 하단 페이저 + 화면 좌우 고정 화살표 + 키보드 ←/→ (PAGE_SIZE=10)
- **PDF 넣기**(pdf.js CDN, 여러 파일 동시 → 각각 별도 그룹). 그룹은 ✏️ 이름변경 / ✕ 삭제. 제목은 PDF 내용에서 자동 추정(Unit/Day/N강 등, 없으면 파일명)
- **무료 뜻·예문 자동 채우기**: PDF 넣으면 MyMemory+dictionaryapi로 자동 채움(진행률 표시). 이미 만든 그룹은 상단 노란 바 "🌐 무료로 채우기" 버튼으로 채움. (`fillDeckFree(deckId)`)
- **⚙️ 설정 패널**: 음성 선택 / "모든 단어 이 음성" / 음량·속도 슬라이더 / **☁️ 기기 간 동기화**(동기화 시작·코드로 연결·코드 복사·끄기)
- **기기 간 동기화**: jsonblob에 done+decks+words 저장. 한 기기 "동기화 시작"→코드 생성, 다른 기기 "코드로 연결"→코드 입력. 변경 시 자동 push(디바운스), 로드 시 pull. (`initSync`, `schedulePush`는 `saveDone`/`saveUserData`에서 호출, `applyingSync`로 에코 방지)

## localStorage 키
`toeic-offline-vocabulary-done`(완료), `vocab-user-decks-v1`, `vocab-user-words-v1`, `vocab-voice-v1`, `vocab-voice-all-v1`, `vocab-volume-v1`, `vocab-rate-v1`, `vocab-sync-code-v1`(동기화 코드). ID는 문자열로 비교(기본단어 숫자, PDF단어 `pdf_...`). ※ localStorage는 기기별이라 동기화는 위 jsonblob로 해결.

## 호스팅 (중요)
- 사용자는 **"링크 하나로 폰·아이패드·맥북에서 접속"** 원함 → **GitHub Pages** 선택함.
- 비공개 **Artifact**(`claude.ai/code/artifact/7a94b169-f81e-423c-9832-1574b1641d8c`, 조각파일 `scratchpad/vocab-app.html`)도 게시했지만 **아티팩트 샌드박스는 외부 네트워크·CDN 차단**이라 PDF·번역·동기화 전부 안 됨 → **무시**. GitHub Pages 링크가 진짜.
- 이 세션엔 `gh` 미설치/미인증이라 내가 직접 못 올림. 사용자가 github.com 웹UI로 `index.html`(리포 루트, 파일명 그대로)을 올리고 Settings→Pages→main/(root) 켜면 `https://<id>.github.io/<repo>/` 링크 생성.

## 지금 열린 할 일 / 사용자에게 안내할 것
1. **사용자가 GitHub Pages에 `index.html` 올리는 단계**를 아직 안 했을 수 있음(도와주기).
2. **이미 넣어둔(뜻 빠진) PDF 그룹**: 자동채우기는 "이번 수정 이후의 새 import"에만 자동 적용. 기존 그룹은 그 그룹을 연 뒤 상단 **"🌐 무료로 채우기"** 버튼을 눌러야 채워짐.
3. MyMemory 뜻은 대표 번역 1개라 토익 특정 뜻이 아닐 수 있음(예: release→"석방하다"). 필요하면 개선 여지.

## 한계 / 주의
- MyMemory 익명 하루 한도(대량 PDF 시 도달 가능) → 한도 감지해서 멈추고 "다시 누르면 이어서" 안내함. 이메일 파라미터(`&de=`)로 한도↑ 가능(개선안).
- dictionaryapi는 단일 단어 위주 → 구/숙어는 예문 없음(뜻만 채워짐).
- 스캔 이미지 PDF는 pdf.js 텍스트 추출 불가(글자 선택 가능한 PDF만).
- 미리보기 브라우저가 자주 멈춤 → 테스트는 `mcp__Claude_Browser`의 `javascript_tool`로 함수 직접 호출해 검증했음(예: `fillDeckFree`, `createSync`/`pullSync` 왕복). `node` 없음.

## 다음 단계 아이디어 (요청 시)
- MyMemory 뜻을 토익 빈출 뜻에 맞게 보정 / 여러 뜻 표기.
- 기존 그룹 자동 일괄 채우기(앱 로드 시 빠진 것 백그라운드 채움).
- 예문 소스 보강(dictionaryapi 없는 단어용 대체 무료 소스).
- 오디오 blob 분리해 파일 크기 축소(모바일 첫 로딩 빠르게).
