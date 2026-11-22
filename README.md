# 소프트웨어 공학 2026 기말 팀 프로젝트

## 1. 개요

- **장르**: 텍스트 기반 턴제 RPG
- **핵심 루프**: 던전 진입 → 랜덤 이벤트(전투 / 상점 / 선택지) 반복 → 보스전 → 다음 스테이지
- **우선 구현 범위**: 전투, 던전, 상점

```
[마을/시작] → [스테이지 1: 이벤트 N회 → 보스] → [마을/시작] → [스테이지 2: ...] → ... → [마을/시작]
```

## 2. 전투 시스템

쯔꾸르(RPG Maker) 스타일의 턴제 커맨드 전투.

### 2.1 커맨드

| 커맨드 | 설명 |
|--------|------|
| **공격** | 기본 공격. MP 소모 없음 |
| **스킬** | MP를 소모해 강한 공격 / 회복 / 버프 사용 |
| **방어** | 이번 턴 받는 데미지 감소 (예: 50%) |
| **아이템** | 인벤토리의 소모품 사용 (포션 등) |


### 2.2 턴 진행

1. 플레이어 커맨드 선택
2. 속도(SPD) 비교로 행동 순서 결정
3. 행동 처리 → 결과 출력
4. 승패 판정 (HP 0 체크)
5. 다음 턴

### 2.3 스탯

| 스탯 | 설명 |
|------|------|
| HP | 체력. 0이 되면 사망 |
| MP | 스킬 사용 자원 |
| ATK | 공격력 |
| DEF | 방어력 |
| SPD | 행동 순서 결정 |

### 2.4 데미지 공식 (초안)

```
데미지 = max(1, ATK - DEF / 2) × 랜덤(0.9 ~ 1.1)
방어 중이면 데미지 × 0.5
```


### 2.5 전투 보상

- 경험치 → 레벨업 시 스탯 상승
- 골드 → 상점에서 사용
- 낮은 확률로 아이템 드롭

### 2.6 출력 예시

```
=== 슬라임이 나타났다! ===
[용사] HP 50/50  MP 20/20
[슬라임] HP 30/30

1. 공격  2. 스킬  3. 방어  4. 아이템
> 1
용사의 공격! 슬라임에게 12의 데미지!
슬라임의 공격! 용사에게 3의 데미지!
```

---

## 3. 던전 시스템

### 3.1 구조

- 던전은 여러 **스테이지**로 구분
- 한 스테이지 = **한 사이클**
- 사이클: 랜덤 이벤트 N회 반복 → 마지막에 **보스전**

```
스테이지 1
 ├─ 이벤트 1 (랜덤)
 ├─ 이벤트 2 (랜덤)
 ├─ ...
 ├─ 이벤트 N (랜덤)
 └─ 보스전
```

### 3.2 랜덤 이벤트 종류

| 이벤트 | 내용 | 확률 (초안) |
|--------|------|-------------|
| **전투** | 스테이지에 맞는 일반 몬스터와 전투 | 60% |
| **상점** | 떠돌이 상인 등장, 아이템 구매/판매 | 15% |
| **선택지** | 텍스트 이벤트, 선택에 따라 보상 또는 페널티 | 25% |

### 3.3 선택지 이벤트 예시

```
낡은 상자를 발견했다.
1. 연다
2. 무시하고 지나간다
```
- 1 선택 → 50%: 골드 획득 / 50%: 함정(HP 감소) 또는 미믹 전투
- 2 선택 → 아무 일 없음

### 3.4 사망 처리 

- 마을로 귀환, 골드 일부 손실

---

## 4. 상점 시스템

### 4.1 등장 위치

- 마을 상점 (스테이지 사이)
- 던전 내 랜덤 이벤트 (떠돌이 상인)

### 4.2 기능

- **구매**: 골드로 아이템 구매
- **판매**: 보유 아이템을 일정 비율(예: 50%) 가격에 판매

