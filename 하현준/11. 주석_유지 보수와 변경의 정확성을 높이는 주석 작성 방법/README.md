## 1. 내용이 낡은 주석

게임에서 적의 공격을 받아 중독 상태가 되거나 할 때, 멤버의 얼굴이 힘든 표정으로 변하는 가상의 메서드이다.

```tsx
// 중독, 마비 상태에서 멤버의 표정을 변화시킴
if (member.isPainful()) {
	face.changeToPainful();
}
```

어떤 상태일 때 표정을 변화시키는지 친절하게 설명해주는 주석 같지만 개발이 진행되면서 주석의 설명이 코드와 맞지 않을 수도 있다.

```tsx
isPainful() {
	if(this.states.includes(StateType.poison) || 
		 this.states.includes(StateType.paralyzed) ||
		 this.states.includes(StateType.fear)) {
		return true;
  }
	return false
}
```

코드에서는 ‘공포’일 때에도 true를 리턴하지만 빠져있다. 주석은 거짓말을 할 가능성이 높기에 주석이 낡아버리지 않게, 구현을 변경할 때 주석도 함께 변경하는 것이 좋다.

### 주석은 실제 코드가 아님을 이해하기

프로그래밍에서 클래스와 메서드의 이름이나 주석의 설명도 실제 코드가 아니므로, 실제 내용을 100% 전달할 수는 없다.

### 로직의 동작을 설명하는 주석은 낡기 쉽다.

코드의 동작을 그대로 설명하는 주석은 코드를 변경할 때마다 주석도 변경해야 할 것이다.

## 2. 주석 때문에 이름을 대충 짓는 예

```tsx
// 수면, 마비, 혼란, 석화, 사망 이외의 상황에서 행동 가능
isNotSleepingAndIsNotParalyzedAndIsNotConfusedAndIsNotStoneAndIsNotDead(): boolean {
	if(this.states.includes(StateType.sleaping) || 
		 this.states.includes(StateType.paralyzed) ||
		 this.states.includes(StateType.confused) ||
		 this.states.includes(StateType.stone) || 
		 this.states.includes(StateType.dead)) {
		return false;
  }
	return true
}
```

메서드의 이름만으로는 어떤 의도를 가진 메서드인지 전달하기 어렵다. 그래서 주석을 달게 되는데 위의 주석은 행동이 추가되거나 로직이 변경되었을때 주석도 함께 변경해야한다. 

그러므로 **메서드의 이름 자체를 수정**하는 것이 좋다.

```tsx
canAct(): boolean {
	// 행동 불능 사양이 변경되는 경우
	// 다음 로직을 변경합니다.
	if(this.states.includes(StateType.sleaping) || 
		 this.states.includes(StateType.paralyzed) ||
		 this.states.includes(StateType.confused) ||
		 this.states.includes(StateType.stone) || 
		 this.states.includes(StateType.dead)) {
		return false;
  }
	return true
}
```

## 3. 의도와 사양 변경 시 주의 사항을 읽는 이에게 전달하기

코드는 기본적으로 유지 보수할 때와 사양을 변경할 때 읽힌다. 

- 코드 유지보수 시 읽는 사람이 주의할 사항: ‘이 로직은 어떤 의도를 갖고 움직이는가’
- 사양을 변경할 때 읽는 사람이 주의할 사항: ‘안전하게 변경하려면 무엇을 주의해야 하는가’

```tsx
// 고통받는 상태일 때 true를 리턴
isPainful() {
	// 이후 사양 변경으로 표정 변화를
  // 일으키는 상태를 추가할 경우
	// 이 메서드에 로직을 추가합니다.
	if(this.states.includes(StateType.poison) || 
		 this.states.includes(StateType.paralyzed) ||
		 this.states.includes(StateType.fear)) {
		return true;
  }
	return false
}
```

## 4. 주석 규칙 정리

| 규칙 | 이유 |
| --- | --- |
| 로직을 변경할 때는 반드시 주석도 함께 변경해야 함 | 주석을 제대로 변경하지 않으면, 실제 로직과 달라져 주석을 읽는 사람에게 혼란을 줌 |
| 로직의 내용을 단순하게 설명하기만 하는 주석은 달지 않음. | 실질적으로 가독성을 높이지 않고, 주석 유지 보수가 힘듦. 결과적으로 내용이 낡은 주석이 될 가능성이 높음. |
| 가독성이 나쁜 로직에 설명을 추가하는 주석은 달지 않음. 대신에 로직의 가독성을 높여야함. | 주석 유지 보수가 힘들고, 갱신되지 않아 낡은 주석이 될 가능성이 높음 |
| 로직의 의도와 사양을 변경할 때 주의할 점을 주석으로 달아야함 | 유지 보수와 사양 변경에 도움이 됨. |

## 5. 문서 주석

문서 주석이란 특정 형식에 맞춰 주석을 작성하면, API 문서를 생성해 주거나 코드 에디터에서 주석의 내용을 팝업으로 표시해 주는 기능이다.

*책에서는 Javadoc을 예시로 들었지만 Jsdoc으로 설명하겠다.*

```tsx
class Money {
	

	/**
	* 금액을 추가합니다.
	* 
	* @params {Money} other 추가 금액
	* @returns {boolean} 추가 후의 금액
	* @throws {Error} 통화 단위가 다르면 예외 발생
	*/
	add(other: Money) {
		if(!currency.equals(other.currency)) {
			throw new Error("통화 단위가 다릅니다.");
		}
		let added = amount + other.amount;
		return new Money(added, currency);
	}
}
```