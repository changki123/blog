+++
title = "MSSQL 서버 — DB 복원 및 로그인 재연결"
date = 2026-04-09
[taxonomies]
tags = ["mssql", "ktcloud", "migration", "ssms"]
+++

KT Cloud에서 NCP(Naver Cloud Platform)로 서버를 이전하면서 MSSQL DB를 새 인스턴스에 복원한 과정을 정리한다.  
플랫폼은 달라졌지만 핵심은 MSSQL 자체의 작업이라 다른 환경에서도 동일하게 적용된다.

---

## 환경

| 항목 | 내용 |
|---|---|
| 서버 이미지 | `mssql(2017std)-win-2016-64-en` |
| SSMS | 18.0 |
| .NET Framework | 4.6 (SSMS 18 설치 전제 조건) |

---

## 1. 사전 준비
<img width="657" height="561" alt="Image" src="https://github.com/user-attachments/assets/0c8dc9e7-c731-4704-9681-2c7b1b7d10d7" />
**.NET Framework 4.6** 설치 후 **SSMS 18.0** 설치.  
설치 완료 후 서버 재기동.

---

## 2. DB 복원

<img width="554" height="290" alt="Image" src="https://github.com/user-attachments/assets/5ad5da64-7754-46c3-b2dc-9ba0b29bca0d" />
SSMS 접속 → 

<img width="391" height="455" alt="Image" src="https://github.com/user-attachments/assets/db5d45eb-0ff8-45c5-880b-add6c2c3010f" />
데이터베이스 우클릭 → **데이터베이스 복원** 선택.

<img width="861" height="744" alt="Image" src="https://github.com/user-attachments/assets/884106cb-e70d-4013-aebe-e5455d33ce6e" />
디바이스 → ... 선택 → 

<img width="572" height="337" alt="Image" src="https://github.com/user-attachments/assets/86135586-cd5a-4a03-9423-0bb7bec456b1" />
추가 → 

<img width="740" height="433" alt="Image" src="https://github.com/user-attachments/assets/bf23e77c-b39b-4b9f-8dff-b7facf59de2c" />
`.bak` 파일 경로 지정 → 

<img width="576" height="341" alt="Image" src="https://github.com/user-attachments/assets/b0bd8f10-6933-4151-8294-e6199a19b65c" />
`.bak` 파일 확인 → 

<img width="863" height="743" alt="Image" src="https://github.com/user-attachments/assets/5105d845-5327-4f1f-8404-a3cfe40bb35d" />
복원 실행.

<img width="347" height="165" alt="Image" src="https://github.com/user-attachments/assets/55e00d87-e650-4d14-8588-33549afa7db2" />

---

## 3. 로그인 재연결 (Orphaned User 처리)

DB만 복원하면 **DB 유저는 들어오지만 서버 로그인과의 매핑이 끊긴다.**  
SID가 같아도 "이 DB 유저 = 저 로그인"이라는 연결 정보가 날아가기 때문이다.

### 3-1. Orphaned 유저 확인

```sql
USE [DB이름];
EXEC sp_change_users_login 'Report';
```

결과에 유저가 나오면 Orphaned 상태.

### 3-2. 원본 서버에서 비밀번호 해시 + SID 추출

원본 서버에서 실행:

```sql
USE master;
SELECT name, password_hash, sid
FROM sys.sql_logins
WHERE name IN ('유저1', '유저2');
```

### 3-3. 새 서버에서 동일 SID + 해시로 로그인 생성

SID가 일치하면 DB 유저와 자동 매핑된다.

```sql
USE master;
CREATE LOGIN 유저1
WITH PASSWORD = 0x해시값 HASHED,
     SID = 0xSID값;

CREATE LOGIN 유저2
WITH PASSWORD = 0x해시값 HASHED,
     SID = 0xSID값;
```

### 3-4. Orphaned 0개 확인

```sql
USE [DB이름];
EXEC sp_change_users_login 'Report';
```

<img width="226" height="78" alt="Image" src="https://github.com/user-attachments/assets/00ec3b83-ccee-4bc8-b580-6c0f477d779b" />

결과가 비어 있으면 완료.

---

## 4. 접속 확인

SSMS → 새 연결 → 해당 로그인으로 접속 후 쿼리 확인.

```sql
USE [DB이름];
SELECT TOP 5 * FROM 테이블명;
```

---

## 정리

| 단계 | 핵심 |
|---|---|
| DB 복원 | .bak 파일로 복원, DB 유저는 따라옴 |
| 로그인 재생성 | SID + 비밀번호 해시를 원본에서 추출해 동일하게 생성 |
| 매핑 확인 | `sp_change_users_login 'Report'` 결과 0건 |

SID만 맞춰주면 `sp_change_users_login 'Auto_Fix'` 없이도 자동 매핑된다는 점이 포인트.