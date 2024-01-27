## 1. 악마를 불러들이는 이름

이름 설계가 부적절해 악마를 불러들이는 경우 (강한 결합 구조를 갖게 된다.)

예시) 상품을 그대로 ‘상품 클래스’ 라고 이름을 붙이는 것

→ 상품 클래스가 관련 있는 여러 로직을 갖게 되어 점점 거대하고 복잡해진다. 변경사항이 생기면 상품 클래스와 관련된 클래스를 모두 확인해봐야한다.

### 관심사 분리

상품 클래스는 **관심사에 따라서 각각 클래스로 분할**하는 **관심사 분리(seperation of concerns)**를 할 수 있어야 한다.

### 포괄적이고 의미가 불분명한 이름

‘상품’ 이라는 이름이 너무 포괄적이라, 상품과 관련된 모든 로직을 구현하면 될 것 처럼 보이게 된다.

‘상품’ → 예약, 주문, 발송 등 다양한 목적으로 사용될 수 있는 **포괄적인 이름**이다.

이름이 너무 포괄적이라서 목적이 불분명한 클래스를 **목적 불명 객체**라고 부른다.

## 2. 이름 설계하기 - 목적 중심 이름 설계

관심사 분리를 생각하고, 비지니스 목적에 맞게 ‘이름을 붙이는 것’은 결합이 느슨하고 응집도가 높은 구조를 만드는데 중요한 역할을 한다.

목적에 맞게 이름을 설계하는데 중요한 점은 다음과 같다.

-   **최대한 구체적이고, 의미 범위가 좁고, 특화된 이름 선택하기**
    -   특정한 목적을 달성하는데 특화된 의미 범위가 좁은 이름을 클래스에 붙인다.
-   **존재가 아니라 목적을 기반으로 하는 이름 생각하기**
    -   목적 기반으로 생각한 이름의 예
        | 존재 기반   | 목적 기반                                          |
        | ----------- | -------------------------------------------------- |
        | 주소        | 발송지, 배송지, 업무지                             |
        | 금액        | 청구 금액, 소비세액, 연체 보증료, 캠페인 할인 금액 |
        | 사용자      | 계정, 개인 프로필, 직무                            |
        | 사용자 이름 | 계정 이름, 닉네임, 본명, 법인명                    |
        | 상품        | 입고 상품, 예약 상품, 주문 상품, 발송 상품         |
-   **어떤 관심사(비지니스 목적)가 있는지 분석하기**
    -   소프트웨어가 추구하는 목적과 내용을 분석하여 비지니스 목적에 특화된 이름을 만들기
-   **소리 내어 이야기해 보기**
    -   어떤 목적을 달성하고 싶은지, 어떤 형태로 사용하고 싶은지, 서로 어떤 관련이 있는지 등 배경과 의도를 함께 정리하고, 팀과 소통해서 일치시키는 것이 중요
    -   고무 오리 디버깅: 어떤 문제가 발생했을 때, 책상 위의 오리 인형에게 문제를 처음부터 차근차근 설명해 보면서 해결하는 방법
-   **이용 약관 읽어 보기**
    -   서비스와 관련된 규칙이 굉장히 엄격한 표현으로 작성되어 있다. 이를 활용해 비지니스와 관련된 이름을 알 수 있다.
    -   서비스 사용료 클래스
        ```tsx
        class ServiceUsageFee {
            readonly amount: number;

            constructor(amount: number) {
                if (amount < 0)
                    throw new Error("금액은 0 이상의 값을 지정하세요.");
                this.amount = amount;
            }

            static determine(
                salesPrice: SalesPrice,
                salesCommissionRate: SalesCommissionRate
            ) {
                let amount = parseInt(
                    salesPrice.amount * salesCommissionRate.value
                );
                return new ServiceUsageFee(amount);
            }
        }
        ```
        ‘매매 계약이 체결되면, 판매자는 당사에 서비스 사용료를 지불해야 한다’ 라는 규약과 일치하게 된다.
        즉, 이용 약관과 실제 로직에 일관성이 생긴다.
        서비스 사용료 변경 → ServiceUsageFee 수정, 판매 수수로율 변경 → SalesCommissionRate 수정
        비지니스 규칙과 클래스를 일치하게 만들면, 정확하고 빠르게 변경이 가능
