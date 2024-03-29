## 1. 커뮤니케이션

### 커뮤니케이션이 부족하면 설계 품질에 문제가 발생

팀 개발에서는 팀원과 작업할 때, 로직이 제대로 맞물리지 않아 버그가 발생하는 상황이 흔하다.
이는 팀원간의 커뮤니케이션이 부족하기 때문이다.

### 콘웨이 법칙

콘웨이 법칙(Conway’s law)는 ‘시스템의 구조는 그것을 설계하는 조직의 구조를 닮아 간다’라는 법칙이다.(개발 부문이 3개의 팀으로 구분되어 있다면 모듈의 수도 팀의 수와 동일하게 3개로 구성되는 시스템이 만들어진다.)

팀 내부에서 이뤄지는 커뮤니케이션은 비용이 낮고, 팀 외부와의 커뮤니케이션은 비용이 높다는 비용 구조 자체가 시스템 구조에 영향을 준다.

최근에는 반대로 접근하는 역콘웨이 법칙이 등장했다. 역콘웨이 법칙이란 ‘소프트웨어의 구조를 먼저 설계하고, 이후 소프트웨어의 구조에 맞게 조직을 편성한다'는 접근 방법이다.

### 심리적 안정성

팀 관계 개선에서는 심리적 안정성이 중요하고, 심리적 안정성이란 ‘어떤 발언을 했을 때, 부끄럽거나 거절당하지 않을 것이라는 확신을 느낄 수 있는 심리 상태’, ‘안심하고 자유롭게 발언 또는 행동할 수 있는 상태’등으로 정의 한다.

커뮤니케이션에 문제가 있을 때는 일단 심리적 안정성 향상에 힘쓰는 것이 좋다.

## 2. 설계

### ‘빨리 끝내고 싶다’는 심리가 품질 저하의 함정

일이 바쁘면 구현을 빨리 끝내고 싶은 마음이 앞서게 되고, 그냥 동작하기만 한다면 코드를 어떻게든 구현해 버린다. 이러한 환경에서는 클래스 다이어그램조차 그려보지 않는 등 설계 품질을 무시하기 쉽다.

소프트웨어는 한 번 만드는 것으로 끝나지 않는다. 이후에 사양 변경이 계속 이루어지고, 기능도 점점 확장될 것이다.

시간이 지날수록 구현 속도가 느려지고 버그 수정을 하게될 것이다.

### 나쁜 코드를 작성하는 것이 좋은 코드를 작성하는 것보다 오래 걸린다.

‘테스트 주도 개발(TDD)를 사용해서 구현하는 경우 와 ‘사용하지 않고 구현하는 경우’ 중에 TDD를 사용해서 개발하는 편이 더 빠르다는 결론이 나왔다.

### 클래스 설계와 구현 피드백 사이클 돌리기

사양을 변경할 때는 최소한 메모로라도 클래스 다이어그램을 그리고,구현하다 변경사항이 생기면 이를 반영하여 사이클을 돌리다 보면 설계 품질이 향상될 것이다.

### 한 번에 완벽하게 설계하려고 들지 말고, 사이클을 돌리며 완성하기

사양을 대규모로 변경할 때는 확실한 클래스 설계가 필요하다. 한 번에 완벽하게 설계하려는 욕심을 버리는 것을 권장한다.

**단 한 번의 설계로 완벽한 구조를 만들어 낼 수는 없다. 설계 품질은 설계와 구현 피드백 사이클을 계속해서 돌리면 조금씩 향상될 것이다.**

### ‘성능이 떨어질 수 있으니 클래스를 작게 나누지 말자’는 맞는 말일까?

‘클래스 인스턴스 생성은 비용이 발생하여 성능을 떨어뜨릴 수 있으므로 클래스를 많이 만들면 안된다’라고 생각하는 사람이 많다.

클래스가 많아지면 비용이 발생하는 것은 맞으나 무시할 수 있는 정도이다.

성능에 지배적인 영향을 미치는 부분은 실제로 측정해 보기 전까지는 제대로 알 수 없다.
**병목이 어디인지 모른 채 성능이 빠른 코드를 작성하려고만 하는 것은 너무 빠른 최적화**라는 안티 패턴에 해당한다.

### 설계 규칙을 다수결로 결정하면 코드 품질은 떨어진다.

코드 품질을 향상시키기 위해 코딩 규칙과 설계 규칙을 정하는 경우가 있는데, 이때 다수결을 따르면 일반적으로 결과가 좋지 않다. 다수결 또는 만장일치로 코드와 설계를 결정하려고 하면, 수준이 낮은 쪽에 맞춰서 하향평준화가 되기 쉽기 떄문이다.

### 설계 규칙을 정할 때 중요한 점

설계 규칙에는 이유와 의도를 함께 적는 것이 좋다. 규칙이 형식만 강제하고, 아무런 의미를 갖지 않는 상황을 막기 위해서이다.

