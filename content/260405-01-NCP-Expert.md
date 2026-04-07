+++
title = "NCP Expert (NCE) 전과목 합격 후기"
date = 2026-04-05
[taxonomies]
tags = ["ncp", "nce", "자격증", "네이버클라우드"]
+++

## NCE 전과목 합격했습니다

MSP에서 NCP 환경을 실무로 다루다 보니 자연스럽게 NCE(Naver Cloud Platform Expert) 시험을 준비하게 됐습니다. NCE는 과목이 여러 개로 나뉘어 있는데, 이번에 전과목을 한 번에 합격했습니다.

<!-- more -->

**합격 과목**

| 과목 | 영역 |
|------|------|
| NCE 301 | Compute / Storage |
| NCE 302 | Network / Media |
| NCE 303 | Database / Management / Analytics / Security |
| NCE 305 | Application / AI |

---

## 공부 방법

**주요 학습 자료**

- **네이버 비즈니스 스쿨 클라우드**: [NCE 공식 강의](https://bizschool.naver.com/online/courses/76) — 과목별로 커리큘럼이 잘 나뉘어 있어서 기본기 잡는 데 충실했습니다.
- **AI 질문**: 애매한 개념은 바로바로 물어보면서 정리했습니다. 특히 NKS 내부 동작(ncp-auth ConfigMap, Cluster Autoscaler, drain vs cordon 차이 등)이나 서비스 간 차이점 비교할 때 유용했습니다.

**과목별 포인트**

- **301 Compute / Storage**: NKS, 쿠버네티스, NCP CLI, 기본 컴퓨트 위주로 출제됩니다. 쿠버네티스는 필수로 알고 가셔야 합니다.
- **302 Network / Media**: VPC 기본 네트워크 구성은 기본이고, VOD 등 평소에 접하기 어려운 미디어 서비스 관련 문제도 많이 나옵니다.
- **303 Database / Management / Analytics / Security**: CDB, Cloud Insight 등 데이터베이스, 모니터링, 보안까지 범위가 넓어 어디서부터 공부해야 할지 감 잡기 어려웠습니다. 일단 범위 안에서 최대한 커버하시길 바랍니다.
- **305 Application / AI**: CLOVA 서비스 관련 문제가 많이 나옵니다. CLOVA 서비스 종류를 정리해두는 게 핵심입니다. 서버리스 쪽은 개인적으로 관심이 있었는데 출제 비중은 높지 않았습니다.

네이버 비즈니스 스쿨 강의 외에도, 네이버 교육장에서 진행하는 NCP/NCE 오프라인 교육을 들을 수 있다면 추천합니다. 강사분들이 시험에 자주 나오는 포인트를 직접 짚어주고, 별도 교육 자료도 받을 수 있어서 훨씬 효율적입니다.

---

## 난이도

실무에서 NCP를 이미 쓰고 있다면 무난한 편입니다. 강의 한 번 보고 개념 정리 정도면 충분했습니다. 다만 303처럼 범위가 넓은 과목은 서비스 종류가 많아서 이름이랑 특성 매핑을 꼼꼼히 해두는 게 좋습니다.

클래식 플랫폼이 종료되면서 현재는 VPC 플랫폼 기준으로만 출제됩니다. 덕분에 범위가 오히려 명확해진 편입니다.

---

## 총평

| 항목 | 내용 |
|------|------|
| 시험 | NCP Expert (NCE) 301 / 302 / 303 / 305 |
| 결과 | **전과목 합격** |
| 추천 자료 | 네이버 비즈니스 스쿨 클라우드 강의 + AI 보조 학습 |
| 난이도 | 실무 경험 있으면 무난 |

NCP를 실무에서 다루고 있다면 충분히 도전할 만한 시험입니다.


<img width="865" height="598" alt="Image" src="https://github.com/user-attachments/assets/190911c0-d9a3-4686-a6e4-6e10e562aab8" />