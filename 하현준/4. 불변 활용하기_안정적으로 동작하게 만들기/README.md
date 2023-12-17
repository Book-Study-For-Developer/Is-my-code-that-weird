# 4장 불변 활용하기: 안정적으로 동작하게 만들기

### 1. 재할당

변수에 값을 다시 할당하는 것을 **재할당, 파괴적 할당** 이라고 한다. 재할당은 변수의 의미를 바꿔 추측하기 어렵게 만든다.

**불변 변수로 만들어 재할당 막기**

재할당을 막기 위해 변수를 다시 변경하지 못하도록 `const` 키워드를 사용해 막는다.

```java
function damage(): void {
  const basicAttackPower = member.power() + member.weakAttack();
	const finalAttackPower = parseInt(basicAttackPower * (1 + member.speed() / 100));
	const reduction = parseInt(enemy.defence / 2);
	const damage = Math.max(0, finalAttackPower - reduction);

	return damage;
}
```

**매개변수도 불변으로 만들기**

매개변수를 변경하면 값의 의미가 바뀔수 있다. 그러므로 매개변수에도 `final` 키워드를 붙이면 된다. (javascript 에서는 매개변수에 따로 변경이 불가능하도록 하는 방법이 없다.)

```java
void addPrice(final int productPrice) {
	final int increasedTotalPrice = totalPrice + productPrice;
  if (MAX_TOTAL_PRICE < increasedTotalPrice) {
	  throw new IllegalArgumentException("구매 상한 금액을 넘었습니다.");
  }
}
```

만약 막고 싶다면 매개변수를 객체로 받아서 변경을 불가능하게 막을 수 있다.

```tsx
function addPrice(params: { producePrice: number }): void {
    const { productPrice } = params;
    // productPrice = 100 변경이 불가능하므로 Error 발생
    const increasedTotalPrice = totalPrice + productPrice;
    if (MAX_TOTAL_PRICE < increasedTotalPrice) {
        throw new Error("구매 상한 금액을 넘었습니다.");
    }
}
```

### 2. **가변으로 인해 발생하는 의도하지 않은 방향**

사례 1: 가변 인스턴스 재사용하기

```tsx
class AttackPower {
    static readonly MIN = 0;
    value: number;
    constructor(value: number) {
        if (value < AttackPower.MIN) {
            throw new Error();
        }

        this.value = value;
    }
}
```

```tsx
class Weapon {
    static attackPower: AttackPower;
    constructor(attackPower: AttackPower) {
        Weapon.attackPower = attackPower;
    }
}
```

```tsx
const attackPower = new AttackPower(20);

const weaponA = new Weapon(attackPower);
const weaponB = new Weapon(attackPower);

weaponA.attackPower.value = 25;

console.log(weaponA.attackPower.value); // 25
console.log(weaponB.attackPower.value); // 25
// 재사용하는 값을 변경하면 두가지에 모두 영향을 준다.
// ---------------------------------------

const attackPowerA = new AttackPower(20);
const attackPowerB = new AttackPower(20);

const weaponA = new Weapon(attackPowerA);
const weaponB = new Weapon(attackPowerB);

weaponA.attackPower.value = 25;

console.log(weaponA.attackPower.value); // 25
console.log(weaponB.attackPower.value); // 20

// 별개의 인스턴스를 사용하면 영향을 끼지지 않는다.
```

사례 2: 함수로 가변 인스턴스 조작하기

```tsx
class AttackPower {
    static readonly MIN = 0;
    value: number;
    constructor(value: number) {
        if (value < AttackPower.MIN) {
            throw new Error();
        }

        this.value = value;
    }

    reinforce(increment: number): void {
        this.value += increment;
    }

    disable(): void {
        this.value = AttackPower.MIN;
    }
}

const attackPower = new AttackPower(20);
attackPower.reinforce(15);
// 어느 순간에 공격력이 0이 되는 순간이 발생
```

`disalbe` 메서드와 `reinforce` 메서드는 구조적인 문제를 가지고 있음

동일한 결과를 내기 위해서는 동일한 순서로 실행해야 한다. → 작업 실행 순서에 의존된다.(부수 효과가 발생)

**부수 효과의 단점**

**함수의 부수 효과**는 ‘함수가 매개 변수를 전달받고, 값을 리턴하는 것’ 이외에 외부 상태를 변경하는 것을 말한다.

-   주요 작용: 함수(메서드)가 매개변수를 전달받고, 값을 리턴하는 것
-   부수 효과: 주요 작용 이외의 상태를 변경을 일으키는 것

