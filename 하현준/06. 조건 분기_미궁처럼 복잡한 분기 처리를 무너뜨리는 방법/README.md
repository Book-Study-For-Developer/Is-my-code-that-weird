## 1. 조건 분기가 중첩되어 낮아지는 가독성

### 조기 리턴으로 중첩 제거하기

조기 리턴(early return)은 조건을 만족하지 않는 경우 곧바로 리턴하는 방법이다.

**조건 로직**과 **실행 로직**을 분리하여 가독성을 향상시킬 수 있다.

```tsx
// 조건 로직
if (member.hitPoint <= 0) return;
if (!member.canAct()) return;
if (member.magicPoin < magic.costMagicPoint) return;

// 실행 로직
member.consumeMagicPoint(magic.costMagicPoint);
member.chant(magic);

```

### 가독성을 낮추는 else 구문도 조기 리턴으로 해결하기

if 와 else로 구성된 코드인데 아래 코드는 else도 필요없이 구현이 가능하다.

개선 전 코드

```tsx
const hitPointRage: number = member.hitPoint / member.maxHitPoint;

let currentHealthCondition: HealthCondition;

if (hitPointRate === 0) {
   currentHealthCondition = HealthCondition.dead;
} else if (hitPointRate < 0.3) {
   currentHealthCondition = HealthCondition.danger;
} else if (hitPointRate < 0.5) {
   currentHealthCondition = HealthCondition.caution;
} else {
   currentHealthCondition = HealthCondition.fine;
}

return currentHealthCondition;
```

개선 후 코드

```tsx
const hitPointRage: number = member.hitPoint / member.maxHitPoint;

if (hitPointRate === 0) return HealthCondition.dead;
if (hitPointRate < 0.3) return HealthCondition.danger;
if (hitPointRate < 0.5) return HealthCondition.caution;

return HealthCondition.fine;
```

## 2. switch 조건문 중복

### switch 조건문을 사용해서 코드 작성하기

```tsx
enum MagicType {
   fire,
   lighting
}

class MagicManager {
	getName(magicType: MagicType): string {
		let name = '';
		
		switch (magicType) {
      case MagicType.fire:
				name = '파이어';
				break;
			case MagicType.lighting:
				name = '라이트닝';
				break;
			default:
				break;
		}
		return name;
	}
}
```

### 같은 형태의 switch 조건문이 여러 개 사용되기 시작

예를 들어 공격 형태에 따라 공격력, 소비량이 달라질 경우 코드를 작성했다고 하자

```tsx
function costMagicPoint(magicType: MagicType, member: Member): number {
  	let magicPoint = 0;
		
		switch (magicType) {
      case MagicType.fire:
				magicPoint = 2;
				break;
			case MagicType.lighting:
				magicPoint = 5 + parseInt(member.level * 0.2);
				break;
			default:
				break;
		}
		return magicPoint;
}

function attackPower(magicType: MagicType, member: Member): number {
  	let attackPower = 0;
		
		switch (magicType) {
      case MagicType.fire:
				attackPower = 20 + parseInt(member.level * 0.5);
				break;
			case MagicType.lighting:
				attackPower = 50 + parseInt(member.aglity * 1.5);
				break;
			default:
				break;
		}
		return attackPower;
}
```

이렇게 되면 switch 구문이 3개나 사용이 되었고 지금은 문제가 없을 것 같지만 추후 요구사항이 추가되거나 변경되면 **악마**가 생겨난다.

### 요구 사항 변경 시 수정 누락(case 구문 추가 누락)

새로운 마법인 ‘헬파이어’ 가 추가해야한다는 요구사항이 되었을 때 이제 그럼 switch문에 ‘헬파이어’ 에 해당하는 case를 추가해야 한다. 그러나 한곳이 아닌 세곳 모두에 case를 추가해야 한다.

여기에 더해서 새로운 요구사항인 ‘테크니컬 포인트’ 를 추가해달라고 했다. 그럼 새로운 switch 구문을 하나더 추가해줘야 한다. 그렇게 되면 총 4곳의 switch문을 수정해야 오류가 없이 배포가 가능하다.

**즉, 새로운 요구사항이 생기고 유지보수가 필요할때마다 점점 힘들어지는 것이다.**

