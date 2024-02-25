## 1. 데드 코드

```tsx
if (level > 99) {
    level = 99;
}

if (level === 1) {
} else if (level === 100) {
}
```

절대로 실행되지 않는 조건 내부에 있는 코드를 **데드 코드**, **도달 불가능한 코드(unreachable code)** 라고 부른다.

-   코드의 가독성을 떨어트림
-   해당 코드가 어떤 조건에서 실행되는지 생각하게 만들어 읽는 사람을 혼란스럽게 만든다.
-   지금까지 실행되지 않는 죽은 코드가 사양이 변경되며 도달 가능해져 버그가 발생할 수 있다.

따라서 데드 코드는 **발견하는 즉시 제거**하는 것이 좋다.

## 2. YANGI 원칙

‘’**Y**ou **A**ren’t **G**onna **N**eed **I**t’, 지금 필요 없는 기능을 만들지 말라! 라는 의미이다.

지금 당장 필요한 것들만 만들어야 문제가 발생하지 않는다.

-   예측에 들어맞지 않는 로직은 데드코드가 된다.
-   에측해서 만든 코드는 일반적으로 복잡하고, 가독성을 낮춰 읽는 사람을 혼란스럽게 만든다.
-   코드를 미리 작성해 두어도 결국 시간 낭비이다. (최대한 간단한 형태로 만드는 것이 중요한 작업에 집중할 수 있다.)

## 3. 매직 넘버

설명이 없는 숫자는 개발자를 혼란스럽게 만든다.

```tsx
// BAD
class ComicManager {
	isOk(): boolean {
		return **60** <= this.value;
	}

	tryConsume() {
		let tmp = value - **60;**
		if (tmp < 0) {
			throw new Error();
		}
		this.value = tmp;
	}
}

// GOOD
class ReadingPoint {
	private static readonly MIN = 0;

	private static readonly TRIAL_READING_POINT = 60;

	readonly value: number;

	constructor(value: number) {
		if (this.value < ReadingPoint.MIN) {
			throw new Error();
		}
		this.value = value;
	}

	// 구독할 수 있는지 확인
	canTryRead(): boolean {
		return ReadingPoint.TRIAL_READING_POINT <= value;
	}

	// 체험 구독하기
	consumeTrial(): ReadingPoint {
		return new ReadingPoint(this.value - ReadingPoint.TRIAL_READING_POINT);
	}

	// 포인트 추가하기
	add(point: ReadingPoint) {
		return new ReadingPoint(this.value + point.value);
	}
}
```

위 코드에서 계속해서 `60` 이라는 숫자가 등장하는데 무슨 숫자인지 알 수가 없다.

이 60은 일주일 동안 웹툰을 체험 구독할 때 필요한 포인트인데, 아무런 설명이 없어 숫자의 의도를 알기 힘들다. 만약 60이 50으로 바뀌었을 때도 매직 넘버가 사용된 곳을 모두 찾아가며 수정해야 한다.

그러므로 `TRIAL_READING_POINT` 라는 상수로 포인트를 관리하여 훨씬 쉽게 이해할 수 있고 유지보수도 편리하게 할 수 있다.

## 4. 문자열 자료형에 대한 집착

문자열에 무리하게 여러 개의 값을 하나의 변수에 넣으면 의미를 알기 어려워 진다.

```tsx
// BAD
// 레이블 문자열, 표시색(RGB), 최대 문자 수
const title = "타이틀,255,250,240,65";

// GOOD
const title = "타이틀";
const rgbColor = "255,250,240";
const maxString = "65";
```

데이터를 split으로 구분해서 분리하여 사용할 경우엔 의미가 다른 값은 각각 다른 변수에 저장하는 것이 좋다.

## 5. 전역 변수

전역 변수란 모든 곳에서 접근할 수 있는 변수를 말한다.

전역 변수로 선언하게 되면 모든 곳에서 참조할 수 있고 조작도 가능한 변수이다.

전역 변수를 사용할 때의 문제점

-   어디에서 어떤 시점에 값을 변경했는지 파악하기 힘들다.
-   동기화를 제대로 설계하지 않으면 락을 얻기 위해 대기하는 시간이 길어져서 성능을 떨어트리고,잘못하면 데드락 상태에 빠질 수도 있다.