-   **다른 이름으로 대체할 수 없는지 검토하기**
    -   정해진 이름의 의미 범위가 생각보다 넓을 수 있고 여러 의미가 내포된 이름일 수 있다.
    -   다른 이름으로 바꿔보고, 의미를 더 좁게 만들 수 없는지, 이상한 점은 없는지 검토
    -   ex) 호텔 숙박 예약 시스템 → ‘고객’(x) , ‘숙박하는 사람’/’결제하는 사람’ (o)
-   **결합이 느슨하고 응집도가 높은 구조인지 검토하기**
    -   목적에 특화된 이름을 선택하면, 목적 이외의 로직을 배제하기 쉬워진다.
    -   목적과 관련된 로직이 모여 응집도가 높아진다.

## 3. 이름 설계 시 주의 사항

### 이름에 관심 갖기

‘**이름에 주의를 기울이고, 이름과 로직을 대응시킨다’** 라는 접근 방법을 전제로 목적 중심 이름 설계를 해야한다.

### 사양 변경 시 ‘의미 범위 변경’ 경계하기

개발 초기에는 고객 클래스가 있었고, 이는 ‘개인 고객’ 을 나태는 것이었다. 이후에 사양이 변경되어 ‘법인 고객’도 포함이 되었다. 이렇게 여러 의미가 섞이면, 이름이 의미하는 바를 다시 검토해 봐야 한다.

### 대화에는 등장하지만 코드에 등장하지 않는 이름 주의하기

```
사원 A: 이전에 이야기했던 '문제가 있는 회원'은 구현했나요?
사원 B: 네, 이미 구현했습니다.
사원 A: 어라? 어떤 클래스인가요?
사원 B: User 클래스 안에 구현했습니다.
사원 A: User 클래스가 '문제가 있는 회원'을 나타내는 건가요?
사원 B: 아뇨, 인스턴스 변수 '대여 연체 횟수'와 '도서 파손 횟수'가 일정 횟수 이상이면 User 클래스를
			'문제가 있는 회원'으로 구분하도록 만들었습니다.
사원 A: 하지만 소스 코드 어디에도 '문제가 있는 회원'과 관련된 이름이 등장하지 않는 걸요?
//...
```

대화에 자주 등장하는 중요한 개념이 **소스 코드에서는 이름조차 붙어 있지 않고**, 잡다한 로직에 묻혀 있는 경우가 꽤 많다.

‘이름 없는 로직’ 은 메서드 또는 클래스로 설계되어 있지 않다는 의미고 소스코드 내부에 사양대로 동작하게만 마구잡이로 작성된다.

**대화에서 등장하는 이름을 기반으로 메서드와 클래스를 설계해야 한다.**

### 수식어를 붙여서 구별해야 하는 경우는 클래스로 만들어 보기

‘최대 히트 포인트’ 를 높여주는 장비들이 있을때, 액세서리에 최대 히트 포인트 증가 효과가 있다고 가정하자

```tsx
let maxHitPoint = member.maxHitPoint + accessory.maxHitPoinatIncrements();
```

이후 방어구에도 같은 효과가 추가 된다. 신입 사원이 들어와 이 코드를 작성한다.

```tsx
maxHitPoint = member.maxHitPoint + armor.maxHitPointIncrements();

// 동작하는 코드
let maxHitPoint =
    member.maxHitPoint +
    accessory.maxHitPoinatIncrements() +
    armor.maxHitPointIncrements();
```

이 코드는 정상적으로 동작하지 않는다.

그 이유는 ‘캐릭터의 원래 최대 히트포인트’ 와 ‘장비 착용으로 높아진 최대 히트포인트’를 `maxHitPoint` 의 이름만 보고 알 수 없기 때문이다.

