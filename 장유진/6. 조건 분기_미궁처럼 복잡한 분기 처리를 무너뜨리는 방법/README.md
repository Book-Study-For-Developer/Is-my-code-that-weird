# 6장 조건 분기: 미궁처럼 복잡한 분기 처리를 무너뜨리는 방법

## 1. 조건 분기가 중첩되어 낮아지는 가독성
중첩을 하면 코드의 가독성이 크게 떨어지는 문제가 발생합니다.

코드를 읽을 때마다 낭비가 생기고, 코드를 읽는 사람 모두가 이런 일을 겪습니다.

가독성이 나쁜 코드는 팀 전체의 개발 생산성을 저하시킵니다.

사양 변경은 더 힘듭니다. 코드가 복잡하고 길면 로직을 정확하게 읽고 이해하기 어렵습니다.

충분히 이해하지 못한 상태에서 로직을 변경하면, 버그가 쉽게 숨어듭니다.

### 1.1 조기 리턴으로 중첩 제거하기
조기 리턴은 조건을 만족하지 않는 경우 곧바로 리턴하는 방법입니다.

```java
// 살아 있지 않은 경우 리턴하므로 처리를 종료합니다.
// 조기 리턴으로 변경하기 위해 조건을 반전했습니다.
if (member. hitPoint « 0) return;

if (member.canAct ()) {
    if (magic.costMagicPoint < member.magicPoint) {
        member.consumMagicPoint(magic.costMagicPoint);
        member.chant(magic);
    }
}
```
조기 리턴하는 형태로 변경하려면, 원래 조건을 반전해야 합니다.

```java
if (member. hitPoint « 0) return;
if (!member.canAct ()) return;
if (member.magicPoint < magic.costMagicPoint) return;

member.consumMagicPoint(magic.costMagicPoint);
member.chant(magic);
```
조기 리턴에는 또 다른 장점이 있습니다. 바로 조건 로직과 실행 로직을 분리할 수 있다는 것입니다.

조기 리턴을 사용해서 조건에 따라 실행 흐름이 달라지는 일을 막는 기법은 앞서 설명했던 가드와 비슷합니다.

가드와 조기 리턴은 모두 가독성을 좋게 해서, 로직을 빠르게 이해할 수 있게 해 줍니다.

### 1.2 가독성을 낮추는 else 구문도 조기 리턴으로 해결하기
else 구문도 중첩과 마찬가지로 조기 리턴을 사용해서 해결할 수 있습니다.

## 2. switch 조건문 중복
다음은 switch 조건문을 사용해서 코드를 작성할 경우 발생될 상황입니다. (2.1 ~ 2.4 요약)

1. switch 조건문을 사용해서 코드 작성하기
2. 같은 형태의 switch 조건문이 여러 개 사용되기 시작
3. 요구 사항 변경 시 수정 누락(case 구문 추가 누락)
4. 폭발적으로 늘어나는 switch 조건문 중복

switch 조건문이 중복 코드가 된 것입니다.(switch 조건문 클론 문제)

switch 조건문이 많아지면, 가독성이 낮아지고, 수정 누락과 개발 생산성 하락이 발생합니다.

결국 요구 사항이 추가될 때마다 case 구문이 누락될 것이고 이로 인해 버그가 만들어질 것입니다.

### 2.5. 조건 분기 모으기
switch 조건문 중복을 해소하려면, '**단일 책임 선택의 원칙**'을 생각해봐야 합니다.

`단일 책임 선택의 원칙: 소프트웨어 시스템이 선택지를 제공해야 한다면, 그 시스템 내부의 어떤 한 모듈만으로 모든 선택지를 파악할 수 있어야 한다.`

간단하게 말해, 조건식이 같은 조건 분기를 여러 번 작성하지 말고 한 번에 작성하자는 뜻입니다.

### 2.6 인터페이스로 switch 조건문 중복 해소하기
클래스가 거대해지면 관심사에 따라 작은 클래스로 분할하는 것이 중요합니다.

이러한 문제를 해결할 때는 '**인터페이스**'를 사용합니다.

인터페이스를 사용하면, 분기 로직을 작성하지 않고도 분기와 같은 기능을 구현할 수 있습니다.