*책에는 설명이 없지만 저 마법을 생성하는 함수를 유틸로 빼고 타입을 체크한다면 괜찮지 않을까?* 

### 조건 분기 모으기

이를 해결하기 위해서는 **단일 책임 선택의 원칙**을 생각해봐야 한다.

```tsx
enum MagicType {
   fire,
   lighting,
   hellfire
}

type Member = {
  level: number;
  agility: number;
}

class Magic {
	readonly name: string;
	readonly costMagicPoint: number ;
  readonly attackPower: number;
  readonly costTechnicalPoint: number;

  constructor(magicType: MagicType, member: Member) {
    switch (magicType) {
				case MagicType.fire:
					this.name = '파이어';
					this.costMagicPoint = 2;
					this.attackPower = 20 + Math.floor(member.level * 0.5);
					this.costTechnicalPoint = 0;
					break;
				case MagicType.lighting:
					this.name = '라이트닝';
					this.costMagicPoint = 5 + Math.floor(member.level * 0.2);
					this.attackPower = 50 + Math.floor(member.agility * 1.5);
					this.costTechnicalPoint = 5;
					break;
				case MagicType.hellfire:
					this.name = '헬파이어';
					this.costMagicPoint = 16;
					this.attackPower =  200 + Math.floor(member.level * 0.2);
					this.costTechnicalPoint = 20 + Math.floor(member.level * 1.5);
					break;
				default:
					throw new Error();
		}
  }
}
```

한곳에 switch문으로 관리해서 변경에도 실수를 줄일 수 있다.

### 인터페이스를 switch 조건문 중복에 응용하기

자바스크립트에는 인터페이스라는 개념이 존재하지 않기 때문에 타입스크립트에서 제공해주는 `interface`를 적용하겠다.

```tsx
interface Magic {
  name: () => string;
  attackPower: () => number;
  costMagicPoint: () => number;
  costTechincalPoint: () => number;
}

class Fire implements Magic {
  private readonly member: Member;

	constructor(member: Member) {
    this.member = member
	}
	name(): string {
		return '파이어';	
	}
	costMagicPoint(): number {
		return 2;
	}
	attackPower(): number {
		return 20 + Math.floor(this.member.level * 0.5);	
	}
	costTechincalPoint(): number {
		return 0
	}
}

// hellfire, lightning 도 동일하게 구현
```

다른 마법들도 마찬가지로 Fire와 동일하게 클래스를 작성해주면 된다.

**switch 조건문이 아닌, Map 으로 변경하기**

```tsx
const magics = new Map<MagicType, Magic>();

const fire = new Fire(member);
const lightning = new Lightning(member);
const hellFire = new HellFire(member);

magics.set(MagicType.fire, fire);
magics.set(MagicType.lightning, lightning);
magics.set(MagicType.hellFire, hellFire);
```

마법 처리를 모두 Map을 사용하는 방법으로 변경해보자

```tsx
class Magic {
  // 생략
	magicAttack(magicType: MagicType): void {
	  const usingMagic = magics.get(magicType);
		
		showMagicName(usingMagic);
		consumeMagicPoint(usingMagic);
		consumeTechnicalPoint(usingMagic);
		magicDamage(usingMagic);
	}
	
	showMagicName(magic: Magic): void {
		const name = magic.name;
		// 생략
	}
	
	consumeMagicPoint(magic: Magic): void {
		const costMagicPoint = magic.costMagicPoint();
		// 생략
	}
	
	consumeTechnicalPoint(magic: Magic): void {
		const costTechnicalPoint = magic.costTechnicalPoint();
		// 생략
	}
	
	 magicDamage(magic: Magic): void {
	  const attackPower = magic.attackPower();
		// 생략
	}
}
```

switch 문을 모두 제거하고 한꺼번에 전환이 가능해졌다.

이를 **전략 패턴(strategy pattern)** 이라고 부른다.

**메서드를 구현하지 않으면 오류로 인식하게 만들기**

```tsx
interface Magic {
  name: () => string;
  attackPower: () => number;
  costMagicPoint: () => number;
  costTechincalPoint: () => number;
	magicPower: () => number; // 새로운 함수 추가
}
```

새로운 함수를 추가했으면 Magic을 implement하고 있던 클래스들은 모두 에러를 내뱉기 때문에 실수를 방지할 수 있다.

**값 객체화하기**

