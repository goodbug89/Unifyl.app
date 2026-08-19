<p align="center"><img src="docs/screenshots/icon.png" alt="Unifyl" width="128" height="128"></p>
<h1 align="center">Unifyl</h1>
<p align="center"><strong>Unifyl(유니파일) — macOS용 AI 듀얼 패인 파일 관리자.</strong><br>Total Commander의 깊이, ForkLift의 마감, 내장 AI 지능.</p>

<p align="center"><a href="https://unifyl.app/ko/">웹사이트</a> &middot; <a href="https://github.com/goodbug89/Unifyl.app/wiki">문서</a> &middot; <a href="https://github.com/goodbug89/Unifyl.app/releases/latest">다운로드</a></p>

<p align="center"><a href="https://apps.apple.com/app/unifyl/id6773977314?mt=12">Mac App Store</a>에서도 다운로드할 수 있습니다 (샌드박스 에디션 — 기능 차이는 FAQ 참조).</p>

<p align="center"><strong>언어:</strong> <a href="README.md">English</a> &middot; <a href="README.ja.md">日本語</a> &middot; <a href="README.zh-Hans.md">简体中文</a> &middot; <strong>한국어</strong> &middot; <a href="README.de.md">Deutsch</a> &middot; <a href="README.es.md">Español</a> &middot; <a href="README.fr.md">Français</a></p>

---

![Unifyl 스크린샷](docs/screenshots/01-dualpane.png)

## 개요

Unifyl은 전문가용 파워 툴과 로컬 AI 지능을 결합한 macOS 듀얼 패인 파일 관리자입니다. 당신의 파일은 Mac을 떠나지 않습니다.

- **듀얼 패인** — 싱글/듀얼/트리플/자유 분할, 탭, 워크스페이스, 패널별 인라인 터미널
- **AI 기반** — 시맨틱 검색, AI 리네임, 스마트 태깅, 중복 탐지 — 모두 Apple Neural Engine에서 로컬 실행
- **원격 & 클라우드** — FTP/SFTP/WebDAV/SMB/S3/Google Drive/Dropbox/OneDrive/Docker/Kubernetes
- **50개 이상의 전문 도구** — 일괄 이름 바꾸기, 텍스트/바이너리/이미지 비교, 디렉터리 동기화, 3-way 머지, EXIF 일괄 편집
- **아카이브를 가상 폴더로** — ZIP/7z/TAR/GZ/BZ2/XZ/ZSTD/RAR (CJK 레거시 ZIP 자동 인코딩 감지 포함)
- **키보드 우선** — 명령 팔레트, Vim 스타일 네비게이션, 120개 이상의 재맵 가능한 단축키
- **15개 언어 지원**
- **CJK 강화** (1.3.0): `.alz` 아카이브 추출 (번들 `unalz`), `.hwpx` 인라인 미리보기, 다국어 파일명 검색 (Pinyin/Romaji/한글-한자)
- **CJK 추가 강화** (1.3.1): 메신저/클라우드 폴더 자동 북마크 (KakaoTalk · LINE · WeChat · iCloud Drive 등 12개), 全角 ↔ 半角 검색 폴딩, EUC-KR/Shift-JIS/GBK → UTF-8 인코딩 변환 (Undo 지원), 한국 정부서식 휴리스틱 검출

## 가격 — 무료 다운로드, Pro 일회성 $24.99

**1.4.0**부터 Unifyl은 Direct 사이트와 Mac App Store 양쪽에서
**무료로 다운로드**할 수 있습니다. **Pro 일회성 구매**로 50+ 파워
유저 기능 전체를 잠금 해제.

| | **Free** (누구나) | **Pro — $24.99 일회성** |
|---|---|---|
| 패인 레이아웃 | 듀얼/트리플/자유 분할 (모두) | (동일) |
| 탭 & 워크스페이스 | 무제한 | (동일) |
| 검색 | 파일명 + Spotlight + CJK 다국어 | + 본문 검색 (정규식) |
| 원격 연결 | FTP, SFTP | + FTPS, WebDAV, SMB, 모든 클라우드 (Google Drive, Dropbox, OneDrive, S3, B2, GCS, Azure Blob) |
| 아카이브 | ZIP 읽기 | + ZIP/7z/TAR/RAR 쓰기 + in-place 수정 + ALZ |
| 비교 & 동기화 | — | 파일 diff, 폴더 diff, 폴더 동기화, 3-way merge |
| Multi-rename | — | 정규식 + EXIF + AI 제안 + 프리셋 |
| AI 도구 | — | 시맨틱 검색, 자동 분류, AI 리네임, 중복 검출, 요약 (모두 온디바이스) |
| 무료로는 안 되는 일 | — | 두 폴더 일치 증명 · 수백 개 일괄 리네임 · 파일 내용 검색 · 전체 클라우드 · 히트맵, hex editor, file X-ray 등 + Git 패널·Docker·SSH 터널 *(직접 다운로드 전용)* |
| 테마 / UI | 12 빌트인 테마 + 15개 언어 + 함수 키 바 + Quick Look | (동일) |

> **에디션 안내.** Mac App Store 빌드는 샌드박스이며 Git 패널, Docker/Kubernetes, SSH 터널, 인라인 터미널, 포트·프로세스 뷰어, ffmpeg 기반 미디어 변환을 포함하지 않습니다 — macOS가 그곳에서 금지합니다. 나머지는 동일하고 Pro 가격도 양쪽 모두 같은 일회성 결제입니다. 위 기능이 필요하면 직접 다운로드를 선택하세요.

**일회성 구매. 구독 없음. 한 번 결제하고 그 Mac에서 평생 사용.**
Direct (LemonSqueezy — Apple Pay/카드, 3대 디바이스 전이 가능) 또는
Mac App Store In-App Purchase (Apple ID 연결, Family Sharing 지원)로
구매. 첫 실행 시 14일 Pro 무료 trial 자동 제공.

전체 기능, 단축키, 빌드 가이드, 플러그인 개발은 [English README](README.md) 참조.

## 라이선스

프로프라이어터리. Copyright 2024–2026 Unifyl. All rights reserved.
