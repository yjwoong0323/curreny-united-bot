# 'Currency United - CUUNIT' 환율 차익 거래

[backend project] 각 은행의 환율 비교 후 차익 거래 시스템 구현

<br/>
<br/>

## 🛠️ Stack
  **Backend**: Python, Kotlin<br/>
  **Database**: MySQL(main), Redis(Token, queue)<br/>
  **etc**: Docker

<br/>

## 🗂️ Directory
- `backend/`: Node.js + Express + Socket.IO + MySQL로 실시간 메시지 브로커 역할
- `frontend/`
  - `mobile/`: React Native (Original, CLI 버전)로 개발된 모바일 채팅 클라이언트
  - `web/`: Next.js로 개발된 웹 대시보드 (상담사 인터페이스)

<br/>

## 🚀 Features
- 실시간 각 은행 환율 정보 크롤링 후 DB 적재
- 각 은행 환율 정보 비교
- JWT 기반 인증
- 최적의 차익 거래 로직

<!-- <img width="322" height="auto" alt="Simulator Screenshot - iPhone 16 Pro - 2025-07-21 at 11 07 55" src="https://github.com/user-attachments/assets/31eda42b-0b2d-41d1-a20d-55002006da7a" /> -->