**함수의 영향 범위 한정하기**

-   데이터(상태)는 매개변수로 받는다.
-   상태를 변경하지 않는다.
-   값은 함수의 리턴 값으로 돌려준다.

_순수 함수와 비슷한 개념으로 생각이 든다._

**불변으로 만들어서 예기치 못한 동작 막기**

`AttackPower` 클래스 변경하기

```tsx
class AttackPower {
    static readonly MIN = 0;
    private readonly value: number;
    constructor(value: number) {
        if (value < AttackPower.MIN) {
            throw new Error();
        }
        this.value = value;
    }

    reinforce(increment: AttackPower) {
        return new AttackPower(this.value + increment.value);
    }

    disable() {
        return new AttackPower(AttackPower.MIN);
    }

    getValue() {
        return this.value;
    }
}
```

`Weapon` 클래스 변경하기

```tsx
class Weapon {
    private attackPower: AttackPower;

    constructor(attackPower: AttackPower) {
        this.attackPower = attackPower;
    }

    reinforce(increment: AttackPower) {
        const reinforced = this.attackPower.reinforce(increment);
        return new Weapon(reinforced);
    }

    getAttackPower() {
        return this.attackPower;
    }
}
```

```tsx
const attackPowerA = new AttackPower(20);
const attackPowerB = new AttackPower(20);

const weaponA = new Weapon(attackPowerA);
const weaponB = new Weapon(attackPowerB);

const increment = new AttackPower(5);
const reinforcedWeaponA = weaponA.reinforce(increment);

console.log(weaponA.getAttackPower().getValue()); // 20
console.log(reinforcedWeaponA.getAttackPower().getValue()); // 25
console.log(weaponB.getAttackPower().getValue()); // 20
```

`weaponA` 와 `reinforcedWeaponA` 는 서로 영향을 주지 않는 인스턴스로 생성이 가능하다.

### 3. 불변과 가변은 어떻게 다뤄야 할까

변수를 불변으로 만들때의 장점

-   변수의 의미가 변하지 않으므로, 혼란을 줄일 수 있음
-   동작이 안정적이게 되므로, 결과를 예측하기 쉬움
-   코드의 영향 범위가 한정적이므로, 유지 보수가 편리해짐

**가변으로 설계해야 하는 경우**

성능이 중요한 경우에는 가변으로 처리하는 것이 좋은 경우도 존재

-   크기가 큰 인스턴스를 새로 생성하는데 시간이 오래걸린다면 가변으로 사용하는게 좋을 수 있다.

상태를 변경하는 메서드 설계하기

히트포인트와 멤버를 나타내는 클래스

-   히트포인트는 0이상
-   히트포인트가 0이 되면, 사망 상태로 변경

```tsx
class HitPoint {
    amount: number;
}

class Member {
    hitPoint: HitPoint;
    states: States;

    damage(damageAmount: number) {
        this.hitPoint.amount -= damageAmount;
    }
}
```

0이 되어도 사망상태로 바꾸지 않는다. 상태를 변화 시키는 메서드(뮤테이터)로 상태를 변경해야 한다.

```tsx
class HitPoint {
	private readonly MIN = 0;
	amount: number;

	constructor(amount: number) {
		if(amount < HitPoint.MIN) {
			 throw new Error();
		}
		this.amount = amount;
	}

	damage(damageAmount): void {
		const nextAmount = amount - damageAmount;
		this.amount = Math.max(HitPoint.MIN, nextAmount);
	}

	boolean isZero() {
		return amount === HitPoint.MIN;
	}
}

class Member {
	private hitPoint: HitPoint;
	private states: States;

	// 초기값 세팅
  constructor(amount: number) {
    this.hitPoint = new HitPoint(amount);
  }

	damage(damageAmount: number): void {
		this.hitPoint.damage(damageAmount);
		if (this.hitPoint.isZero()) {
			this.states.add(StateType.dead);
		}
	}
}
```

**코드 외부와 데이터 교환은 극소화하기**

불변을 활용하더라도 코드 외부와의 데이터 교환은 주의해야 한다.

이를 위해 리포지터리 패턴(repository pattern)을 활용한다.

> **리포지터리 패턴이란?**
> 데이터베이스 등 데이터 소스의 제어 로직을 캡슐화하는 패턴이다. 특정 클래스 내부에 데이터베이스 관련 로직을 격리하므로, 애플리케이션 로직이 데이터베이스 관련 로직과 섞이지 않는다.