-   캐릭터의 원래 최대 히트포인트: `originalMaxHitPoint`
-   장비 착용으로 높아진 최대 히트포인트: `correctedMaxHitPoint`

이름을 각각 붙이고 클래스도 별도로 설계하여 구조화 해야 한다.

```tsx
// '캐릭터의 원래 최대 히트포인트'를 나타내는 클래스
class OriginalMaxHitPoint {
    private static readonly MIN = 10;
    private static readonly MAX = 999;

    readonly value: number;

    constructor(value: number) {
        if (
            value < OriginalMaxHitPoint.MIN ||
            OriginalMaxHitPoint.MAX < value
        ) {
            throw new Error();
        }
        this.value = value;
    }
}

// '장비 착용'으로 높아진 최대 히트포인트'를 나타내는 클래스
class CorrectedMaxHitPoint {
    readonly value: number;

    constructor(
        originalMaxHitPoint: OriginalMaxHitPoint,
        accessory: Accessory,
        armor: Armor
    ) {
        this.value =
            originalMaxHitPoint.value +
            accessory.maxHitPointIncrements() +
            armor.maxHitPointIncrements();
    }
}
```

의미가 다른 개념들은 서로 다른 클래스로 설계해 구조화해야 한다.

## 4. 의미를 알 수 없는 이름

```tsx
let tmp3 = tmp1 - tmp2;
if (tmp3 < tmp4) {
    tmp3 = tmp4;
}
let tmp5 = tmp3 * tmp6;
return tmp5;
```

계산 결과를 임시로 저장하기 위한 지역변수를 tmp로 만든 경우가 있는데, **이름만 보고 목적이 무엇인지 알기 굉장히 힘들다.**

### 기술 중심 명명

이름을 프로그래밍과 관련된 용어, 컴퓨터와 관련된 용어에서 유래되는 경우가 많은데, 이를 기반으로 이름 짓는 방법을 **기술 중심 명명**이라고 부른다.

_기술 중심 명명의 예_

| 종류                 | 예                                 |
| -------------------- | ---------------------------------- |
| 컴퓨터 기술 유래     | memory, cache, thread, register 등 |
| 프로그래밍 기술 유래 | function, method, class, module 등 |
| 자료형 이름 유래     | int, str(string), flag(boolean) 등 |

> **기술 중심 명명을 사용해야 하는 분야**

임베디드 처럼 하드웨어와 가까운 레이어의 미들웨어에서는 메모리와 프로세서 등 하드웨어에 직접 접근하는 로직이 많이 사용된다. 이때는 어쩔수 없이 기술을 중심으로 이름을 짓는다. 최대한 목적과 의도를 전달할 수 있게 지어야 된다.

>

### 로직 구조를 나타내는 이름

```tsx
class Magic {
    // Bad
    isMemberHpMoreThanZeroAndIsMemberCanActAndIsMemerMpMoreThanMagicCostMp(
        member: Member
    ) {
        if (0 < member.hitPoint) {
            if (member.canAct()) {
                if (costMagicPoint <= member.magicPoint) {
                    return true;
                }
            }
        }
        return false;
    }
    // Good
    canEnchant(member: Member) {
        if (member.hitPoint <= 0) return false;
        if (!member.canAct()) return false;
        if (member.magicPoint < costMagicPoint) return false;
        return true;
    }
}
```

메소드의 이름만 가지고는 어떤 로직을 의도하는지 알기 힘들다. 따라서 의도와 목적을 이해하기 쉽게 이름을 붙인다.

### 놀람 최소화 원칙

```tsx
let count = order.itemCount();

class Order {
    private readonly id: OrderId;
    private readonly items: Items;
    private readonly giftPoint: GiftPoint;

    itemCount() {
        let count = this.items.count();

        if (10 <= count) {
            this.giftPoint = this.giftPoint.add(new GiftPoint(100));
        }
        return count;
    }
}
```

주문 상품 수를 리턴하는 것처럼 보이지만 메소드 내용은 그렇지 않다. 실제 하고 있는 일을 깨달으면 **놀랄** 것이다.

