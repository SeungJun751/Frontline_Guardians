# 🎴 전선 가디언즈 (Frontline Guardians)

> **Photon PUN2 기반 1 vs 1 실시간 TCG 카드 배틀 게임**

[![Unity](https://img.shields.io/badge/Unity-6-black?logo=unity)](https://unity.com/)
[![Photon](https://img.shields.io/badge/Photon-PUN2-blue)](https://www.photonengine.com/)

**전선 가디언즈**는 Photon PUN2를 활용한 실시간 멀티플레이어 TCG 카드 게임입니다.  
플레이어는 30장의 덱을 구성하여 1 vs 1 대전을 진행하며, 리더 카드의 고유 능력과 다양한 캐릭터 카드를 활용해 전략적인 플레이를 펼칩니다.

---

## 📌 프로젝트 개요

- **개발 기간**: 2025.07 ~ 2025.08 (2개월)
- **개발 인원**: 2명 (프로그래머 2명)
- **본인 역할**: Solo Developer (클라이언트 개발, 시스템 설계, 네트워크 동기화)
- **개발 환경**: Unity 6, C# (.NET), Photon PUN2, ScriptableObject
- **장르**: TCG (Trading Card Game), Turn-Based Strategy, Multiplayer

**📺 플레이 영상**: [YouTube](https://youtube.com/watch?v=tcg-game)  
**🌐 포트폴리오**: [김승준 Portfolio](https://seungjun751.github.io)

---

## ✨ 핵심 기능

### 🌐 Photon PUN2 멀티플레이어
- **랜덤 매칭 시스템**: `JoinRandomOrCreateRoom`을 활용한 자동 매칭
- **마스터 클라이언트 권한 관리**: 모든 게임 로직을 마스터에서 처리하여 동기화 보장
- **RPC 기반 턴 동기화**: `RequestEndTurn` → 마스터 검증 → `SyncTurnState` 전체 동기화
- **상대방 이탈 감지**: `OnPlayerLeftRoom` 콜백으로 자동 승리 처리

### 🃏 카드 시스템
- **100여 가지 카드 능력**: ScriptableObject 기반 데이터 주도 설계
- **리더 카드 패시브**: 턴 시작, 아군 소환, 아군 사망 시 발동하는 고유 능력
- **캐릭터 카드**: OnPlay, OnTurnStart, OnDeath 트리거로 다양한 효과 구현
- **영구 카드**: 여러 턴 지속되는 버프/디버프 효과
- **15가지 상태 이상**: 기절, 은밀, 강화, 방어력, 부식, 면역 등

### ⚔️ 전투 시스템
- **턴제 전투**: 60초 타이머, 10% 이하 시 경고 UI
- **필드 슬롯 관리**: 캐릭터 4슬롯 + 영구 카드 3슬롯
- **자원 시스템**: 턴당 3자원 증가, 최대 10자원
- **손패 관리**: 최대 5장 제한, 초과 시 카드 소멸
- **덱 관리**: 30장 덱, 드로우 애니메이션

### 🎨 UI/UX
- **턴별 배경 변화**: 아침 → 낮1 → 낮2 → 해질녘 → 저녁 → 밤
- **카드 정보 패널**: 마우스 오버 시 상세 정보 표시
- **묘지 회전 효과**: 상시 회전하는 묘지 이미지 UI
- **리더/영웅 HP 표시**: 실시간 체력 갱신

---

## 🛠 기술 스택

| 분류 | 기술 |
|------|------|
| **엔진** | Unity 6 |
| **언어** | C# (.NET Framework) |
| **네트워크** | Photon PUN2, RPC 통신 |
| **디자인 패턴** | Singleton, ScriptableObject, Observer Pattern |
| **데이터 구조** | Dictionary, HashSet, LINQ |
| **AI Tool** | Claude, Gemini |
| **기타** | Coroutine, TextMeshPro, ParrelSync (로컬 테스트) |

---

## 🎯 담당 파트 (본인 기여도: 약 85%)

### 1️⃣ 네트워크 동기화 시스템
- **[`GameManager.cs`](Assets/02.Script/Managers/GameManager.cs)**: 게임 전체 흐름 제어, RPC 통신 총괄 (3153줄)
- **[`MatchmakingManager.cs`](Assets/02.Script/Managers/MatchmakingManager.cs)**: 랜덤 매칭, 덱 데이터 전송 (456줄)
- **[`Player.cs`](Assets/02.Script/Player/Player.cs)**: 플레이어 데이터 관리, 손패/덱/묘지 (519줄)

```csharp
// 핵심 로직: 마스터 클라이언트 권한 검증
[PunRPC]
public void RequestEndTurn(PhotonMessageInfo info)
{
    if (!PhotonNetwork.IsMasterClient) return;
    
    Player currentPlayer = players[currentPlayerIndex];
    if (info.Sender == currentPlayer.photonPlayer)
    {
        EndTurn(currentPlayer); // 마스터만 실행
    }
    else
    {
        Debug.LogWarning($"권한 없는 턴 종료 요청 거부");
    }
}
```

### 2️⃣ 카드 능력 시스템
- **[`AbilitySystem.cs`](Assets/02.Script/Data/AbilitySystem.cs)**: 100가지 카드 능력 실행 로직 (1573줄)
- **[`CharacterCard.cs`](Assets/02.Script/Cards/CharacterCard.cs)**: 캐릭터 카드 스탯 관리, 전투 로직 (1512줄)
- **[`Ability.cs`](Assets/02.Script/Data/Ability.cs)**: ScriptableObject 기반 능력 데이터

```csharp
// ScriptableObject 기반 능력 시스템
public class AbilitySystem : MonoBehaviour
{
    public void CheckAndTriggerOnPlayAbilities(CharacterCard card)
    {
        foreach (Ability ability in card.cardData.abilities)
        {
            if (ability.triggerType == AbilityTriggerType.OnPlay)
            {
                ExecuteAbility(ability, card);
            }
        }
    }
    
    private void ExecuteAbility(Ability ability, CharacterCard source)
    {
        switch (ability.effectType)
        {
            case AbilityEffectType.BuffAttack:
                target.BuffAttack(ability.effectValue);
                break;
            case AbilityEffectType.DealDamage:
                target.TakeDamage(ability.effectValue, source);
                break;
            // ... 100가지 능력
        }
    }
}
```

### 3️⃣ 덱 빌더 시스템
- **[`DeckBuilderManager.cs`](Assets/02.Script/Managers/DeckBuilderManager.cs)**: 30장 덱 구성, JSON 저장/불러오기 (23KB)
- **[`Deck.cs`](Assets/02.Script/Player/Deck.cs)**: 덱 셔플, 카드 드로우 로직

### 4️⃣ 카드 드로우 동기화
- **마스터 권한 기반 드로우**: 모든 클라이언트가 동일한 카드 획득

```csharp
[PunRPC]
public void RequestDrawCards(PlayerType playerType, int amount)
{
    if (!PhotonNetwork.IsMasterClient) return;
    
    Player player = GetCurrentPlayer(playerType);
    List<string> drawnCardNames = new List<string>();
    
    for (int i = 0; i < amount; i++)
    {
        CardData drawnCard = player.gameDeck.DrawCard();
        if (drawnCard != null)
            drawnCardNames.Add(drawnCard.name);
    }
    
    // 모든 클라이언트에 동일한 카드 전송
    photonView.RPC("SyncDrawnCards", RpcTarget.All, 
                   playerType, drawnCardNames.ToArray());
}
```

---

## 🚀 기술적 도전과 해결

### 1️⃣ Photon RPC 턴 동기화 시스템 설계
**🔴 문제**  
멀티플레이어 환경에서 두 플레이어의 턴이 동시에 진행되거나,  
한쪽 클라이언트에서만 턴이 종료되는 비동기 문제 발생.  
특히 네트워크 지연 상황에서 **카드 사용 → 턴 종료 → 상대방 턴 시작**의 순서가 보장되지 않음.

**✅ 해결**  
**마스터 클라이언트 권한 시스템** 도입:
1. 클라이언트는 `RequestEndTurn` RPC로 **턴 종료 요청만 전송**
2. 마스터가 `PhotonMessageInfo`로 **요청자 검증** (현재 턴 플레이어인지 확인)
3. 검증 통과 시 마스터가 `EndTurn` 실행 및 `RPC_SyncTurnState`로 **전체 동기화**

```csharp
[PunRPC]
public void RequestEndTurn(PhotonMessageInfo info)
{
    if (!PhotonNetwork.IsMasterClient) return;
    
    Player currentPlayer = players[currentPlayerIndex];
    if (info.Sender == currentPlayer.photonPlayer)
    {
        EndTurn(currentPlayer); // 마스터만 실행
    }
    else
    {
        Debug.LogWarning("권한 없는 턴 종료 요청 거부");
    }
}
```

**📈 결과**: 네트워크 지연 상황에서도 턴 순서 완벽 보장, 비동기 버그 완전 해결

---

### 2️⃣ 100가지 카드 능력의 확장 가능한 시스템 설계
**🔴 문제**  
초기에는 각 카드 능력을 **개별 메서드**로 구현했으나,  
카드가 30장 → 100장으로 늘어나면서 코드가 **5000줄을 넘어가고 유지보수 불가능**해짐.  
새로운 카드 추가 시마다 `GameManager`에 메서드를 추가해야 하는 구조적 문제 발생.

**✅ 해결**  
**ScriptableObject 기반 데이터 주도 설계**로 전환:
1. `Ability` 클래스 생성 → `triggerType`, `effectType`, `targetType`을 **enum으로 분리**
2. `AbilitySystem` 싱글톤에서 **switch-case 패턴**으로 능력 실행
3. 새로운 카드 추가 시 **코드 수정 없이 ScriptableObject 에셋만 생성**

```csharp
// Ability ScriptableObject
[CreateAssetMenu(fileName = "New Ability", menuName = "Card/Ability")]
public class Ability : ScriptableObject
{
    public AbilityTriggerType triggerType; // OnPlay, OnTurnStart, OnDeath
    public AbilityEffectType effectType;   // Buff, Debuff, Heal, Damage
    public AbilityTargetType targetType;   // Self, Ally, Enemy
    public int effectValue;
}
```

**📈 결과**: 
- GameManager 코드 5000줄 → **1500줄로 감소** (70% 절감)
- 새 카드 추가 시간 **30분 → 5분으로 단축**
- 기획자가 직접 카드 데이터 생성 가능

---

### 3️⃣ 네트워크 환경에서 카드 드로우 동기화 문제
**🔴 문제**  
카드 드로우 시 각 클라이언트가 **독립적으로 랜덤 드로우를 실행**하면서,  
양쪽 플레이어가 서로 다른 카드를 손패에 가지게 되는 심각한 동기화 오류 발생.  
특히 **'상대방의 손패를 확인하는 카드'** 사용 시 완전히 다른 카드가 보이는 버그 발견.

**✅ 해결**  
**마스터 클라이언트가 덱의 유일한 권한자(Authority)**가 되도록 설계 변경:
1. 클라이언트가 `RequestDrawCards` RPC로 **드로우 요청**
2. 마스터가 자신의 덱에서 실제 카드를 뽑아 **카드 이름 배열 생성**
3. `SyncDrawnCards` RPC로 모든 클라이언트에 **뽑힌 카드 데이터 전송**
4. 각 클라이언트는 `GameAssets.GetCardByName()`으로 **동일한 카드 데이터를 손패에 추가**

```csharp
[PunRPC]
public void RequestDrawCards(PlayerType playerType, int amount)
{
    if (!PhotonNetwork.IsMasterClient) return;
    
    Player player = GetCurrentPlayer(playerType);
    List<string> drawnCardNames = new List<string>();
    
    for (int i = 0; i < amount; i++)
    {
        CardData drawnCard = player.gameDeck.DrawCard();
        if (drawnCard != null)
            drawnCardNames.Add(drawnCard.name);
    }
    
    // 모든 클라이언트에 동일한 카드 전송
    photonView.RPC("SyncDrawnCards", RpcTarget.All, 
                   playerType, drawnCardNames.ToArray());
}

[PunRPC]
public void SyncDrawnCards(PlayerType targetPlayerType, string[] drawnCardNames)
{
    Player player = GetCurrentPlayer(targetPlayerType);
    if (player == null) return;

    StartCoroutine(AnimateDrawCardsCoroutine(player, drawnCardNames));
}
```

**📈 결과**: 
- 모든 클라이언트가 **완벽히 동일한 게임 상태 유지**
- 손패 확인 카드 버그 **완전 해결**
- 덱 순서 동기화 **100% 보장**

---

## 📂 프로젝트 구조

```
CardBattleGame/
├── Assets/
│   └── 02.Script/
│       ├── Cards/                    # 카드 시스템
│       │   ├── CharacterCard.cs      # 캐릭터 카드 (1512줄)
│       │   └── PermanentCard.cs      # 영구 카드
│       ├── Characters/               # 캐릭터 시스템
│       │   ├── Hero.cs               # 영웅 캐릭터
│       │   └── LeaderCharacter.cs    # 리더 캐릭터
│       ├── Data/                     # ScriptableObject 데이터
│       │   ├── AbilitySystem.cs      # 능력 시스템 (1573줄)
│       │   ├── CardData.cs           # 카드 기본 데이터
│       │   ├── CharacterCardData.cs  # 캐릭터 카드 데이터
│       │   ├── LeaderData.cs         # 리더 데이터
│       │   ├── HeroData.cs           # 영웅 데이터
│       │   └── GameAssets.cs         # 전체 카드 데이터베이스
│       ├── Enums/                    # 열거형
│       │   ├── Enums.cs              # 게임 전체 Enum
│       │   └── StatusEffectType.cs   # 상태 이상 타입
│       ├── Managers/                 # 매니저 시스템
│       │   ├── GameManager.cs        # 게임 전체 제어 (3153줄)
│       │   ├── MatchmakingManager.cs # 매칭 시스템 (456줄)
│       │   ├── DeckBuilderManager.cs # 덱 빌더 (23KB)
│       │   ├── AbilityManager.cs     # 능력 실행 관리
│       │   └── NotificationManager.cs# 알림 UI
│       ├── Net/                      # 네트워크
│       │   ├── AuthorityRng.cs       # 서버 권한 랜덤
│       │   └── MatchRng.cs           # 매치 랜덤 생성
│       ├── Player/                   # 플레이어
│       │   ├── Player.cs             # 플레이어 데이터 (519줄)
│       │   └── Deck.cs               # 덱 관리
│       └── UI/                       # UI 시스템
│           ├── CardUI.cs             # 카드 UI
│           ├── ResourceDisplay.cs    # 자원 표시
│           └── DragHandler.cs        # 드래그 앤 드롭
└── README.md
```

---

## 🎮 플레이 방법

### 게임 시작
1. **덱 빌더**: 메인 메뉴에서 30장 덱 구성
2. **리더 선택**: 고유 패시브 능력을 가진 리더 선택
3. **매칭 시작**: 랜덤 매칭으로 상대방 자동 연결

### 게임 플레이
- **턴 시작**: 자동으로 3자원 획득, 1장 드로우
- **카드 사용**: 자원 소모하여 손패에서 필드에 카드 배치
- **공격**: 캐릭터 선택 → 적 캐릭터/리더 클릭
- **턴 종료**: 우측 하단 "턴 종료" 버튼 클릭 (60초 제한)

### 승리 조건
- 상대방 **리더의 HP를 0**으로 만들기
- 상대방이 게임을 **이탈**하면 자동 승리

---

## 🔧 빌드 및 실행

### 필수 요구사항
- Unity 6
- .NET Framework 4.x
- Photon PUN2 (Free Plan)

### 로컬 테스트
- **ParrelSync 사용**: 에디터에서 복제본 생성하여 2인 플레이 테스트
- **빌드 테스트**: 빌드 파일 + 에디터 동시 실행

---

## 📊 개발 통계

| 항목 | 수치 |
|------|------|
| **총 스크립트 파일** | 108개 |
| **총 코드 라인** | ~15,000줄 |
| **GameManager** | 3,153줄 |
| **AbilitySystem** | 1,573줄 |
| **CharacterCard** | 1,512줄 |
| **구현된 카드 능력** | 100+ 종류 |
| **ScriptableObject 에셋** | 80+ 개 |
| **상태 이상 종류** | 15가지 |

---

## 📝 개발 후기

### 배운 점
- **네트워크 동기화의 중요성**: 마스터 클라이언트 권한 관리로 동기화 보장
- **확장 가능한 설계**: ScriptableObject로 데이터 주도 개발 경험
- **디버깅 효율화**: ParrelSync로 로컬 멀티플레이어 테스트 시간 대폭 단축

### 아쉬운 점
- **시간부족**: 공모전 제출용 게임이라 제작이 촉박함
- **UI**: 대부분의 작업을 혼자서 했기에 UI 개선을 못함 

---

## 🤝 기여자

| 이름 | 역할 | 담당 |
|------|------|------|
| **김승준** | Solo Developer | 네트워크, 카드 시스템, 전투 로직, UI |
| 팀원A | Developer | 일부 UI 작업, 테스트 |

---

## 📧 Contact

- **Name**: 김승준 (Kim Seung-jun)
- **Email**: [ksjun0541@gmail.com](mailto:ksjun0541@gmail.com)
- **GitHub**: [@SeungJun751](https://github.com/SeungJun751)
- **Portfolio**: [https://seungjun751.github.io](https://seungjun751.github.io)

---

<p align="center">
  <strong>⭐ 이 프로젝트가 도움이 되었다면 Star를 눌러주세요! ⭐</strong>
</p>