인터페이스들의 메소드 들은 원시값의 자료형을 리턴하기에 이를 값 객체로 변경하면 다른값을 전달할 가능성을 줄일 수 있다.

```tsx
// AttackPower, MagicPoint, TechnicalPoint 는 클래스들
// 직접적인 구현은 하지 않음
interface Magic {
  name: () => string;
  attackPower: () => AttackPower;
  costMagicPoint: () => MagicPoint;
  costTechincalPoint: () => TechnicalPoint
}

class Fire implements Magic {
  private readonly member: Member;

	constructor(member: Member) {
    this.member = member
	}
	name(): string {
		return '파이어';	
	}
	costMagicPoint(): number {
		return new MagicPoint(2);
	}
	attackPower(): number {
		const value = 20 + Math.floor(this.member.level * 0.5);	
		return new AttackPower(value);
	}
	costTechincalPoint(): number {
		return new TechnicalPoint(0);
	}
}

```

## 3. 조건 분기 중복과 중첩

인터페이스를 활용하면 다중 중첩된 복잡한 분기를 제거하는데 활용이 가능하다.

골드, 실버 회원을 판별하는 코드가 다음과 같다

```tsx
function isGoldCustomer(history: PurchaseHistory) {
  if (1000000 <= history.totalAmount) {
    if (10 <= history.purchaseFrequencyPerMonth) {
      if (history.returnRate <= 0.001) {
        return true;
      }
    }
  }
  return false
}

function isSilverCustomer(history: PurchaseHistory) {
  if (10 <= history.totalAmount) {
		if (history.returnRate <= 0.001) {
       return true;
    }
  }
  return false;
}
```

딱봐도 로직이 재사용되는 것으로 보인다.

### 정책 패턴으로 조건 집약하기

정책 패턴이란 조건을 부품처럼 만들고, 부품으로 만든 조건을 조합해서 사용하는 패턴이다.

```tsx
type PurchaseHistory ={
  totalAmount: number;
  purchaseFrequencyPerMonth: number;
  returnRate: number;
}
// 먼저 판정 조건을 나타내는 인터페이스를 만든다.
interface ExcellentCustomerRule {
  ok: (history: PurchaseHistory) => boolean;
}

// 골드 회원의 정책을 클래스 형태로 만든다.
class GoldCustomerPurchaseAmountRule implements ExcellentCustomerRule {
  ok(history:PurchaseHistory): boolean {
    return 100000 <= history.totalAmount;
  }
}

class PurchaseFrequencyRule implements ExcellentCustomerRule {
 ok(history:PurchaseHistory): boolean {
    return 10 <= history.purchaseFrequencyPerMonth;
  }
}

class ReturnRateRule implements ExcellentCustomerRule {
 ok(history:PurchaseHistory): boolean {
    return history.returnRate <= 0.001;
  }
}

// 각 정책을 추가하고 정책이 올바른지 확인하는 클래스를 만든다.
class ExcellentCustomerPolicy {
  rules: Set<ExcellentCustomerRule>;

  constructor() {
    this.rules = new Set()
  }

  add(rule: ExcellentCustomerRule) {
    this.rules.add(rule)
  }

  complyWithAll(history: PurchaseHistory){
    let flag = false;
    this.rules.forEach((each) => {
      if(!each.ok(history)) flag = false;
    })
    return flag;
  }
}
```

정책을 사용하기 쉽게 구현했으면 이제 골드 회원에 적용하는 방법이다.

```tsx
const goldCustomerPolicy = new ExcellentCustomerPolicy();
goldCustomerPolicy.add(new GoldCustomerPurchaseAmountRule());
goldCustomerPolicy.add(new PurchaseFrequencyRule());
goldCustomerPolicy.add(new ReturnRateRule());
goldCustomerPolicy.complyWithAll(purchaseHistory);
```

사용하는 방법을 알았으니 이제 골드회원을 클래스로 만들면 된다.

```tsx
class GoldCustomerPolicy {
  private readonly policy: ExcellentCustomerPolicy;

  constructor(){
    this.policy = new ExcellentCustomerPolicy()
    this.policy.add(new GoldCustomerPurchaseAmountRule());
    this.policy.add(new PurchaseFrequencyRule());
    this.policy.add(new ReturnRateRule());
  }

  complyWithAll(history:PurchaseHistory): boolean {
    return this.policy.complyWithAll(history)
  }
}
```