**놀람 최소화 원칙(Principle of least astonishment)**: 사용자가 예상하지 못한 놀라움을 최소화하도록 설계해야 한다는 접근 방법

놀람 최소화 원칙을 지키도록 변경해보자

```tsx
class Order {
    private readonly id: OrderId;
    private readonly items: Items;
    private readonly giftPoint: GiftPoint;

    itemCount(): number {
        return this.items.count();
    }

    shouldAddGiftPoint(): boolean {
        return 10 <= itemCount();
    }

    tyryAddGiftPoint(): void {
        if (shouldAddGiftPoint()) {
            this.giftPoint = this.giftPoint.add(new GiftPoint(100));
        }
    }
}
```

## 5. 구조에 악영향을 미치는 이름

### 데이터 클래스처럼 보이는 이름

ProductInfo는 상품 정보를 저장하는 클래스이고 데이터 클래스이다.

```tsx
class ProductInfo {
    id: number;
    name: string;
    price: number;
    productCode: string;
}
```

~Info와 ~Data처럼 데이터만 갖는다는 인상을 주는 이름은 피하는 것이 좋다. 그리고 ProductInfo → Product로 변경해 관련이 깊은 로직들을 캡슐화 하는 것이 좋다.

**DTO(Data Transfer Object)**

예외적으로 데이터 클래스를 사용하는 경우가 있다.

변경 책무와 참조 책무를 모듈로 분리하는 **명령 쿼리 역할 분리(CORS)**라고 불리는 아키텍쳐 패턴이 있다.

단순히 값을 추출해서 출력하면 되므로 계산과 데이터 변경을 동반하지 않는다.

```tsx
class ProductDto {
    readonly id: number;
    readonly name: string;
    readonly price: number;
    readonly productCode: string;

    constructor(name: string, price: number, productCode: string) {
        this.name = name;
        this.price = price;
        this.productCode = productCode;
    }
}
```

이는 DTO로 데이터 전송 용도로 사용되는 디자인 패턴이다.

### 클래스를 거대하게 만드는 이름

대표적으로 Manager가 있다.

```tsx
class MemberManager {
    // 멤버의 히트포인트 추출하기
    getHitPint(memberId: number) {}

    // 멤버의 매직포인트 추출하기
    getMagicPoint(memberId: number) {}

    // 멤버 보행 애니메이션 시작하기
    startWalkAnimation(memberId: number) {}

    // 멤버의 능력치를 CSV 형식으로 내보내기
    exportParamsToCSV() {}

    // 적이 생존해 있는지 확인하기
    enemyIsAlive(enemyId: number) {}

    // BGM 재생하기
    playBgm(bgmName: string) {}
}
```

MemberManager가 너무 많은 책무를 떠안아서 단일 책임 원칙을 위반하고 있다. 이는 ‘관리’ 라는 **단어가 가진 의미가 너무 넓고 애매하기 때문**이다.

또 다른 예로는 Processor와 Controller와 같은 이름도 주의해야 한다.

### 상황에 따라 의미가 달라질 수 있는 이름

예를 들어 ‘Account’는 금융에서는 ‘계좌’, 컴퓨터 보안에서는 ‘로그인 권한’을 의미한다.

상황(컨텍스트)이 달라지면, 자동차와 관련된 개념이 서로 반대가 될 수 있다.

-   배송 컨텍스트: 자동차가 화물로 배송되는 컨텍스트 → 자동차 - 발송지, 배송지, 배송 경로와 관련
-   판매 컨텍스트: 딜러에 의해서 고객에게 판매되는 컨텍스트 → 자동차 - 판매 가격, 판매 옵션과 관련

컨텍스트가 서로 다른데 하나의 Car 클래스에 모든걸 구현하면 **여러 로직을 갖게 되고 클래스가 거대해진다.**

즉, 컨텍스트별로 클래스를 설게해야 한다.

### 일련번호 명명

