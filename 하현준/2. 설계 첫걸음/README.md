# 2장 설계 첫걸음

### 1. 의도를 분명히 전달할 수 있는 이름 설계하기

변수의 이름은 코드의 의도에 맞는 이름을 붙여 읽고 이해하기 쉽게 만들도록 한다.

```java
// BAD
int d = 0;
d = p1 + p2;
d = d - ((d1 + d2) /2);
if (d < 0) {
   d = 0;
}

// GOOD
int damageAmount = 0;
damageAmount = playerArmPower + playerWeaponPower;
damageAmount = damageAmount - ((enemyBodyDefence + enemyArmorDefence) / 2);

if(damageAmount < 0) {
   damageAmount = 0;
}
```

```javascript
// BAD
let d = 0;
d = p1 + p2;
d = d - ((d1 + d2) /2);
if (d < 0) {
   d = 0;
}

// GOOD
let damageAmount = 0;
damageAmount = playerArmPower + playerWeaponPower;
damageAmount = damageAmount - ((enemyBodyDefence + enemyArmorDefence) / 2);

if(damageAmount < 0) {
   damageAmount = 0;
}
```

### 2. 목적별로 변수를 따로 만들어 사용하기

위 코드에서는 `damageAmount` 라는 변수에 여러번 **재할당**해서 사용되는 코드가 존재한다.

이를 목적에 맞는 변수를 선언해 코드를 변경할 수 있다.

```java
int totalPlayerAttackPower = playerArmPower + playerWeaponPower;
int totalEnemyDefence = enemyBodyDefecne + enemyArmorDefence;

int damageAmount = totalPlayerAttackPower - (totalEnemyDefence / 2);

if(damageAmount < 0) {
   damageAmount = 0;
}
```

```tsx
const totalPlayerAttackPower = playerArmPower + playerWeaponPower;
const totalEnemyDefence = enemyBodyDefecne + enemyArmorDefence;

const damageAmount = totalPlayerAttackPower - (totalEnemyDefence / 2);

if(damageAmount < 0) {
   damageAmount = 0;
}
```

### 3. 단순 나열이 아니라, 의미 있는 것을 모아 메서드로 만들기

일련의 흐름으로 나열해서 로직을 작성하는 것이 아니라 의미 있는 로직을 메서드로 만들어서 사용하면 더 좋은 코드로 구현이 가능하다.

```java
int sumUpPlayerAttackPower(int playerArmPower, int playerWeaponPower) {
   return playerArmPower + playerWeaponPower;
}
int sumUpEnemyDefence(int enemyBodyDefence, int enemyArmorDefecne) {
   return enemyBodyDefecne + enemyArmorDefence;
}
int estimateDamage(int totalPlayerAttackPower, int totalEnemyDefence) {
   int damageAmount = totalPlayerAttackPower - (totalEnemyDefence / 2);
   if(damageAmount < 0) {
     damageAmount = 0;
   }
   return damageAmount;
}

// 사용하는 코드
int totalPlayerAttackPower = sumUpPlayerAttackPower(playerArmPower, playerWeaponPower);
int totalEnemyDefence = sumUpEnemyDefence(enemyBodyDefecne, enemyArmorDefence);
int damageAmount = estimateDamage(totalPlayerAttackPower, totalEnemyDefence);
```

```tsx
function sumUpPlayerAttackPower(playerArmPower: number, int playerWeaponPower: number) {
   return playerArmPower + playerWeaponPower;
}
function sumUpEnemyDefence(enemyBodyDefence: number, enemyArmorDefecne: number) {
   return enemyBodyDefecne + enemyArmorDefence;
}
function estimateDamage(totalPlayerAttackPower: number, totalEnemyDefence: number) {
   const damageAmount = totalPlayerAttackPower - (totalEnemyDefence / 2);
   if(damageAmount < 0) {
     damageAmount = 0;
   }
   return damageAmount;
}

// 사용하는 코드
const totalPlayerAttackPower = sumUpPlayerAttackPower(playerArmPower, playerWeaponPower);
const totalEnemyDefence = sumUpEnemyDefence(enemyBodyDefecne, enemyArmorDefence);
const damageAmount = estimateDamage(totalPlayerAttackPower, totalEnemyDefence);
```

### 4. 관련된 데이터와 로직을 클래스로 모으기

서로 밀접한 데이터와 로직을 한곳에 모아서 관리하면 유지보수도 쉬워지고 코드를 이해하기도 좋다.

```tsx
class HitPoint {
  private static readonly MIN = 0;
  private static readonly MAX = 999;
  private value: number;

  constructor(value: number) {
     if(value < HitPoint.MIN) throw new Error(HitPoint.MIN + " 이상을 지정해주세요.");
     if(HitPoint.MAX < value) throw new Error(HitPoint.MAX + " 이하를 지정해주세요.");
	 this.value = value;
  }

  private damage(damageAmount: number) {
     const damaged = this.value - damageAmount;
     const corrected = damaged < HitPoint.MIN ? HitPoint.MIN : damaged;
     return new HitPoint(corrected);  
	}

  private recover(recoveryAmount: number) {
      const recovered = this.value + recoveryAmount;
      const corrected = HitPoint.MAX < recovered ? HitPoint.MAX : recovered;
      return new HitPoint(corrected);
  }
}
```

**[예제 코드](https://www.typescriptlang.org/play?#code/MYGwhgzhAEASCWAXACge3gO0dA3gKGmgAcAneANzEQFNoJEr5hoTqwATVDEAT2gFkAkgDloAXmgAGANwFiZSjToNETFm07c+-AIIANcdACcJ2YVIUqtSiACu1AFzQMtgLYAjaiVlzgXeiS2wIioJAAUNvZOLh5eAJS4coTQ8ABmEWB2tAA8cEhomIgAdELCCYgAFiSoAO7O1HUAoiTV4Qgo6FglItAA1NAARNCALuOAgwOAIuPQgAOTgKgTgC6rgD8TgByDgCljRQNxZskp6e0FXboGuZHU5VW19U0toWF7ncWHfYMjgBqrgD6d0-PLaxuyAJDQSrwCBFE6GE5bAC+eDkFkUtHYYFcYAA5tQwojkWidK5ULYsNE3J4SAl8NtoH4MPRoJjUdR2IYgSCwQBaGlIuk4vFYLbJSnUvwtajBemGWlohm5O6FbqiAD8eQ6MtK0Cc4vpvMIrEQthIGEuiv2iDCgtYIvYm0IeD+0NhCis6j85C8YTNqGdJB4XPxiEJsRJiXJFP82DdHtFEiZoMy9ieYa8XtxPs1hH52FNwpoDIk0oO+mguXjrAZCtzD3zTiLGqSyW1uv1GAahvuJtCZqzmzk0OhQA)**