도형을 예로 들어 봅시다.

```java
// 사각형
class Rectangle {
    private final double width;
    private final double height;

    double area() {
        return width * height;
    }
}

// 원
class Circle {
    private final double radius;
    
    double area() {
        return radius * radius * Math.PI;
    }
}
```
이 코드에서 면적을 구하는 메서드는 area로 같지만 서로 다른 클래스이기 때문에

아래처럼 instanceof를 사용해서 자료형을 판단한 뒤, 자료형을 강제로 변환해서 메서드를 호출해야 합니다.

```java
void showArea(Object shape){
    if (shape instanceof Rectangle) {
        System.out.println(((Rectangle) shape).area());
    }
    if (shape instanceof Circle) {
        System.out.println(((Circle) shape).area());
    }
}
```

인터페이스는 서로 다른 자료형을 같은 자료형처럼 사용할 수 있게 해 줍니다.

위의 사각형과 원을 정의하는 코드를 다음과 같이 수정해봅시다.

```java
// 도형
interface Shape {
    double area();
}

// 사각형
class Rectangle implements Shape{
    private final double width;
    private final double height;

    double area() {
        return width * height;
    }
}

// 원
class Circle implements Shape{
    private final double radius;
    
    double area() {
        return radius * radius * Math.PI;
    }
}
```

이렇게 하면 면적을 구하는 코드는 Rectangle과 Circle 클래스가 서로 다르지만 Shape이라는 자료형으로 다룰 수 있습니다.

```java
void showArea(Shape shape){
    System.out.println(shape.area());
}

...
        
Rectangle rectangle = new Rectangle(8, 12);
showArea(rectangle);
```

인터페이스를 사용하면, 조건 분기를 따로 작성하지 않고도 각각의 코드를 적절하게 실행할 수 있습니다.

인터페이스를 활용하면, 자료형 판정 분기를 따로 작성하지 않아도 됩니다.

이처럼 '각각의 코드를 간단하게 실행할 수 있게 하는 것'이 인터페이스의 큰 장점 중 하나입니다.

### 2.7 인터페이스를 switch 조건문 중복에 응용하기(전략 패턴)

다음은 게임 스킬을 예로 든 전략 패턴입니다.

1. 종류별로 다르게 처리해야 하는 기능을 인터페이스의 메서드로 정의하기
```java
String name();
int costMagicPoint();
int attackPower();
int costTechnicalPoint();
```
2. 인터페이스의 이름을 결정하는 방법: '어떤 부류에 속하는가?'
```java
// 마법 자료형
interface Magic {
    String name();
    int costMagicPoint();
    int attackPower();
    int costTechnicalPoint();
}
```
3. 종류별로 클래스로 만들기

| 마법   | 클래스       |
|------|-----------|
| 파이어  | Fire      |
| 라이트닝 | Lightning |
| 헬파이어 | HellFire  |
4. 각각의 클래스에 인터페이스 구현하기
```java
// 각각의 클래스에는 Magic 인터페이스의 함수가 구현되어 있다고 가정합니다.
// 마법 '파이어'
class Fire implements Magic {
    // 생략
}

// 마법 '라이트닝'
class Lightning implements Magic {
    // 생략
}

// 마법 '헬파이어'
class HellFire implements Magic {
    // 생략
}
```
5. switch 조건문이 아니라, Map으로 변경하기
   - Map을 만들고 키를 enum으로 정의하고, 값을 인터페이스 구현 클래스의 인스턴스로 지정합니다.
   - **인터페이스를 사용해서 처리를 한꺼번에 전환하는 설계를 디자인 패턴 중 '전략 패턴(strategy pattern)'이라고 합니다.**
