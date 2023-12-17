# 5장 응집도: 흩어져 있는 것들

## 1. static 메서드 오용

### **static 메서드는 인스턴스 변수를 사용할 수 없음**

static 메서드로 만든 시점에 이미 데이터와 데이터를 조작하는 로직 사이에 괴리가 생겨 응집도가 낮아진다.

**인스턴스 메서드 척하는 static 메서드 주의하기**

static 키워드가 붙어 있지 않지만 static메서드와 동일한 역할을 하는 메서드를 주의해라

```tsx
class PaymentManager {
	private discountRate: number;
	
	add(moneyAmount1: number, moneyAmount1: number) {
		return moneyAmount1 + moneyAmount1;
	}
}
```

`add` 메서드는 static을 붙여도 아무 문제없이 동작한다.

### **어떠한 상황에서 Static메서드를 사용하면 좋을까?**

응집도의 영향을 받지 않는 경우, static 메서드를 사용해도 좋다.

## 2. 초기화 로직 분산

```tsx
class GiftPoint {
	private static readonly MIN_POINT = 0;
	private readonly value: number;

	constructor(point: number) {
		if(point < GiftPoint.MIN_POINT) throw new Error("포인트를 0 이상 입력해야 합니다.");
		this.value = point;
	}

	add(other: GiftPoint): GiftPoint {
		return new GiftPoint(this.value + other.value);
	}
	
	isEnough(point: ConsumptionPoint): boolean {
		return point.value <= this.value;
	}

	consume(point: ConsumptionPoint): GiftPoint {
		if(this.isEnough(point)) throw new Error("포인트가 부족합니다.");
		return new GiftPoint(this.value - point.value);	
	}
}
```

```tsx
const standardMembershipPoint = new GiftPoint(3000);
const premiumMembershipPoint = new GiftPoint(10000);
```

위 코드에서는 회원가입 포인트를 변경하고 싶다면 전체 코드를 확인해야 한다.

### **private 생성자 + 팩토리 메서드를 사용해 목적에 따라 초기화한다.**

```tsx
class GiftPoint {
	private static readonly MIN_POINT = 0;
	private static readonly STANDARD_MEMBERSHIP_POINT = 3000;
	private static readonly PREMIUM_MEMBERSHIP_POINT = 10000;
	private value: number;

	constructor(point: number) {
		if(point < GiftPoint.MIN_POINT) throw new Error("포인트를 0 이상 입력해야 합니다.");
		this.value = point;
	}
	
	static forStandardMembership() {
		return new GiftPoint(GiftPoint.STANDARD_MEMBERSHIP_POINT);
	}

	static forPremiumMembership() {
		return new GiftPoint(GiftPoint.PREMIUM_MEMBERSHIP_POINT);
	}

	// ...
}
```

표준 회원을 위한 `forStandardMembership` ,프리미엄 회원을 위한 `forPremiumMembership` **팩토리 메서드**를 만들어 응집도를 높인다. 

다른 클래스에서 선언했던 포인트 선언식을 찾을 필요 없이 하나의 클래스에서 관련된 로직을 수행할 수 있어진다.

**생성 로직이 너무 많아지면 팩토리 클래스를 고려해보자**

위와 같은 로직들이 매우 많아져 클래스의  일이 불분명해지면 팩토리 클래스로 분리하자

## 3. 범용 처리 클래스(Common/Util)

범용 처리를 위한 클래스로 `static`과 마찬가지로 응집도가 낮은 구조를 만들 수 있다.

**너무 많은 로직이 한 클래스에 모이는 문제**

```tsx
class Common {
	// 세금 포함 금액 계산하기
	static calcAmountIncludingTax(amountExcludingTax: BigInteger, taxRate: BigInteger): BigInteger {}
	
	// 사용자가 이미 탈퇴했다면 true
	static hasResigned(user: User): boolean { }
	
	// 상품 주문하기
	static createOrder(product: Product): void { } 

	// 유효한 전화번호라면 true
	static IsValidPhoneNumber(phoneNumber: string): boolean { }
}
```

관련 없는 로직이 `Common` 클래스에 static메서드로 모여있어서 응집도가 낮아진 구조가 된다. 

범용이라는 의미에 치우쳐 한곳에 몰아넣어 원인이 되었다.

### **객체 지향 설계의 기본으로 돌아가기**

꼭 필요한 경우에만 범용 처리 클래스를 만드는 것이 좋음

```tsx
class AmmountIncludingTax {
	private readonly value: BigInteger;

	constructor(amountExcludingTax: AmmountIncludingTax, taxRate: TaxRate) {
		this.value = amountExcludingTax.value.multiply(taxRate.value);
	}
}
```

### **횡단 관심사**

로그 출력, 오류 확인은 모든 애플리케이션의 동작에 필요한 기능이다. 횡단 관심사에 해당하는 기능은 범용 코드로 만들어도 괜찮다.

횡단관심사의 대표적인 예시

- 로그 출력
- 오류 확인
- 디버깅
- 예외 처리
- 캐시
- 동기화
- 분산 처리

## 4. 결과를 리턴하는 데 매개변수 사용하지 않기

```tsx
class ActorManager {
	shift(location: Location, shiftX: number, shiftY: number) {
		location.x += shiftX;
		location.y += shiftY;
	}
}
```

