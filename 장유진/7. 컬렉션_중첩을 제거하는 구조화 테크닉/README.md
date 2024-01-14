# 7장 컬렉션: 중첩을 제거하는 구조화 테크닉

## 1. 이미 존재하는 기능을 다시 구현하지 말기

다음은 게임에서 소지품 중에 '감옥 열쇠'가 있는지 확인하는 자바 코드입니다.
```java
boolean hasPrisonKey = false;
// items는 List<Item> 자료형
for (Item each : items) {
    if (each.name.equals("감옥 열쇠")) {
        hasPrisonKey = true;
        break;
    }
}
```
같은 기능을 다음처럼 구현할 수 있습니다.
```java
boolean hasPrisonKey = items.stream().anyMatch(
    item -> item.name.equals("감옥 열쇠");
)
```
이처럼 anyMatch 메서드를 알고 있으면, 복잡한 로직을 직접 구현하지 않아도 됩니다.<br>
반대로 모르면 로직을 직접 구현해야 하므로, 코드가 복잡해지고 구현 실수로 버그가 발생항 수 있습니다.

따라서 for문으로 사용해 컬렉션을 직접 조작하고 있다면, 잠시 멈추고 표준 라이브러리에 같은 기능을 하는 메서드가 있는지 확인해보세요.

`바퀴의 재발명: 이미 널리 사용되고 있는 기술과 해결법이 존재하는데도, 이를 전혀 모르거나 의도적으로 무시하고 비슷한 것을 새로 만들어내는 것. 위의 anyMatch 예시`

`네모난 바퀴의 재발명: 이미 존재하는 것보다 좋지 못한 결과물을 만들어 내는 것`

## 2. 반복 처리 내부의 조건 분기 중첩
RPG 게임에서 멤버 전원의 상태를 확인하고 중독된 경우 히트포인트를 감소하는 로직을 설계한다고 합시다.

아래의 코드는 반복문 내부에 여러 조건이 중첩되어 가독성이 떨어지는 코드입니다.
```java
for (Member member : members) {
    if (0 < member.hitPoint) {
        if (member.containsState(StateType.poison)) {
            member.hitpoint -= 10;
            if (member.hitPoint <= 0) {
                member.hitPoint = 0;
                member.addState(StateType.dead);
                member.removeState(StateType.poison);
            }
        }
    }
}
```

### 2.1 조기 컨티뉴로 조건 분기 중첩 제거하기
조기 리턴을 응용한 **조기 컨티뉴**로 해결할 수 있습니다.

'조건을 만족하지 않는 경우, continue로 다음 반복으로 넘어간다'라는 기법입니다.
```java
for (Member member : members) {
    if (0 == member.hitPoint) continue;
    if (!member.containsState(StateType.poison)) continue;
    
    member.hitpoint -= 10;
    if (member.hitPoint > 0) continue;
    
    member.hitPoint = 0;
    member.addState(StateType.dead);
    member.removeState(StateType.poison);
}
```
이렇게 작성하면 3겹으로 중첩된 if 조건문이 모두 제거되어 가독성이 좋아집니다.

또한 어디까지 실행되는지 continue로 쉽게 확인할 수 있으므로 이해도도 올라갑니다.

### 2.2 조기 브레이크로 중첩 제거하기
continue 이외에도 break가 있습니다.

조기 컨티뉴와 마찬가지로 조기 브레이크를 사용하면 로직이 단순해지는 경우가 많습니다.

반복문 처리 내부에서 if 조건문이 중첩될 경우, 조기 컨티뉴와 조기 브레이크를 활용할 수 있는지 검토해 보기 바랍니다.

## 3. 응집도가 낮은 컬렉션 처리
컬렉션 처리도 응집도가 낮아지기 쉽습니다.

예를 들어 다음과 같이 파티(멤버들)에 관련된 로직이 필드 맵을 관리하는 클래스와 특별 이벤트 클래스, 전투 클래스 등에 있다고 칩시다.