```java
enum MagicType {
    fire,
    lightning,
    hellfire
}
```
```java
final Map<MagicType, Magic> magics = new HashMap<>();

final Fire fire = new Fire();
final Lightning lightning = new Lightning();
final HellFire hellFire = new HellFire();

magics.put(MagicType.fire, fire);
magics.put(MagicType.lightning, lightning);
magics.put(MagicType.hellfire, hellFire);

void magicAttack(final MagicType magicType) {
    final Magic usingMagic = magics.get(magicType);
    
    showMagicName(usingMagic);
    consumeMagicPoint(usingMagic);
    consumeTechnicalPoint(usingMagic);
    magicDamage(usingMagic);
}

// 마법 이름을 화면에 출력하기
void showMagicName(final Magic magic) {
    final String name=magic.name();
    // nane을 사용해서 출력하는 처리
}

// 매직포인트 소비하기
void consumeMagicPoint (final Magic magic) {
    final int costMagicPoint=magic.costMagicPoint();
    // costNagicPoint를 사용해서 마법 소비 처리
}

// 테크니컬포인트 소비하기
void consumeTechnicalPoint (final Magic magic) {
    final int costTechnicalPoint = magic.costTechnicalPoint();
    // costTechnicalPoint를 사용해서 테크니컬포인트 소비 처리
}

// 대미지 계산하기
void magicDamage(final Magic magic) {
    final int attackPower = magic.attackPover):
    // attackPower를 사용해서 대미지 계산
}
```
6. 메서드를 구현하지 않으면 오류로 인식하게 만들기
   - 인터페이스의 메서드는 반드시 구현되어야 컴파일할 수 있기 때문에 실수를 방지할 수 있습니다.
7. 값 객체화 하기(기본 자료형 대신 값 객체 만들기)
   - 위에서 선언한 Magic 인터페이스를 다음과 같이 변경합니다.
```java
interface Magic {
    String name();
    MagicPoint costMagicPoint();
    AttackPower attackPower();
    TechnicalPoint costTechnicalPoint();
}
```

### 2 챕터 마무리
나쁜 switch 조건문 설계가 발생하는 이유

#### 첫 번째 이유
종류에 따라 처리를 전환하는 방법을 switch 조건문밖에 모르기 때문입니다. 이 문제가 가장 크다고 생각합니다.

객체 지향 언어 대부분은 인터페이스 혹은 인터페이스와 동등한 개념을 제공합니다.

그런데 인터페이스의 목적과 효과(조건 분기 감소 등)을 모르는 경우가 많으며, 어느정도 알아도 활용하지 않는 경우가 많습니다.

조건 분기를 해야 하는 경우, 무턱대고 switch 조건문을 사용하면 안됩니다. 일단 인터페이스를 활용해서 설계할 수 없는지 검토하는 것이 좋습니다.

#### 두 번째 이유
커뮤니케이션과 관련된 문제입니다.

중복 코드의 문제점을 알고 있지만, 이를 막기 위한 커뮤니케이션이 제대로 이루어지지 않는 것입니다.

특별한 이유 없이 switch 조건문을 사용하면, 중복 코드가 많아지고 제어가 어려워집니다.

복잡해지기 쉬운 부분은 동료들과 확실하게 논의하면서 진행합시다.

## 3. 조건 분기 중복과 중첩
인터페이스는 switch 조건문의 중복을 제거할 수 있을 뿐만 아니라, 다중 중첩된 복잡한 분기를 제거하는 데 활용할 수 있습니다.

같은 판정 로직을 재사용하려면 어떻게 해야할까요?

```java
// 골드 회원 조건 : 구매한 금액 100만원 이상, 한 달에 구매하는 횟수 10회 이상, 반품률 0.1% 이하
/**
 * @return 골드 회원이라면 true
 * @param history 구매 이력
 */
boolean isGoldCustomer (PurchaseHistory history) {
    if (1000000 <= history.totalAmount) {
        if (10 <= history.purchaseFrequencyPerMonth) {
            if (history.returnRate <= 0.001) {
                return true;
            }
        }
    }
    return false;
}

// 실버 회원 조건 : 한 달에 구매하는 횟수 10회 이상, 반품률 0.1% 이하
/**
 * @return 실버 회원이라면 true
 * @param history 구매 이력
 */
boolean isSilverCustomer (PurchaseHistory history) {
    if (10 <= history.purchaseFrequencyPerMonth) {
        if (history.returnRate <= 0.001) {
            return true;
        }
    }
    return false;
}
```