### 영향 범위가 최소화되도록 설계하기

전역 변수는 영향 범위가 너무 넓기에 가능한 한 되도록 좁게 설계해야 한다.

전역 변수를 사용하고 싶다면, 정말로 필요한지 꼭 검토해보고 최대한 한정된 클래스에만 접근할 수 있는 형태로 설계해라

## 6. null 문제

방어구를 계산하는 간단한 클래스이다.

```tsx
class Member {
    private head: Equipment;
    private body: Equipment;
    private arm: Equipment;
    private defence: number;

    // 생략

    // 방어구의 방어력 + 캐릭터의 방어력 합산 값
    totalDefnece(): number {
        let total = this.defence;
        total += this.head.defence;
        total += this.body.defence;
        total += this.arm.defence;
        return total;
    }

    // 모든 방어구 장비 해제
    takeOffAllEquipments() {
        this.head = null;
        this.body = null;
        this.arm = null;
    }

    // 방어구 출력
    showBodyEquipment() {
        this.showParam(this.body.name);
        this.showParam(this.body.defence);
        this.showParam(this.body.magicDefence);
    }
}
```

이 코드에서는 NullPointerException이 발생하는 경우가 있는데 방어구를 장착하지 않는 상태를 null로 처리하기 때문이다. 이를 해결하기 위해서는 null 인지 체크하는 함수가 있어야 한다.

```tsx
	totalDefnece(): number {
		let total = this.defence;
		if (this.head !== null) total += this.head.defence;
		if (this.body !== null) total += this.body.defence;
		if (this.arm !== null) total += this.arm.defence;
		return total;
	}

	showBodyEquipment() {
		if (this.body !== null) {
			this.showParam(this.body.name);
			this.showParam(this.body.defence);
			this.showParam(this.body.magicDefence);
		}
	}
```

이렇게 null 체크를 계속해서 한다면 모든 곳에서 null 체크를 해야 하고 코드도 많아져 가독성이 떨어질 것이고 실수도 생길 수 있다.

null 대신에 **‘무언가를 갖고 있지 않은 상태’**와 **‘무언가 설정되지 않은 상태’** 그 자체로 의미가 있는 좋은 상태이다.

### null을 리턴/전달하지 말기

null 체크를 하지 않으려면 null을 다루지 않게 만들면 된다.

다음 두가지를 만족하는 설계를 하면 된다.

-   null을 리턴하지 않는 설계
-   null을 전달하지 않는 설계

‘장비하지 않음’을 null이 아닌 방법으로 표현하는 방법이다.

```tsx
class Equipment {
    static readonly EMPTY = new Equipment("장비 없음", 0, 0, 0);
    private head: Equipment;
    private body: Equipment;
    private arm: Equipment;
    private defence: number;

    readonly name: string;
    readonly price: number;
    readonly defence: number;
    readonly magicDefence: number;

    constructor(
        name: string,
        price: number,
        defence: number,
        magicDefence: number
    ) {
        if (name.length === 0) throw new Error("잘못된 이름입니다.");

        this.name = name;
        this.price = price;
        this.defence = defence;
        this.magicDefence = magicDefence;
    }

    takeOffAllEquipments() {
        this.head = Equipment.EMPTY;
        this.body = Equipment.EMPTY;
        this.arm = Equipment.EMPTY;
    }
}
```

모든 장비를 해제할 때에도 `EMPTY`를 할당해 항상 인스턴스가 존재하기 만들어 null 예외가 발생하지 않도록 한다.

### null 안전

null 안전이란 null에 의한 오류가 아예 발생하지 않게 만드는 구조를 말한다.

코틀린의 경우에는 모든 자료형에 null 안전 자료형을 사용해 null을 할당하지 못한다.

## 7. 예외를 catch하고서 무시하는 코드

try-catch로 예외를 catch해놓고 별다른 처리를 하지 않는 코드이다.

```tsx
try {
    reservations.add(product);
} catch (e) {
    // 아무 처리도 하지 않음
}
```

예외를 무시하는 코드는 사악한 로직이다. 😈