| 규칙                                                                                                 | 이유와 의도                                                          |
| ---------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| 중첩을 3번 이하로 해야한다. 중첩의 깊이가 깊어질 경우, 조기 리턴을 활용해서 줄일 수 없는지 검토한다. | 가독성을 높일 수 있다.                                               |
| 같은 조건 분기가 여러 개 구현되면, 인터페이스 설계를 검토한다.                                       | 수정 누락의 가능성이 높아진다.                                       |
| 클래스와 메서드의 이름은 목적을 표현하는 형태로 작성한다.                                            | 목적을 모르면 로직이 난잡해질 수 있고, 유지보수와 변경이 힘들어진다. |

설계 규칙의 의도가 한 번에 전달되기는 힘들다. 리뷰와 스터디를 통해서 의도를 계속해서 전달하면, 다른 구성원들도 조금씩 의도를 이해하게 될 것이다.

## 3. 구현

### 깨진 유리창 이론과 보이스카우트 규칙

**깨진 유리창 이론**

1. 건물에 깨진 유리창 하나가 생긴다.
2. 꺠진 유리창이 오래 방치되면, 아무도 신경쓰지 않는 건물이라는 인식이 퍼진다.
3. 다른 창문을 깨기도 하고, 쓰레기를 버리는 등의 경범죄가 추가로 발생해서, 치안이 조금씩 나빠진다.
4. 상황이 더욱 악화되어 흉악 범죄까지도 일어나게 된다.

이는 나쁜 코드를 방치하게 되면 점점 더 무질서해지게 된다.

**보이스카우트**에는 ‘캠핑장을 자신이 왔을 때보다 더 꺠끗하게 치우고 가기’ 라는 규칙이 있다.

코드를 변경할 때, 변경하기 전보다 더 꺠끗한 상태로 만들어 커밋하는 것이다.

### 기존의 코드를 믿지 말고, 냉정하게 파악하기

많은 사람이 조악한 코드를 보고도, 특별한 의심 없이 따라 하는 경우가 많다. 레거시 코드를 박멸하려면, 기존의 코드를 맹신하지 않는 마음가짐이 중요하다.

이상적인 설계를 처음부터 다시 설계(정체를 파악하는 행위)할 때 넘어야 할 장애물이 몇가지 있다.

첫번쨰, 앵커링 효과라고 하는 심리 작용이다. 앵커링 효과란 처음 제시한 수치와 정보가 기준이 되어, 이후의 판단을 왜곡하는 인지 편향을 의미한다. 기존의 클래스 이름과 메서드 이름이 기준이 되어서, 개발자 판단을 왜곡시키는 경우가 굉장히 많다.

두번쨰, 이름이 없거나, 이름을 모르는 것은 인지하기 어렵다는 것이다.

예를 들어 ‘매매 계약’ 이라는 이름이나 개념을 모르면 관련된 조건의 존재도 알기 힘들다.
따라서 ‘해결하고 싶은 내용’과 ’달성하고 싶은 목적’ 을 배워야 한다.

### 코딩 규칙 사용하기