### 3.1 정책 패턴으로 조건 집약하기
조건을 부품처럼 만들고, 부품으로 만든 조건을 조합해서 사용하는 패턴입니다.

우선 하나하나의 규칙(판정 조건)을 나타내는데 사용할 인터페이스를 만듭니다.
```java
interface ExcellentCustomerRule {
    /**
     * @return 실버 회원이라면 true
     * @param history 구매 이력
     */
    boolean ok(final PurchaseHistory history);
}
```

각각의 조건을 인터페이스를 사용해서 클래스로 만듭니다.
```java
class GoldCustomerPurchaseAmountRule implements ExcellentCustomerRule {
    public boolean ok(final PurchaseHistory history) {
        return 1000000 <= history.totalAmount;
    }
}

class PurchaseFrequencyRule implements ExcellentCustomerRule {
    public boolean ok(final PurchaseHistory history) {
        return 10 <= history.purchaseFrequencyPerMonth;
    }
}

class ReturnRateRule implements ExcellentCustomerRule {
    public boolean ok(final PurchaseHistory history) {
        return history.returnRate <= 0.001;
    }
}
```

이어서 정책 클래스를 만듭니다. add 메서드로 규칙을 집약하고, complyWithAll 메서드로 내부에서 규칙을 모두 만족하는지 판정합니다.

```java
class ExcellentCustomerPolicy {
    private final Set<ExcellentCustomerRule> rules;
    
    ExcellentCustomerPolicy() {
        rules = new HashSet();
    }

    /**
     * 규칙 추가
     * 
     * @param rule 규칙
     */
    void add(final ExcellentCustomerRule rule) {
        rules.add(rule);
    }

    /**
     * @param history 구매 이력
     * @return 규칙을 모두 만족하는 경우 true
     */
    boolean complyWithAll(final PurchaseHistory history) {
        for (ExcellentCustomerRule each : rules) {
            if (!each.ok(history)) return false;
        }
        return true;
    }
}
```

마지막으로 방금 선언한 정책 클래스를 멤버변수로 갖는 골드 회원 정책 클래스, 실버 회원 정책 클래스를 생성합니다.

```java
class GoldCustomerPolicy {
    private final ExcellentCustomerPolicy policy;
    
    GoldCustomerPolicy() {
        policy = new ExcellentCustomerPolicy();
        policy.add(new GoldCustomerPurchaseAmountRule());
        policy.add(new PurchaseFrequencyRule());
        policy.add(new ReturnRateRule());
    }

    /**
     * @param history 구매 이력
     * @return 규칙을 모두 만족하는 경우 true
     */
    boolean complyWithAll(final PurchaseHistory history) {
        return this.policy.complyWithAll(history);
    }
}

class SilverCustomerPolicy {
    private final ExcellentCustomerPolicy policy;

    GoldCustomerPolicy() {
        policy = new ExcellentCustomerPolicy();
        policy.add(new PurchaseFrequencyRule());
        policy.add(new ReturnRateRule());
    }

    /**
     * @param history 구매 이력
     * @return 규칙을 모두 만족하는 경우 true
     */
    boolean complyWithAll(final PurchaseHistory history) {
        return this.policy.complyWithAll(history);
    }
}
```

##  4. 자료형 확인에 조건 분기 사용하지 않기
인터페이스를 사용했는데도 자료형 확인에 조건 분기가 사용된다면 **리스코프 치환 원칙**을 위반한 것입니다.

`리스코프 치환 원칙: 기반 자료형을 하위 자료형으로 변경해도 코드는 문제없이 동작해야 한다.`

여기서 기반 자료형은 인터페이스, 하위 자료형은 인터페이스를 구현한 클래스입니다.

리스코프 치환 원칙을 위반하면 자료형 판정을 위한 조건 분기 코드가 점점 많아져서, 유지 보수하기 어려운 코드가 되어버립니다.

**인터페이스의 의미를 충분히 이해하지 못하고 사용하면 이와 같은 로직이 자주 만들어집니다.**

## 5. 인터페이스 사용 능력이 중급으로 올라가는 첫걸음
인터페이스를 잘 사용하면 조건 분기를 크게 줄일 수 있습니다.