### 원인 분석을 어렵게 만듦

예외를 무시하는 코드의 문제는 오류가 나도 오류를 탐지할 방법이 없어진다는 것이다.

데이터에 문제가 생기는 경우에 외부에서는 아무런 문제가 없는 것처럼 보이게 만들게 된다.

추후 ‘예약 리스트에 추가가 안된다’, ‘예약 리스트에 알 수 없는 상품이 추가된다’ 라는 문제가 보고되어 원인 분석을 할 때에 어떤 코드에서 문제가 발생했는지 찾기 힘들다.

### 문제가 발생했다면 소리치기

이러한 상황을 피하려면, 예외를 곧바로 통지, 기록하는 것이 좋다. 위 예시에서는 catch문에서는 최소한 로그로 기록하고, 상위 레이어의 클래스로 오류를 통지하는 것이 좋다.

```tsx
try {
    reservations.add(product);
} catch (e) {
    reportError(e);
    requestNotifyError("예악할 수 없는 상품입니다.");
}
```

**문제가 발생하는 즉시 소리쳐서 잘못된 상태를 막는 좋은 구조**로 변경했다.

## 8. 설계 질서를 파괴하는 메타 프로그래밍

프로그램 실행 중에 해당 프로그램 구조 자체를 제어하는 프로그래밍을 **메타 프로그래밍**이라고 한다. 자바에서 메타 프로그래밍을 활용해 클래스 구조를 읽고 쓸 때는 리플렉션 API를 사용한다. 자바스크립트에서는 [Reflect](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Reflect)를 사용하면 강제로 변환할 수 있다.

그렇기에 메타 프로그래밍을 ‘흑마법’이라고 표현하기도 한다.

### 리플렉션으로 인한 클래스 구조와 값 변경 문제

```tsx
class Level {
    private static readonly MIN = 1;
    private static readonly MAX = 99;
    readonly value: number;

    constructor(value: number) {
        if (value < Level.MIN || Level.MAX < value) {
            throw new Error();
        }
        this.value = value;
    }

    static initialize(): Level {
        return new Level(Level.MIN);
    }

    increase(): Level {
        if (this.value < Level.MAX) return new Level(this.value + 1);
        return this;
    }
}

const level = Level.initialize();
// Level : 1
console.log("Level : " + level.value);

Reflect.set(level, "value", 999);
// Level : 999
console.log("Level : " + level.value);
```

상수 MIN/MAX로 가드를 했고 `readonly`키워드로 불변하게 만들었지만 리플렉션을 사용하면 지정한 변수의 값도 바뀌고 접근하지 못하게 만든 변수도 접근이 가능해진다.

따라서 리플렉션을 남용하면 ‘잘못된 상태로부터 클래스를 보호하는 설계’ 와 ‘영향 범위를 최대한 좁게 만드는 설계’ 가 아무런 의미를 갖지 못하게 된다.

비유하자면 집을 외부 침임으로부터 안전하게 보호하려 시도는 했지만, 뒷문을 열어 놓은 것과 같다.

### 자료형의 장점을 살리지 못하는 하드 코딩

클래스의 인스턴스를 만들 때는 일반적으로 new 키워드르 사용하지만 리플렉션을 사용하면 메타 정보를 기반으로 인스턴스를 생성할 수 있다.

```java
static Object generateInstance(String packageName, String className) throws Exception {
	String fileName = packageName + "." + className;
	Class klass = Class.forName(fillName);
	Constructor constructor = klass.getDeclaredConstructor();
	return constructor.newInstance();
}

User user = (User)generateInstance("customer", "User");
```

```tsx
class User {
    private name = "홍길동";
    constructor() {
        console.log("User class 생성");
    }
    getName() {
        return this.name;
    }
}

const classMap = [User];

function createUser(className: string) {
    const someClass = classMap.find(
        (classItem) => classItem.name === className
    );
    if (!someClass) return;
    const instance = Reflect.construct(someClass, []);

    return instance;
}

const userInstance = createUser("User");
console.log((userInstance as User)?.getName());
```