프로그래밍 언어는 다양한 방법으로 작성이 가능해서 변수의 이름을 어떻게 할지, ‘들여쓰기를 공백 몇 개로 할것인지’, ‘{ 앞 줄바꿈’등 자유롭게 지정이 가능하다.

코딩 규칙을 잘 지켜서 코드의 구조와 이름에 질서가 생기고, 읽기 쉽게 한다.

Prettier와 eslint의 도움을 받으면 된다.

### 명명 규칙

명명 규칙이란 변수 이름, 클래스 이름, 메서드 이름을 정하는 규칙이다.

| 요소   | 규칙                            | 예              |
| ------ | ------------------------------- | --------------- |
| 클래스 | 어퍼 카멜 케이스                | Customer        |
| 메서드 | 로어 카멜 케이스                | payMone         |
| 상수   | 모두 대문자, 구분자로 \_를 사용 | MAX_NAME_LENGTH |

코딍 규칙은 팀마다 다르므로 팀 전체에서 통일된 규칙을 정하고, 이를 활용해 가독성을 높이는 것이 중요하다.

## 4. 리뷰

### 코드 리뷰 구조화하기

코드 리뷰를 하는 습관 자체가 없는 경우가 많은데, 이는 누구의 확인도 받지 않고 병합된다. 깃허브에는 다른 팀원이 승인한 풀 리퀘스트만 병합할 수 있는 기능이 있다. 풀 리퀘스트한 코드는 ‘코드의 히스트뢰와 경위를 알고 있는 사람’ 또는 ‘설계를 자세하게 알고 있는 사람’이 리뷰하는 것이 좋다.

### 코드를 설계 시점에 리뷰하기

대부분 코드 리뷰를 ‘로직이 기능 요건을 만족하는지, 결함이 존재하는지, 코딩 스타일을 지키고 있는지 리뷰하는 것’이라고 생각한다. 그러나 이보다 설계적 타당성을 중심으로 리뷰해야 한다.

### 존중과 예의

코드 리뷰 시 ‘기술적 올바름’을 두고 공격적 코멘트를 허용해서는 안된다. 이러한 리뷰는 사람에게 상처를 주고, 생산성을 저하시키는 것은 물론, 코드를 좋게 만든다는 본래의 목표도 저해한다.

코드 리뷰에서 중요한 것은 존중과 예의이다. 존중과 예의를 갖추고 지적하는 것이 코드 품질을 높이는 가장 빠른 길이다.

### 정기적으로 개선 작업 진행하기

구현 또는 코드 리뷰 중간에 좋지 않은 코드를 발견했는데 ‘나중에 고치자’하고 넘어가곤 한다.

이렇게 넘어간 결함은 대부분 대책 없이 방치된다. 왜냐하면 새로운 업무가 계속해서 할당되어 정신이 팔려있기 떄문이다.

좋지 않은 코드에 대처하는 일은 작업 관리도구에 개선 작업으로 추가해 두고, 정기적으로 이러한 작업을 모아 개선 작업을 진행해서 확실하게 대처할 수 있게 만드는 것이 좋다. 이러한 도구로 깃허브의 이슈등을 활용하면 좋다.

## 5. 팀의 설계능력 높이기

### 영향력을 갖는 규모까지 동료 모으기

설계뿐만 아니라 일의 방식을 상향식으로 개선하려면, 주위의 협력이 반드시 필요하다.
시장 점유율의 목표를 정의하는 쿠프만 목표라는 것이 있는데 쿠프만 목표치는 점유율 10.9% 이다.

이를 팀에 적용해서 생각해보면 팀이 20명 정도일때, 나 뺴고 동료를 한명만 끌어들여도 진행할 수 있다는 이야기이다.

### 천리길도 한 걸음부터

매일매일 조금씩 설계 지식을 공유하도록 하고 동료가 설계에 관심이 더 있다면, 조금 더 많은 내용을 공유하고 함께 의논해보는 것도 좋다.

### 백문이 불여일견

‘백문이 불여일견’이라는 말처럼 말로 설명을 듣는 것보다, 실제로 보고 실감할 떄 확실하게 알 수 있다. 개선전과 개선후를 비교해 보면 가독성이 좋아졌음을 실감할 수 있다.

### 팔로우업 스터디 진행하기

이전에 언급했던 것처럼 스터디를 진행할 때도 실제 코드를 개선해 보는 것이 효과적이다.(인풋 보다 아웃풋이 학습 효과가 높다.)

1. 책에 적혀 있는 노하우를 1~2개 읽어본다.
2. 프로덕션 코드에서 노하우를 적용해 볼 수 있는 부분을 찾는다.
3. 노하우를 사용해 코드를 개선해본다.
4. 어떻게 개선했는지 비포&애프터를 비교할 수 있게 발표한다.
5. 발표 내용에 대해 질의 응답고 논의를 한다.

### 스터디 그룹에서 발생할 수 있는 문제 해결 노하우

단순하게 책을 읽기만 하는 스터디는 추천하지 않는다. 한 번에 많이 인풋해도, 사람들은 업무가 바빠 관련된 애용을 금방 잊어버린다. 즉, 코드를 개선해서 실제로 좋아졌음을 느끼는 것이 중요하다.

스터디에서도 설계 관련 기술을 무리하게 전달하려 하지 말고, 차근차근 전달하며 시작하는 것이 좋다.

### 리더와 매니저에게 설계의 중요성과 비용 대비 효과 설명하기

변경 용이성을 지속적으로 개선하지 못하고 오히려 저하하기만 하는 조직은 대부분 애초에 개발 리소스에 변경 용이성과 관련된 설계비용이 포함되어 있지 않다. 그 이유는 예산과 계획을 결정하는 사람이 변경 용이성과 관련된 지식을 아예 갖고 있지 않기 때문이다.

조직 차원에서 설계 품질을 향상시키려면, 개발 프로세스 흐름에 설계를 추가하고 그 필요성도 공유해야 한다.

개바리 효율 저하 문제를 이야기하고 이런 문제를 해결할 수 있는 설계 임무가 있음을 알리고 설득해라.

### 설계 책임자 세우기

개발 팀원 대부분이 설계 품질을 좋게 만드는 데 적극적이라면, 품질 향상 작업이 자연스럽게 이루어진다. 그러나 그렇지 않은 경우 설계 책임자를 내새우는 것이 좋다.

설계 책임자는 변경 용이성 품질 향상을 위해 다음과 같은 내용을 추진한다.

-   설계 품지과 관련된 규칙이나 개발 프로세스 수립
-   규칙을 반복적으로 알리고 교육
-   리더와 매니저에게 효과 공유
-   품질 시각화
-   설계 품질 유지

그렇다면 누가 설계 책임자가 되는 것이 좋을까?

팀내 적당한 인물이 없다면 지금 이 책을 익는 내가 책임자가 되어야 할 수 도 있다.(네???)

책을 읽고 있는다는 것이 설계와 관련된 문제를 느끼고 위기의 식을 갖고 있는 증거라고 생각한다(네2???)
