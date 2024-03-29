# 13장 모델링: 클래스 설계의 토대

동작 원리와 구조를 강단하게 설명하기 위해, 사물의 특징과 관계를 그림으로 나타낸 것을 **모델**, 모델을 만드는 활동을 **모델링**이라고 부릅니다.

## 1. 악마를 불러들이기 쉬운 User 클래스

User 클래스는 사양 변경이 굉장히 잦아서, 여러 가지 문제를 일으키기 쉬운 클래스입니다.

User 클래스는 모델링이 제대로 되지 않기 쉬운 클래스 입니다.

## 2. 모델링으로 접근해야 하는 구조

### 2.1 시스템이란?

시스템의 정의를 찾아보면 '수많은 구성 요소로 이루어진 집합체로서, 각각의 부분이 유기적으로 연결되어, 전체적으로 하나의 목적을 갖고 움직이는 것'이라고 되어 있습니다.

예를 들어 걸어서 목적지로 이동할 때 '이족 보행 시스템'을 사용합니다.<br>
목적지로 이동하기 위해 마차, 자동차, 비행기 등 다른 시스템을 발명했습니다.<br>
**시스템은 목적 달성을 위한 수단**입니다. 기술의 본질은 능력을 확장하는 것입니다.

시스템 중에서 컴퓨터를 활용하는 시스템을 정보 시스템이라고 부릅니다.

### 2.2 시스템 구조와 모델링

시스템 구조를 설명하기 위해 단순한 상자로 도식화한 것이 모델입니다.<br>
그리고 모델의 의도를 정의하고, 구조를 설계하는 것이 모델링입니다.

시스템은 목적을 달성하기 위한 수단입니다. 그리고 모델은 시스템의 구성 요소들입니다.<br>
즉, 모델은 목적을 달성하기 위한 수단의 일부를 개념화한 것입니다.

정리하자면 **모델이란 특정 목적 달성을 위해 최소한으로 필요한 요소를 갖춘 것** 입니다.

### 2.3 소프트웨어 설계와 모델링

온라인 쇼핑몰을 예로 들어 생각해 봅시다.<br>
상품을 모델로 나타내면 어떻게 될까요?<br>
상품에는 다양한 부대 요소(정보)가 있습니다.<br>
상품명, 원가, 판매 가격, 제조년월일, 제조 업체, 구성 부품, 부품 재료, 부품 제조업자, 유통 기한, 소비 기한 등 나열하라면 계속해서 나열할 만큼 많은 정보가 있습니다.<br>
이 정보들을 모두 포함하면, 모델의 목적을 알 수 없을 것입니다.<br>
다루어야 하는 데이터가 폭발적으로 많을 것이고, 구현할 수도 없을 것입니다.

| 상품                                                           |
|--------------------------------------------------------------|
| ID<br>상품명<br>원가<br>판매 가격<br> 제조년월일<br> 제조 업체<br>보증 기간<br>... |

주문 시 상품 모델을 만들기 위해 최소한으로 필요한 요소들을 생각해 봅시다.<br>
상품 ID, 상품명, 판매 가격, 재고 수량 등이 있을 수 있습니다.

배송 시에는 어떨까요? 상품 가격과 재고 수량을 따로 고려하지 않아도 됩니다.<br>
반면에 상품 포장과 관련된 상품 크기와 무게 등의 요소가 필요합니다.

| 주문 시의 상품 모델                 | 
|-----------------------------|
| ID<br>상품명<br>판매 가격<br>재고 수량 |

| 배송 시의 상품 모델    |
|----------------|
| ID<br>크기<br>무게 |

주문과 배송은 달성해야 하는 목적이 다릅니다. 즉, 목적에 따라 상품의 모델이 달라지는 것입니다.

## 3. 안 좋은 모델의 문제점과 해결 방법

모델링 관점에서 User 클래스의 문제점을 검토해 봅시다.

생년월일은 개인 프로필과 관련된 요소입니다.<br>
법인 등록번호는 법인 정보 검증과 관련된 요소입니다.<br>
이메일 주소와 비밀번호는 로그인 인증과 관련된 요소입니다.<br>

즉, User 클래스(모델)는 '여러 목적에 무리하게 사용되고 있으며, 모델링된 것처럼 보이지만 모델링되어 있지 않다'라고 말할 수 있습니다.<br>
이런 모델을 **일관성 없는 모델**이라고 부릅니다.

모델링이 제대로 이루어지지 않으면, User 클래스처럼 여러 가지 문제의 원흉이 됩니다.<br>
모델링을 잘하려면, 반드시 대상이 되는 사회적 활동과 목적을 이해해야 합니다.

### 3.1 User와 시스템의 관계

그렇다면 User 클래스(모델)는 어떻게 모델링해야 할까요?<br>
'User가 대체 무엇인가'<br>
User를 직역하면 사용자
'사용자'는 '무엇'을 사용할까요?<br>
시스템을 사용할 것입니다.<br>
따라서 User는 '시스템 사용자'라고 생각할 수 있습니다.

액터(시스템 사용자)는 시스템 바깥에 있습니다.<br>
정보가 안과 밖에 얽혀 있는 관계를 해소하려면, 자동차와 비행기 등의 물리적인 시스템과 달리 정보 시스템만이 갖는 특징을 활용해야 합니다.

### 3.2 가상 세계를 표현하는 정보 시스템