`generateInstance` 메서드의 매개변수로 전달된 것이 단순한 문자열이기 때문에 자료형을 User로 아는 것이 아닌 string 타입으로 알게 된다. 또 IDE의 이름 변경 기능을 사용하면 User를 참조하는 모든 코드를 확인해서 빠짐없이 변경해주지만 문자열이기에 변경이 불가능하다.

### 단점을 이해하고 용도를 한정해서 사용하기

메타 프로그래밍을 사용할 때 단점을 이해하지 못하고 사용하면 유지 보수와 변경이 힘들어진다.

메타 프로그래밍을 사용하고 싶다면 시스템 분석 용도로 한정하거나 아주 작은 범위에서만 활용하는 등 리스크를 최소화 해야 한다.

## 9. 기술 중심 패키징

패키지를 구분할 때도 폴더를 적절하게 나누지 않으면 악마를 불러들일 수 있다.

-   온라인 쇼핑몰의 폴더 구성
    -   UseCases
        -   재고\_유스케이스
        -   지문\_유스케이스
        -   지불\_유스케이스
    -   Entities
        -   입고\_엔티티
        -   출고\_엔티티
        -   장바구니\_엔티티
        -   주문\_엔티티
        -   발주\_엔티티
        -   청구\_엔티티
    -   ValueObjects
        -   안전\_재고량
        -   재고*회전*기간
        -   발주\_금액
        -   주문처
        -   청구\_금액
        -   할인\_포인트
        -   신용카드\_번호

위 파일에서 주문처는 주문 유스케이스와 관련이 있을 것이다. 그렇다면 발주 금액은 어디에 관련이 있을까? 이는 재고 유스케이스에서 사용되고 있다. 이름으로 구분이 어려워 발주 금액이 주문 유스케이스로 바뀌어 사용될 가능성이 있다. → 버그 발생

파일들을 구분해보면 **재고, 주문, 결제** 용도로 나눌 수 있다. 이를 디자인 패턴에 따라 구분했기에 어떤 것이 어떤 종류와 관련됐는지 구분하기 힘든 것이다. 이처럼 구조에 따라 폴더와 패키지를 나누는 것을 **기술 중심 패키징**이라고 부른다.

비지니스 클래스를 기술 중심 패키징에 따라 폴더를 구분하면 응집도가 낮아져 관련성을 알기 매우 힘들어진다.

-   비지니스 개념 종류별로 구분한 폴더 구성
    -   재고
        -   재고\_유스케이스
        -   발주\_엔티티
        -   입고\_엔티티
        -   출고\_엔티티
        -   안전\_재고량
        -   재고*회전*기간
        -   발주\_금액
    -   주문
        -   주문\_유스케이스
        -   장바구니\_엔티티
        -   주문\_엔티티
        -   주문처
    -   결제
        -   결제\_유스케이스
        -   청구\_엔티티
        -   청구\_금액
        -   할인\_포인트
        -   신용카드\_번호

관련된 개념끼리 모아놔 관련 파일을 찾아다니지 않아도 된다.

## 10 샘플 코드 복사해서 붙여넣기

샘플 코드를 그대로 복사하고 붙여넣기 하면 설계 측면에서 좋지 않은 구조가 되기 쉽다.

샘플 코드는 참고만 하고, 클래스 구조를 잘 설계해서 사용하기 바란다.

## 11. 은 탄환

서양에서 늑대 인간과 악마는 은으로 만든 총알로 죽일 수 있다고 알려져 있다. 그래서 어떤 문제를 해결하는 비장의 무기, 묘책을 ‘**은 탄환**’이라고 부른다. **하지만 소프트웨어 개발에는 은 탄환이 없다.**

중요한 점은 어떤 문제가 있을 때, 어떤 방법이 해당 문제에 효과적인지, 비용이 더 들지는 않는지 평가하고 판단하는 자세다.

설계에 Best라는 것은 없다. 항상 Better를 목표로 할 뿐이다.

> 매슬로의 망치(Maslow’s hammer)

가진 것이 망치밖에 없다면, 다른 모든 것을 못으로 취급하게 된다는 확증 편향이다.
해석하자면 ‘한 가지 절대적인 방법으로 모든 것을 해결할 수는 없다’ 라는 뜻이다.

>