### 4.3 아이템 목록 (초안)

| 아이템 | 종류 | 효과 | 가격 |
|--------|------|------|------|
| 포션 | 소모품 | HP 30 회복 | 20G |
| 하이포션 | 소모품 | HP 80 회복 | 50G |
| 에테르 | 소모품 | MP 20 회복 | 30G |
| 철검 | 장비(무기) | ATK +5 | 100G |
| 가죽갑옷 | 장비(방어구) | DEF +3 | 80G |

### 4.4 출력 예시

```
=== 상점 ===  보유 골드: 120G
1. 포션 (20G)
2. 에테르 (30G)
3. 철검 (100G)
0. 나가기
>
```

---


## 5. 데이터 구조 (초안)

```
Player   : name, level, exp, hp, maxHp, mp, maxMp, atk, def, spd, gold, inventory, skills
Monster  : name, hp, atk, def, spd, expReward, goldReward, dropTable
Skill    : name, mpCost, power, type(공격/회복/버프)
Item     : name, type(소모품/장비), effect, price
Stage    : number, eventCount, monsterPool, boss
Event    : type(전투/상점/선택지), data
```

---

## 6. 저장 시스템

### 6.1 방식: JSON 파일

- 플레이어 데이터는 인벤토리, 스킬 목록처럼 **중첩/리스트 구조**라 JSON이 자연스러움
- CSV는 한 줄 = 한 레코드인 표 형식이라 중첩 데이터 표현이 불편
- 사람이 읽고 직접 수정하기 쉬움 (디버깅·테스트 편함)
- 대부분 언어에 기본 또는 표준 라이브러리 지원

> CSV는 **몬스터/아이템 테이블** 같은 고정 데이터 정의용으로는 쓸 만함 (엑셀로 밸런스 편집)


### 6.2 저장 시점

- 마을 귀환 시 / 스테이지 클리어 후 자동 저장
- 던전 진행 중 저장 여부는 사망 처리 방식과 함께 결정 (로그라이크면 던전 중 저장 불가)


### 6.3 저장 데이터 예시 (`save.json`)

```json
{
  "player": {
    "name": "용사",
    "level": 3,
    "exp": 45,
    "hp": 62,
    "maxHp": 70,
    "mp": 15,
    "maxMp": 25,
    "atk": 14,
    "def": 8,
    "spd": 6,
    "gold": 230
  },
  "inventory": [
    { "id": "potion", "count": 3 },
    { "id": "ether", "count": 1 }
  ],
  "equipment": {
    "weapon": "iron_sword",
    "armor": "leather_armor"
  },
  "skills": ["slash", "heal"],
  "progress": {
    "clearedStage": 1
  }
}
```

- 아이템/스킬은 **id만 저장**, 실제 수치는 게임 데이터에서 조회 → 밸런스 바꿔도 세이브 안 깨짐
- 파일 없거나 깨졌으면 새 게임으로 시작

---

## 7. 2인 협업 구조 (Java)

### 7.1 원칙

- **공통 영역(model, api)만 먼저 같이 확정**, 이후엔 각자 구현
- 서로의 코드는 **인터페이스로만 호출** → 상대 구현이 없어도 작업 가능
- 상대 파트가 미완성이면 **Fake(가짜) 구현**으로 대체해서 테스트
- 실제 구현 연결은 `Main`에서 생성자로 주입

### 7.2 역할 분담

| 담당 | 파트 | 내용 |
|------|------|------|
| **A** | 전투 | 턴 진행, 4개 커맨드(공격/스킬/방어/아이템), 데미지 계산, 경험치·레벨업, 몬스터 데이터 |
| **B** | 진행 | 메인 메뉴, 던전 사이클, 랜덤 이벤트 3종, 상점, 저장/불러오기 |
| 공통 | 모델 | Player, Monster, Item, Skill, 인터페이스 정의 (첫 주에 함께) |

### 7.3 패키지 구조