현실에 있는 자동차와 비행기 등 물리적인 시스템과 크게 다른 점으로<br>
정보 시스템은 현실 세계에 존재하는 개념을 컴퓨터 내부의 가상 세계 안에 만들고, 개념적인 처리를 컴퓨터로 빠르게 만들어 효율을 높일 수 있다는 장점이 있습니다.

### 3.3 목적별로 모델링하기

정보 시스템에서는 '현실 세계에 있는 물리적인 존재'와 '정보 시스템에 있는 모델'이 무조건 일대일 대응되지 않고, 일대다 대응되는 경우가 많다는 특징이 있습니다.<br>
예를 들어, 현실 세계의 사람은 정보 시스템에 '개인 계정', '법인 계정', '프로필', '회사 개요', '직무 경험' 등으로 일대다 대응될 수 있습니다.

목적 중심 이름 설계에서 설명했던 것처럼 구체적이고, 의미 범위가 좁고, 목적을 나타내는 이름으로 다시 설계하는 것도 좋습니다.<br>
목적 기반으로 이름을 설계해보면 User를 '개인 로그인 인증', '법인 로그인 인증', '프로필' 등 같이 수정할 수 있습니다.

### 3.4 모델은 대상이 아니라 목적 달성의 수단

사용자와 상품 모두 단순한 '대상'으로 해석하면, 여기에 모든 목적이 담깁니다.<br>
데이터는 거대해질 수밖에 없고, 일관성 없는 구조가 됩니다.

모델은 시스템 전체가 아니라 특정한 목적 달성과 관련된 부분만을 추려 표현한, 시스템의 일부입니다.<br>
목적 달성 수단으로 해석해야 제대로 모델링할 수 있습니다.

**목적 중심으로 이름을 잘 설계하면, 목적을 달성하기에 적절한 모델을 설계할 수 있습니다.**

### 3.5 단일 책임이란 단일 목적

단일 책임 원칙은 단일 목적 원칙이라고 바꿔 이야기할 수도 있을 것입니다.<br>
'클래스가 이루어야 하는 목적은 반드시 하나여야 한다'라는 것입니다.

클래스를 '공통으로 사용 가능한 범용적인 부품으로 설계해야 한다'고 생각하는 분도 있을 것입니다.<br>
하지만 반대입니다. **특정 목적에 특화되게 설계해야, 변경하기 쉬운 고품질 구조를 갖게 됩니다.**

### 3.6 모델을 다시 확인하는 방법

클래스 구조에 문제가 있다는 것은 모델에 문제가 있다는 말입니다.<br>
모델에 문제가 있는지 확인하려면 다음 사항을 검토해 보세요.

- 해당 모델이 달성하려는 목적을 모두 찾아냅니다.
- 목적별로 모델링을 다시 수정합니다.
- 목적 중심 이름 설계를 기반으로 모델에 이름을 붙입니다.
- 모델에 목적 이외의 요소가 들어가 있다면 다시 수정합니다.

### 3.7 모델과 구현은 반드시 서로 피드백하기

모델은 구조를 단순화한 것에 불과하므로, 모델을 기반으로 클래스를 설계하고, 코드를 구현하면서 세부적인 내용을 수정해야 합니다.

'모델 = 클래스'는 아닙니다. 일반적으로 모델 하나는 여러 개의 클래스로 구현됩니다.

클래스 설계와 구현에서 무언가를 깨닫는다면, 이를 모델에 피드백해야 합니다.<br>
피드백하면 모델이 더 정확해집니다.<br>
피드백하지 않으면, '모델의 구조'와 '소스 코드' 사이에 괴리가 발생합니다.<br>
피드백 사이클을 계속 돌리는 것이 설계 품질을 높이는 비결입니다.

## 모델링의 중요성(User 클래스)

웹 서비스는 대부분의 유스케이스에서 사용자 인증을 활용합니다.<br>
그래서 여러 클래스에서 사용자 인증과 관련된 클래스에 의존하게 됩니다.<br>
의존하는 클래스 수가 많을수록 영향 범위가 넒어지므로, 사용자 인증과 관련된 클래스의 변경이나 유지 보수가 어려워집니다.

개발이 진행되면, 클래스 구조가 복잡해지면서 서로에 대한 의존도가 높아집니다.<br>
이렇게 되면 이후에 모델링을 다시 하고 구조를 정리하려 마음먹어도, 이미 의존도가 높아져서 주눅이 들기 쉽습니다.

개발 초기부터 목적에 특화되게 모델링하는 것이 좋습니다.

## 4. 기능성을 좌우하는 모델링

기능성이란 소프트웨어의 품질 특성 중 하나로, '고객의 니즈를 만족하는 정도'를 의미합니다.

### 4.1 숨어 있는 목적 파악하기

상품 구매는 법적으로 매매 계약입니다. 법적으로 생각해 보면, 무게가 완전히 달라집니다.<br>
기능을 제대로 발휘하려면, '개념의 정체'와 '뒤에 숨어 있는 중요한 목적'을 잘 파악해야 합니다.

### 4.2 기능성을 혁신하는 '깊은 모델'

뛰어난 변환 능력을 갖춘 모델을 설계하는 것이 곧 기능성의 혁신으로 이어진다고 생각합니다.<br>
도메인 주도 설계에서는 이처럼 '본질적 과제를 해결하고, 기능성 혁신에 공헌하는 모델'을 **깊은 모델**이라고 부릅니다.<br>
설계는 한 번 했다고 끝나는 것이 아니라, 매일매일 반복해서 개선하는 것이 중요합니다.