클래스와 메서드에 이름에 번호를 붙여 만드는 것(Class001, method001)을 **일련번호 명명** 이라고 한다.

**목적과 의도를 알기 힘들기도 하지만 구조를 개선하기가 훨씬 힘들다.**

## 6. 이름을 봤을 때, 위치가 부자연스러운 클래스

### ‘동사 + 목적어’ 형태의 메서드 이름 주의하기

```tsx
class Enemy {
    isAppeared: boolean;
    magicPoint: number;
    dropItem: Item;

    escape(): void {
        this.isAppeared = false;
    }
    consumeMagicPoint(costMagicPoint: number) {
        this.magicPoint -= costMagicPoint;
        if (this.magicPoint < 0) {
            this.magicPoint = 0;
        }
    }

    addItemToParty(items: Item[]) {
        if (items.size() < 99) {
            items.add(dropItem);
            return true;
        }
        return false;
    }
}
```

Enemy 클래스의 관심사는 적이다. 매직 포인트를 다루는 `consumeMagicPoint`는 적의 관심사이다. 그러나 `addItemToParty`는 **주인공의 소지품을 다루기에 관심사가 전혀 상관이 없다.**

관심사가 다른 메서드는 `addItemToParty` 처럼 동사 + 목적어 형태가 되는 경향이 있다.

### 가능하면 메서드의 이름은 동사 하나로 구성되게 하기

관심사가 다른 메서드가 섞이지 못하게 막으려면 되도록 메서드의 이름이 동사 하나로 구성되도록 설계하는 것이 좋다.

```tsx
class PartyItems {
    static readonly MAX_ITEM_COUNT = 99;
    readonly items: Item[];

    constructor() {
        this.items = new Array();
    }

    ㅁㄴㅇㅇ;
}
```

### 부적절한 위치에 있는 boolean 메서드

boolean 자료형을 리턴하는 메서드도 적절하지 않은 클래스에 정의되어 있는 경우가 많다.

```tsx
// Bad
class Common {
    // 멤버가 혼란 상태라면 true를 리턴
    static isMemberInConfusion(member: Member) {
        return member.status.contains(StateType.confused);
    }
}

// Good
class Member {
    private readonly states: States;

    isInConfusion() {
        return this.states.contains(StateType.confused);
    }
}
```

멤버와 관련된 관심사이므로 Common에 정의되어 있는 것은 자연스럽지 않다.

boolean 자료형의 메서드를 추가할 때는 **‘클래스 is 상태’** 형태로 읽어 봤을 때 자연스러운 영어 문장이 되는지 확인해보면 된다.

## 7. 이름 축약

### 의도를 알 수 없는 축약

긴 이름이 싫어서 이름을 축약하는 경우가 있다. `fee` 의 단어로 어떤 요금인지 유추는 할 수 있지만 정확한 이름은 알 수가 없다.

```tsx
let trFee = brFee + LRF * dod;
```

이는 ‘기본 요금 + 연체료 \* 연체일’ 을 계산한 렌탈 요금 총액을 계산하는 계산식이다.

### 기본적으로 이름은 축약하지 말기

이름을 입력하는것이 에디터에서 자동완성해주는 경우가 많으므로 축약하지 말고 사용하자

```tsx
let totalRentalFee = basicRenterFee + LATE_RENTAL_FEE_PER_DAY * daysOverdue;
```

변수명 뿐만 아니라 메서드, 클래스 등에도 모두 축약하지 말고 작성하자. (SNS, VIP 등 관습적으로 축약한 형태는 의미를 전달하는 데 아무 문제가 없기에 사용해도 된다.)

### 이름을 축약할 수 있는 경우

for 문의 카운터 변수는 관습적으로 i, j 처럼 짧은 한글자의 문자로 표현하는 경우가 많다.

**이름을 축약할 때 의미가 사라지지 않는지, 추가적인 다른 문제는 발생하지 않는지 확인해본다.**
프로그래밍 언어에 따라 관례가 다를 수 있으므로, 팀이나 회사 차원에서 결정하는 것이 좋다.