이 경우 코드가 중복되고 파티 컬렉션에 관련된 기능이 사방에 흩어져 있는 것이라 응집도가 낮아진 상태입니다.

```java
class FieldManager {
    // 멤버 추가
   void addMember(List<Member> members, Member newMember) {
      if (members.stream().anyMatch(member -> member.id == newMember.id)) {
          throw new RuntimeException("이미 존재하는 멤버입니다.");
      }
      if (members.size() == MAX_MEMBER_COUNT) {
          throw new RuntimeException("이 이상 멤버를 추가할 수 없습니다.");
      }
      members.add(newMember);
   }
   
   // 파티 멤버가 1명이라도 살아있는지 확인
   boolean partyIsAlive(List<Member> members) {
       return members.stream().anyMatch(member -> member.isAlive());
   } 
}

class SpecialEventMamager {
    // 멤버 추가
    void addMember(List<Member> members, Member member) {
        members.add(member);
    }
}

class BattleMamager {
    // 파티 멤버가 1명이라도 살아있는지 확인
    boolean membersAreAlive(List<Member> members) {
        boolean result = false;
        for (Member each : members) {
            if (each.isAlive()) {
                result = true;
                break;
            }
        }
        return result;
    }
}
```
### 3.1 컬렉션 처리를 캡슐화하기
컬렉션과 관련된 응집도가 낮아지는 문제는 **일급 컬렉션 패턴**을 사용해 해결할 수 있습니다.

**일급 컬렉션**이란 클래스의 설계 원리와 같이 컬렉션과 관련된 로직을 캡슐화하는 디자인 패턴입니다.

- 컬렉션 자료형의 인스턴스 변수
- 컬렉션 자료형의 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정사적으로 조작하는 메서드

다음은 위의 중복된 코드와 낮아진 응집도를 해결하기 위해,<br>
컬렉션과 컬렉션을 조작하는 로직을 Party라는 한 클래스에 응집한 코드입니다.
```java
class Party {
    static final int MAX_MEMBER_COUNT = 4;
    private final List<Member> members;

    Party() {
        members = new ArrayList<Member>;
    }

    private Party(List<Member> members) {
        this.members = menbers;
    }

    /**
     * 멤버 추가하기
     * @paran newMenber 추가하고 싶은 멤버
     * @return 멤버를 추가한 파티
     */
    Party add(final Member newMember) {
        if (exists(newMember)) {
            throw new RuntimeException("이미 파티에 참가되어 있습니다.");
        }
        if (isFulll()) {
            throw new RuntimeException("이 이상 멤버를 추가할 수 없습니다.");
        }
        
        // 부수 효과를 막기 위해 새로운 리스트를 생성
        final List<Member> adding = new ArrayList<>(members);
        adding.add(newMember);
        return new Party(adding);
    }

    /** @return 파티 멤버가 1명이라도 살아 있으면 true를 리턴 */
    boolean isAlive() {
        return members.stream().anyMatch(each -> each.isAlive());
    }

   /**
    * @param member 파티에 소속되어 있는지 확인하고 싶은 멤버
    * @return 이미 소속되어 있는 경우 true를 리턴
    */
   boolean exists(final Member member) {
       return members.stream().anyMatch(each -> each.id == member.id);
   }
   
   /** @return 파티 인원이 최대일 경우 true를 리턴*/
   boolean isFull() {
       return members.size() == MAX_MEMBER_COUNT;
   }
}
```

### 3.2 외부로 전달할 때 컬렉션의 변경 막기
외부로 전달할 때는 컬렉션의 요소를 변경하지 못하게 막아 두는 것이 좋습니다.

이때는 unmodifiableList 메서드를 사용합니다.
```java
class Party {
    // 생략
   
   /** @return 멤버 리스트(다만 요소를 외부에서 변경할 수 없습니다.) */
   List<Member> members() {
       return Members.unmodifiableList();
   }
}
```





