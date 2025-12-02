---
title: "[Winform] UI 커스터마이징"

categories:
  -  Winform
  
tags:
  - [Winform, C#]

toc: true
toc_sticky: true

published: false

date: 2024-12-30
last_modified_at: 2024-12-30
---

## AutoUpdater

### 개요

1. 프로그램 시작시, AutoUpdateForm 등장
2. 서버에 있는 txt 파일 확인
3. txt파일 안에 있는 버전 정보 확인
4. 업데이트가 필요하다면 업데이트 실행(필요없다면, Form 닫고 메인폼으로 이동)
5. txt파일에 있는 서버 url을 가져와서, 서버 경로에 있는 zip파일 다운로드
6. zip파일 압축해제 후, 새로운 실행파일 가져옴
7. 배치파일 생성, 배치파일은 현재 실행되고 있는 프로세스를 중지하고, 새로 가져온 실행파일을 가져와 기존의 exe를 삭제하고, 새로운 exe로 대체 후 실행
8. 업데이트 완료


### 클래스

1. AutoUpdate : 업데이트 관련 로직 관련 클래스
2. WebData : 웹에서 데이터 받아오는 로직 관련 클래스
3. AutoUpdateConfig : AutoUpdate에 필요한 정보들이 모여있는 클래스
4. AutoUpdateForm : 제일 처음 실행되어 업데이트가 필요한지 체크후, 업데이트 해주는 클래스
5. MainForm : 프로그램 메인화면

### 살펴봐야 할 곳

1. AutoUpdateConfig 라는 클래스에 정보 기입
2. update.txt파일을 열어, 정보를 형식에 맞게 변경 (파일명 변경 시, AutoUpdateForm에도 변경 필요)
3. 새 exe는 Zip 안에 exeName과 동일한 이름으로 있다고 가정 (필요시 수정 필요 - RunUpdate 함수 아래에 주석으로 달아놓음)
4. 어셈블리 버전 정보는 새로운 exe 파일에 반영되어 있을것으로 가정함. 따라서 현재 코드에는 업데이트 후에 어셈블리 버전 정보를 업데이트해주는 로직은 없음
5. Ionic.Zip.dll 을 사용하여 압축해제함