```
rpg/
 ├─ model/     (공통) Player, Monster, Item, Skill
 ├─ api/       (공통) 인터페이스, BattleResult, SaveData
 ├─ battle/    (A)   TurnBattleSystem, StageMonsterFactory
 ├─ dungeon/   (B)   Dungeon, BattleEvent, ShopEvent, ChoiceEvent
 ├─ shop/      (B)   ItemShop
 ├─ save/      (B)   JsonSaveManager
 └─ Main.java  (B)
```

### 7.4 공통 인터페이스 (`api` 패키지)

```java
public enum BattleResult {
    WIN, LOSE
}
```

```java
public interface BattleSystem {
    BattleResult fight(Player player, Monster monster);
}
```

```java
public interface MonsterFactory {
    Monster createNormal(int stage);
    Monster createBoss(int stage);
}
```

```java
public interface GameEvent {
    void run(Player player);
}
```

```java
public interface Shop {
    void open(Player player);
}
```

```java
public interface SaveManager {
    void save(SaveData data);
    SaveData load();
}
```

| 인터페이스 | 구현 | 사용 |
|------------|------|------|
| BattleSystem | A | B (던전 전투 이벤트, 보스전) |
| MonsterFactory | A | B (스테이지별 몬스터 생성) |
| GameEvent | B | B (던전 내부) |
| Shop | B | B (마을, 떠돌이 상인 이벤트) |
| SaveManager | B | B (메인 메뉴) |

### 7.5 Fake 구현으로 독립 작업

B는 전투가 완성되기 전에도 던전을 테스트할 수 있음:

```java
public class FakeBattleSystem implements BattleSystem {
    public BattleResult fight(Player player, Monster monster) {
        return BattleResult.WIN;
    }
}
```

A도 던전 없이 전투만 따로 실행해서 테스트:

```java
public class BattleTest {
    public static void main(String[] args) {
        BattleSystem battle = new TurnBattleSystem();
        Player player = new Player("용사");
        Monster monster = new StageMonsterFactory().createNormal(1);
        System.out.println(battle.fight(player, monster));
    }
}
```

### 7.6 연결 (`Main`)

```java
public class Main {
    public static void main(String[] args) {
        BattleSystem battle = new TurnBattleSystem();
        MonsterFactory monsters = new StageMonsterFactory();
        Shop shop = new ItemShop();
        SaveManager save = new JsonSaveManager();

        Dungeon dungeon = new Dungeon(battle, monsters, shop);
        new Game(dungeon, shop, save).start();
    }
}
```

- 완성 전에는 `new FakeBattleSystem()`으로 바꿔 끼우면 됨
- `Dungeon`은 `BattleSystem` 인터페이스만 알고, `TurnBattleSystem` 내부는 모름

### 7.7 협업 규칙

- 인터페이스나 model을 바꿀 땐 **반드시 상대와 합의 후 변경**
- Git 브랜치: `develop` / `battle`(A) / `progress`(B), 기능 완성 단위로 merge
- 각자 자기 패키지만 수정 → 충돌 최소화
- JSON 저장은 Gson 같은 라이브러리 사용 여부 미리 확인

---

## 8. 개발 순서 (초안)

| 단계 | A (전투) | B (진행) |
|------|----------|----------|
| 1 | 공통 model / api 함께 확정 | 공통 model / api 함께 확정 |
| 2 | 기본 공격 전투 + MonsterFactory | 메인 메뉴 + 던전 사이클 (FakeBattleSystem) |
| 3 | 방어 / 스킬 / 아이템 커맨드 | 이벤트 3종 (전투 / 상점 / 선택지) |
| 4 | 경험치 / 레벨업 / 보상 | 상점 구매·판매 + 저장/불러오기 |
| 5 | 통합: Fake → 실제 구현 교체 | 통합: Fake → 실제 구현 교체 |
| 6 | 밸런스 조정, 스테이지 확장 | 밸런스 조정, 스테이지 확장 |

---