실버 회원도 동일하게 클래스로 구현해주면 된다.

```tsx
class SliverCustomerPolicy {
  private readonly policy: ExcellentCustomerPolicy;

  constructor(){
    this.policy = new ExcellentCustomerPolicy()
    this.policy.add(new PurchaseFrequencyRule());
    this.policy.add(new ReturnRateRule());
  }

  complyWithAll(history:PurchaseHistory): boolean {
    return this.policy.complyWithAll(history)
  }
}
```

[구현된 코드 예시](https://www.typescriptlang.org/play?#code/C4TwDgpgBACgrgJwMYAsCGBnCAJAlh4AewRCgF4BvAWACgoojg0AbAQQFtC4A7YALijc47AEYQEAblr0wiVJggAxBBACOcCNyQgY4gLKFeKAUNHipdKCuCJuAJTTAIJ4WMm0AvrVq5e4gGZoSNAAogAewczMmsAAwnAEhOzidnDRUNSWhADWAgAUKPhEJALwyOhYeIkkAJTkAHxQIoSE0WjcFl40tEjMmBhQAOKtACbxickIZfJYHFy8qem47GDRybwD4ZHRvONEk4vQmfQ5BUXEIHzTFTjntQLNrRDtGdL0VhA2CNxQAIwADIDAVAADxkKCFaogAB0jBYcx4wAs9C6XR6fQwA2uCmUag0WhAhygy1WEHWwE2EQgURieySKTSR2kp0hxUu2MqdxANQeLTaP2O72stj+-1B4NZF2hsnKOJU6k02l0CAMRmRUFR3hovX6UDsn1sDicRJJaxilO2tIS+wZ6WOLK5VzkNyqbJ5TT5zwFb3owu+EK50L99kc0DBUH+0MBv3Vmu62oxFupOzi1vpU1auG0r0sCEZGAEAGVPiCtsmrRNbRB6hY3khDAQ80hink6oL6MBIUH8+RBBAAO5QYvAVtvNGWNAjEZ5PPRARlmm7NMHRltn0MLuziAYaGT6dbmpjrX0esrZggADquE7rCiZyhpWdCldFxq7ag0WAUH8fQA5r3AmYLB1Q7Td82hfxiBCIIUDyPJnlQOoyEad96Fwfw8gAQgQlBoQdKEajqH80H-cFAOA9cPEPSxfQNf1iN-WNPC1dFdWGZgxmXcQYEzbNBTABBcAAN1DD5J0Mc8oDAXjLigBcUzpSYeOYLMQFrSx624Rs4GbYhW3fTt8GlGTe24Ac5KpRdU0rDMVO0UcaI3IzpLsmE9zyMzB3YzibI5CAEQWRlWxqECnJ3FzVN3KcPPMvzcQVAlDmC0LDPCmSounTy9TokNjSCwimPjE8klWS9rxQW9mHvNknVlTkCN5J4XnfYMwuM1zoVPUqrxvO9JVqI8aHHHVMSHFShPERTuJM-jBJEpwxJGCTSAi7R50shSuNs1T1OKrTgCbFs33XVL2tU0zzPkisbW2+zqPeNrVrc6Ksri+V8W0JKCpOrsnoymLB31L5cogL6QsGusSvPHqKr6x0-Jfe4PSa71HNa06-q66Hysq6rX0GjwgA)

## 4. 자료형 확인에 조건 분기 사용하지 않기

인터페이스를 사용했을 때에도 조건 분기가 줄어들지 않는 경우가 존재한다.

```tsx
interface HotelRates {
  fee: () => Money;
}

class RegularRates implements HotelRates {
  fee() {
    return new Money(70000);
  }
}

class PremiumRates implements HotelRates {
  fee() {
    return new Money(120000);
  }
}
```

전략 패턴을 사용해 호텔 요금을 구현한 코드이다.

성수기에만 숙박요금이 비싸져야해서 다음과 같은 코드를 추가했다.

```tsx
let busySeasonFee: Money;

if (hotelRates instanceof RegularRates) {
  busySeasonFee = hotelRates.fee().add(new Money(30000))
}
if (hotelRates instanceof PremiumRates) {
  busySeasonFee = hotelRates.fee().add(new Money(50000))
}
```

분명 인터페이스를 추가했는데도 불구하고 조건분기가 생겨버렸다.

> **리스코프 치환 원칙**

클래스의 기반 자료형과 하위 자료형 사이에 성립하는 규칙, 즉, 기반 자료형을 하위 자료형으로 변경해도 코드는 문제없이 동작해야 하는 규칙이다. 쉽게 말하면 부**모 객체를 호출하는 동작에서 자식 객체가 부모 객체를 완전히 대체할 수 있다는 원칙**이다. (이는 자식 객체의 확장이 부모 객체의 방향을 온전히 따르도록 하는 원칙이다.)
> 

인터페이스를 잘못 사용하고 있기 때문에 발생하는 문제이기 때문에 인터페이스를 수정해야 한다.

```tsx
interface HotelRates {
  fee: () => Money;
	buseySeasonFee: () => Money;
}

class RegularRates implements HotelRates {
  fee() {
    return new Money(70000);
  }
	busySeasonFee() {
		return this.fee().add(new Money(30000));
	}
}

class PremiumRates implements HotelRates {
  fee() {
    return new Money(120000);
  }
	busySeasonFee() {
		return this.fee().add(new Money(50000));
	}
}

// 사용처
const busySeasonFee = hotelRates.busySeasonFee();
```

이제 성수기에 필요한 금액을 쓸 때 분기처리 없이 사용이 가능해진다.

## 5. 인터페이스 사용 능력이 중급으로 올라가는 첫걸음

조건 분기를 써야하는 상황에는 인터페이스 설계를 고려해보자.

|  | 초보자 | 중급자 이상 |
| --- | --- | --- |
| 분기 | if조건문과 switch조건문만 사용 | 인터페이스 설계 사용 |
| 분기마다의 처리 | 로직을 그냥 작성 | 클래스 사용 |

## 6. 플래그 매개변수

```tsx
function damage(damageFlag: boolean, damageAmount: number) {
  if (damageFlag) {
      member.hitpoint -= damageAmount;
      if(0 < member.hitPoint) return;

      member.hitPoint = 0;
      member.addState(StateType.dead);
  } else {
     member.hitpoint -= damageAmount;
	   if(0 < member.magicPoint) return;

     member.magicPoint = 0;
  }
}
```

플래그 매개변수를 통해 내부에서 물리/마법 공격을 구분하는 함수다.

플래그 매개변수는 어떤 일을 하는지 예측이 어려워 가독성도 낮아지고 개발 생산성도 저하된다.

### 메서드 분리하기

```tsx
hitPointDamage(damageAmount: number) {
	  member.hitpoint -= damageAmount;
    if(0 < member.hitPoint) return;

    member.hitPoint = 0;
    member.addState(StateType.dead);
}

magicPointDamage(damageAmount: number) {
    member.hitpoint -= damageAmount;
	  if(0 < member.magicPoint) return;
    
		member.magicPoint = 0;
}
```

각각의 메서드를 분리하여 가독성을 높이는 것이 좋다.

### 전환은 전략패턴으로 구현하기

물리 데미지와 마법 데미지를 서로 변환(전환)해야하는 상황이 생긴다면 인터페이스를 사용한다.

```tsx
interface Damage {
  execute: (damageAmount: number) => void;
}

class HitPointDamage implements Damage {
	execute(damageAmount: number): void {
		member.hitpoint -= damageAmount;
    if(0 < member.hitPoint) return;

    member.hitPoint = 0;
    member.addState(StateType.dead);
	}
}

class MagicPointDamage implements Damage {
	execute(damageAmount: number): void {
    member.hitpoint -= damageAmount;
	  if(0 < member.magicPoint) return;
    
		member.magicPoint = 0;
	}
}
```

각 클래스를 구현한뒤에 이제 전환해야할 때 써야할 메서드를 구현한다.

```tsx
enum DamageType {
	hitPoint,
	magicPoint
}

const damages = new Map<DamageType, Damage>();

function applyDamage(damageType: DamageType, damageAmount: number) {
	const damage: Damage = damages.get(damageType);
	damage.execute(damageAmount);
}

// 사용처
applyDamage(DamageType.magicPoint, damageAmount);
```

사용하는 곳에서는 이제 어떤 데미지를 쓸지(매직, 물리) 중 선택해서 호출하면 해결이 된다.