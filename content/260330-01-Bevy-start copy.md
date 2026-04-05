+++
title = "Rust로 게임 만들기 — Bevy 엔진 Windows 환경 세팅부터 첫 실행까지"
date = 2026-03-30
[taxonomies]
tags = ["rust", "bevy", "gamedev", "windows"]
+++

Rust로 게임을 만들어보고 싶어서 이것저것 찾아봤는데, 현재 Rust 게임 개발 생태계에서 가장 활발한 엔진은 단연 **Bevy**다. ECS(Entity Component System) 기반의 풀스택 게임 엔진으로, 커뮤니티도 크고 문서도 잘 되어있다.

이 글에서는 Windows 환경에서 Rust + Bevy 개발 환경을 세팅하고 첫 번째 창을 띄우기까지의 과정을 기록한다.

<!-- more -->

## Bevy가 뭔데?

Bevy는 Rust로 작성된 오픈소스 게임 엔진이다. 2D/3D 모두 지원하고, 크로스플랫폼(Windows, Mac, Linux, WASM)을 지원한다. 가장 큰 특징은 **ECS 아키텍처**를 채택했다는 점이다.

ECS는 세 가지 개념으로 이루어진다.

- **Entity** — ID만 가진 빈 컨테이너. 플레이어, 적, 총알 등이 각각 하나의 Entity다.
- **Component** — 데이터만 보유하는 구조체. `Transform`(위치), `Sprite`(이미지), `Health`(체력) 같은 것들.
- **System** — 매 프레임 실행되는 함수. Component를 읽고 수정하는 로직이 여기 들어간다.

전통적인 OOP 방식과 달리 상속 없이 컴포넌트 조합으로 게임 오브젝트를 구성한다. 처음엔 낯설지만 익숙해지면 Rust의 소유권 모델이랑 정말 잘 맞는다.

## 환경 세팅 (Windows)

### 1. Rust 설치

[rustup.rs](https://rustup.rs)에서 `rustup-init.exe`를 다운로드해서 실행한다.

설치 중에 Visual C++ 빌드 툴 관련 옵션이 나오는데, **옵션 1 (Visual Studio Community 설치)** 을 선택한다. VS Community 인스톨러에서 **"C++를 사용한 데스크톱 개발"** 워크로드만 체크하면 된다.

설치가 끝나면 PowerShell을 새로 열고 확인한다.

```powershell
rustc --version
cargo --version
```

버전이 출력되면 성공이다.

### 2. VS Code + 확장 설치

에디터는 VS Code를 추천한다. 설치 후 아래 두 확장을 설치한다.

- `rust-analyzer` — 자동완성, 타입 힌트, 에러 표시. 사실상 필수다.
- `Even Better TOML` — `Cargo.toml` 편집 시 문법 강조.

### 3. 프로젝트 생성

```powershell
cargo new my_game
cd my_game
code .
```

`code .`를 입력하면 VS Code가 해당 폴더로 바로 열린다.

## Bevy 추가 및 첫 실행

### Cargo.toml 설정

`Cargo.toml`을 아래처럼 수정한다. `[profile.dev]` 설정은 개발 중 컴파일 속도를 개선해준다. 없어도 되지만 있으면 체감이 꽤 다르다.

```toml
[package]
name = "my_game"
version = "0.1.0"
edition = "2021"

[dependencies]
bevy = "0.15"

[profile.dev]
opt-level = 1

[profile.dev.package."*"]
opt-level = 3
```

### main.rs

`src/main.rs`를 아래로 교체한다.

```rust
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .run();
}
```

`DefaultPlugins`는 윈도우 생성, 렌더링, 입력 처리, 오디오 등 게임에 필요한 기본 기능을 한 번에 추가해준다.

### 첫 실행

```powershell
cargo run
```

**처음 빌드는 5~10분 정도 걸린다.** Bevy 전체를 컴파일하는 거라 정상이다. 이후 증분 빌드는 훨씬 빠르다.

빌드가 끝나면 검은 빈 창이 하나 뜨는데, 이게 성공이다.

## 다음 단계

빈 창이 떴으면 이제 뭔가 채울 차례다. 앞으로 다룰 내용들이다.

- 스프라이트 띄우기 (캐릭터 화면에 그리기)
- 키보드 입력으로 캐릭터 움직이기
- Bevy UI로 HUD 만들기
- 충돌 처리

Bevy 공식 예제(`cargo run --example sprite` 등)도 매우 풍부하니 참고하면 좋다.

## 참고 링크

- [Bevy 공식 사이트](https://bevyengine.org)
- [Bevy 공식 예제](https://github.com/bevyengine/bevy/tree/main/examples)
- [Bevy Cheat Book](https://bevy-cheatbook.github.io) — 비공식이지만 실용적인 레퍼런스