출력으로 사용되는 매개변수를 출력 매개변수(`location` )라고 부른다. 즉, 데이터와 로직이 각자 다른 클래스에 있어 응집도가 낮아진다.

```tsx
class Location {
	private readonly x: number;
	private readonly y: number;

	constructor(x: number; y: number) {
		this.x = x;
		this.y = y;
	}

	shift(shiftX: number, shiftY: number): Location {
		const nextX = this.x + shiftX;
		const nextY = this.y + shiftY;

		return new Location(nextX, nextY);
	}
}
```

‘데이터’와 ‘데이터를 조작하는 논리’ 를 하나의 클래스에 배치해서 응집도를 높인다.

## 5. 매개변수가 너무 많은 경우

매개변수가 너무 많은 메서드는 응집도가 낮아지기 쉽다.

```tsx
/*
* 매직포인트 회복하기
* currentMaxMagicPoint 현재 매직포인트 잔량
* originalMaxMagicPoint 원래 매직포인트 최댓값
* maxMagicPointIncrements 장비로 증가하는 매직포인트 최댓값 증가량
* recoveryAmount 회복량
* @return 회복 후의 매직포인트 잔량
*/

recoverMagicPoint(currentMaxMagicPoint: number, originalMaxMagicPoint: number, maxMagicPointIncrements: Array<number>, recoveryAmount: number): number [
	let currentMaxMagicPoint = originalMaxMagicPoint;

	maxMagicPointIncrements.forEach((each) => {
			currentMaxMagicPoint += each;
	});
	return Math.min(currentMagicPoint + recoveryAmount, currentMagicPoint);
}
```

매개변수를 많이 받는다는 것 

→ 함수에서 많은 기능을 수행하고 싶다는것 

→ 처리할게 많아지면 로직이 복잡해지거나 중복코드가 발생

### **기본 자료형에 대한 집착**

매개변수와 리턴 값에 모두 기본 자료형만 쓰고 이를 남용하는 현상을 기본 자료형 집착(primitive obsession) 이라고 한다.

```tsx
// Bad
class Common {
	discountRate(regularPrice: number, discountRate: number) {
		if(regularPrice < 0 || discountRate < 0) throw new Error();	
	}
}

// Good

class RegularPrice {
	private readonly amount: number;
	
	constructor(amount: number) {
		if(amount < 0) {
			throw new Error();
		}	
		this.amount = amount;
	}
}

class DiscountedPrice {
	private readonly amount: number;

	constructor(regularPrice: RegularPrice, discountRate: DiscountRate) {
		// regularPrice와 discountRate를 사용해서 계산
	}
}
```

관련 있는 로직을 각각의 클래스로 분리해 응집도를 높인다.

### 의미 있는 단위는 모두 클래스로 만들기

아까 처음에 했던 MagicPoint를 개선해보자

```tsx
class MagicPoint {
	private currentAmount: number;
	private originalMaxAmount: number;
	private readonly maxIncrements: Array<number>

	current(): number {
		return this.currentAmount;
	}
	
	max(): number {
		let amount = this.originalMaxAmount;
		maxIncrements.forEach((each) => {
			amount += each;
		});
		return amount;
	}
	
	consume(consumeAmount: number) {
		// ...
	}
}
```

매개변수가 많다면 인스턴스 변수를 갖는 클래스로 만들자

## 6. 메서드 체인

`.` 으로 여러 메서드를 연걸해서 리턴 값의 요소에 차례 차례 접근하는 방법을 **메서드 체인**이라고 한다.

```tsx
// BAD
equipArmor(memberId: number, newArmor: Armor) {
	if(party.members[memberId].equipments.cancahge) {
		party.members[memberId].equipments.armor = newArmor;
	}
}
```

`members` 의 사양이 조금이라도 변경이 된다면 모든 코드들 확인하고 수정해야 할 것이다.

### 묻지 말고 명령하기

**다른 객체의 내부 상태를 기반으로 판단하거나 제어하려고 하지 말고, 메서드로 명령해서 객체가 알아서 판단하고 제어하도록 설계하라** 라는 의미다.

```tsx
class Equipments {
	private canChange: boolean;
	private head: Equipment;
	private armor: Equipment;
	private arm: Equipment;
	
	// 새로운 갑옷 장비하기
	equipArmor(newArmor: Equipment) {
		if(this.canChange) this.armor = newArmor;
	}

	// 전체 장비 해제
	deactivateAll() {
		this.head = Equipment.EMPTY;
		this.armor = Equipment.EMPTY;
		this.arm = Equipment.EMPTY; 
	}
}
```

방어구의 탈착의 로직을 `Equipments` 로 응집시켜 요구사항 대응이 편해졌다.

> **메소드 체이닝을 보며 다른 해결법?**
> 

위와 같이 메서드 체이닝이 아주 복잡하게 얽혀있다면 클래스로 만들기보다 프론트엔드에서는 
보통 구조분해를 사용하는 것 같은데 클래스로 과연 구현이 될지 생각이 든다.

```tsx
// 가독성을 높이기(?)
equipArmor(equipments: Equipments, newArmor: Armor) {
	const { canChange, armor } = equipments
	if(cancahge) armor = newArmor;
}
```

위처럼 매개변수로 메소드 체이닝된 모든걸 전달하는 것이 아닌 일부만 전달하는 것은 어떨지 생각이 들었다.