따라서 코드를 단순하게 만들 수 있고, **인터페이스를 잘 사용하는지**가 곧 설계 능력의 전환점이라고 할 수 있습니다.

|    | 초보자                   | 중급자 이상      |
|------|-----------------------|-------------|
| 분기  | if 조건문과 switch 조건문만 사용 | 인터페이스 설계 사용 |
| 분기마다의 처리 | 로직을 그냥 작성             | 클래스 사용      |
설계 스킬별 사고 방식의 차이(필자의 생각)

**조건 분기를 써야 하는 상황에는 일단 인터페이스 설계를 떠올리자!**

## 6. 플래그 매개변수

메서드의 기능을 전환하는 boolean 자료형의 매개변수를 **플래그 매개변수**라고 부릅니다.

플래그 매개변수를 받는 메서드는 어떤 일을 하는지 예측하기 굉장히 힘듭니다.

예측을 하기 위해서는 반드시 메서드 내부 로직을 확인해야 하므로, 가독성이 낮아지며 개발 생산성이 저하됩니다.

boolean 자료형뿐만 아니라, int 자료형을 사용해 기능을 전환하는 경우에도 마찬가지입니다. (기능 전환을 하기 위한 매개변수는 셀렉터 매개변수라고도 부릅니다.)

#### Bad 로직 예시
```java
damage(true, damageAmount);

void damage(boolean damageFlag, int damageAmount) {
    if (damageFlag == true) {
        // 물리 대미지
        member.hitPoint -= damageAmount;
        if (0 < member.hitPoint) return;
        
        member.hitPoint = 0;
        member.addState(StateType.dead);
    }
    else {
        // 마법 대미지
        member.magicPoint -= damageAmount;
        if (0 < member.magicPoint) return;
        
        member.magicPoint = 0;
    }
}
```

### 6.1 메서드 분리하기
메서드는 하나의 기능만 하도록 설계하는 것이 좋습니다.

따라서 플래그 매개변수를 받는 메서드는 기능별로 분리하는 것이 좋습니다.

#### 위의 물리 대미지, 마법 대미지 기능을 각각의 메서드로 분리
```java
void hitPointDamage(final int damageAmount) {
    member.hitPoint -= damageAmount;
    if (0 < member.hitPoint) return;
    
    member.hitPoint = 0;
    member.addState(StateType.dead);
}

void magicPointDamage(final int damageAmount){
    member.magicPoint-=damageAmount;
    if(0<member.magicPoint)return;

    member.magicPoint=0;
}
```

### 6.2 전환은 전략 패턴으로 구현하기
플래그 매개변수가 아니라 전략 패턴을 사용합니다.

#### 대미지 인터페이스를 선언하고, 전략 패턴을 적용하여 완성

```java
interface Damage {
    void execute(final int damageAmount);
}

class HitPointDamage implements Damage {
    public void execute(final int damageAmount) {
        member.hitPoint -= damageAmount;
        if (0 < member.hitPoint) return;

        member.hitPoint = 0;
        member.addState(StateType.dead);
    }
}

class MagicPointDamage implements Damage {
    public void execute(final int damageAmount){
        member.magicPoint-=damageAmount;
        if(0<member.magicPoint)return;

        member.magicPoint=0;
    }
}

enum DamageType {
    hitPoint,
    magicPoint
}

final Map<DamageType, Damage> damages;
damages.put(DamageType.hitPoint, new HitPointDamage());
damages.put(DamageType.magicPoint, new MagicPointDamage());

void applyDamage(final DamageType damageType, final int damageAmount) {
    final Damage damage = damages.get(damageType);
    damage.execute(damageAmount);
}
```

플래그 매개변수를 사용한 처음의 메서드는 최종적으로 다음과 같이 변경되었습니다.

```java
// 플래그 매개변수를 사용한 처음의 코드
damage(true, damageAmount)

// 최종 코드
applyDamage(DamageType.hitPoint, damageAmount)
```

처음의 Bad 로직과 비교해보면 조건 분기를 사용하지 않아 가독성이 높아졌습니다.

또한 이렇게 전략 패턴으로 설계하면, 이후에 새로운 대미지가 추가되었을 때도 쉽게 대응할 수 있습니다.
