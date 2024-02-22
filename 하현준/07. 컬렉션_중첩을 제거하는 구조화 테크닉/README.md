## 1. 이미 존재하는 기능을 다시 구현하지 말기

이미 구현되어 있는 고차함수(map, filter, findIndex 등)을 사용해서 복잡해지는 로직을 직접 구현하지 않고도 가독성을 높일 수 있다.

```tsx
// bad
let hasPrisonKey = false;

for (let i = 0; i <= items.length; i++) {
  if (items[i].name === "감옥 열쇠") {
    hasPrisonKey = true;
    break;
  }
}

// good
const hasPrisonKey = items.findIndex((item) => item.name === "감옥 열쇠") !== -1;
// some 함수로도 가능
```

## 2. 반복 처리 내부의 조건 분기 중첩

### 조건 커티뉴로 조건 분기 중첩 제거하기

```tsx
for (let i = 0; i <= members.length; i++) {
	const member = members[i];
  if (members.hitPint === 0) continue;
	if (!members.containsState(StateType.poison)) continue;

	member.hitPoint -= 10;

	if (member.hitPoint > 0) continue;

	member.hitPoint = 0;
	member.addState(StateType.dead);
	member.removeState(StateType.poison);
	}
}
```

중첩된 if문을 사용하는 대신에 continue로 건너뛰게 해준다.

### 조건 브레이크로 중첩 제거하기

```tsx
let totalDamage = 0;
for (let i = 0; i <= members.length; i++) {
	const member = members[i];
  if (!member.hasTeamAttackSucceeded()) break;
	let damage = Math.floor(member.attack() * 1.1);

	if (damage < 30) break;
	totalDamage += damage;
	}
}
```

마찬가지로 중첩된 if문을 사용하는 대신에 break로 가독성을 높인다.

## 3. 응집도가 낮은 컬렉션 처리

_프론트의 자료형에는 컬렉션이 없기에, 배열, set 으로 보면 될 것 같다._

```tsx
class FieldManager {
  // 멤버를 추가한다.
  addMember(members: Array<Member>, newMember: Member) {
    if (members.findIndex((member) => member.id === newMember.id) !== -1) {
      throw new Error("이미 존재하는 멤버입니다.");
    }
    if (members.size() === MAX_MEMBER_COUNT) {
      throw new Error("이 이상 멤버를 추가할 수 없습니다.");
    }
    members.push(newMember);
  }

  // 파티 멤버가 1명이라도 존재하면 true를 리턴
  partyIsAlive(members: Array<Member>) {
    return members.filter((member) => member.isAlive()).length > 0;
  }
}
```

게임에서 필드 맵을 관리하는 클래스라고 할 때, 게임 내에서 멤버를 추가하는 시점이 있을 수 도 있다.

```tsx
class SpecialEventManager {
  addMember(members: Array<Member>, member: Member) {
    members.push(member);
  }
}
```

멤버를 추가하기 위한 메서드가 하나 더 생성되어 중복된 코드가 만들어졌다. 즉, members관련된 로직을 처리하기 위해 코드가 여기저기에 구현되어 **응집도가 낮아**진 것이다.

### 컬렉션 처리를 캡슐화하기

일급 컬렉션 패턴을 사용해 해결할 수 있고, 일급 컬렉션이란 컬렉션과 관련된 로직을 캡슐화 하는 디자인 패턴이다.

[클래스에 있어야 할 두가지](https://www.notion.so/3-27907c35dd8643829d7887a532d3cdf2?pvs=21)를 기본적으로 따라야 한다.

일급 컬렉션은

- 컬렉션 자료형의 인스턴스 변수
- 컬렉션 자료형의 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메서드

member 컬렉션을 인스턴스 변수로 가지는 클래스를 다음과 같이 설계할 수 있다.

```tsx
type Member = {
  id: number;
  isAlive: () => boolean;
};

class Party {
  static readonly MAX_MEMBER_COUNT = 4;
  private readonly members: Array<Member>;

  constructor(members?: Array<Member>) {
    if (members) {
      this.members = members;
    } else this.members = new Array<Member>();
  }

  add(newMember: Member) {
    if (this.exists(newMember)) {
      throw new Error("이미 파티에 참가되어 있습니다.");
    }
    if (this.isFull()) {
      throw new Error("이 이상 멤버를 추가할 수 없습니다.");
    }

    const adding = [...this.members];
    adding.push(newMember);

    return new Party(adding);
  }

  isAlive() {
    return this.members.some((member) => member.isAlive());
  }

  exists(member: Member) {
    return this.members.some((member) => member.id === member.id);
  }

  isFull() {
    return this.members.length === Party.MAX_MEMBER_COUNT;
  }
}
```

컬렉션을 조작하고 확인하는 로직을 한곳으로 모아 응집도를 높였다.

### 외부로 전달할 때 컬렉션의 변경 막기

외부에서 멤버 인스턴스를 참조하려고 할 때 그대로 외부로 전달하면 추가와 제거가 가능해진다.

이를 막기 위해 unmodifiableList 메서드를 사용한다. (자바스크립트에는 존재하지 않는 메서드이므로 다른 방법으로 설명하겠다.)

```tsx
class Party {
  // 생략
  members() {
    return this.members;
  }
}
```

여러가지 방법이 있어보이는데

방법1. 일단 원본을 복사해서 리턴하기

```tsx
class Party {
  // 생략
  members() {
    return [...this.members];
  }
}
```

방법2. getter와 setter 사용

```tsx
class Party {
  // 생략
  get getMembers() {
    return [...this.members];
  }
  set setMembers() {}
}
```

위 방법은 모두 원본을 복사해서 넘기기에 원본은 변하지 않지만 넘어가서 사용할때는 변경이 가능하다.

**방법3. `[Object.freeze](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)` 사용하기**

```tsx
class Party {
  // 생략
  getMembers() {
    return Object.freeze(this.members);
  }
}
```

Object.freeze를 쓰면 객체를 조작하는게 불가능해지기에 조작 불가능하게 된다.
