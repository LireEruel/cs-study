## 22.1 this 키워드

19.1절 객체지향 프로그래밍에서 살펴보았듯, 객체는 상태를 나타내는 프로퍼티와 동작을 나타내는 메서드를 하나의 논리적인 단위로 묶은 복합적인 자료구조다.

동작을 나타내는 **메서드**는 자신이 속한 객체의 상태, 즉 프로퍼티를 참조하고 변경할 수 있어야 한다. 이때 메서드가 **자신이 속한 객체의 프로퍼티를 참조하려면 먼저 자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야 한다.**

( 메서드와 일반 함수는 다르다!)

객체 리터럴 방식으로 생성한 객체의 경우 메서드 내부에서 메서드 자신이 속한 객체를 가리키는 식별자를 재귀적으로 참조할 수 있다.


```js
const circle = {
	radius: 5,
	getDiameter() {
		return 2 * circle.radius
	}
}

console.log(circle.getDiameter()) // 10
```

`getDiameter` 메서드 내에서 자신이 속한 객체를 가리키는 식별자 `circle` 을 참조하고 있다. 이 참조 표현식이 평가되는 시점은 `getDiameter` 메서드가 호출되어 몸체가 실행되는 시점이다.

객체 리터럴은 circle 변수에 할당되기 직전에 평가된다. 따라서 메서드가 호출되는 시점에는 이미 객체리터럴의 평가가 완료되어 객체가 생성되었고  circle 식별자에 생성된 객체가 할당된 이후다. 따라서 메서드 내부에서 ccircle 식별자를 참조할 수 있다.

-> 하지만 이 방식은 이상하므로 this가 등장했다.

---

### this 란?

- **this는 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수(self-referencing variable)다. tihs를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있다.**
- this는 자바스크립트 엔진에 의해 암묵적으로 생성되며, 코드 어디서든 참조할 수 있다.
- **this가 가리키는 값, 즉 this 바인딩은 함수 호출 방식에 의해 동적으로 결정된다.** 


---


```js

// 객체 리터럴
const circle = {
	radius: 5,
	getDiameter() {
		// this는 메서드를 호출한 객체를 가리킨다.
		return 2 * this.radius;
	}
}

console.log(circle.getDiameter());

//  생성자 함수
function Circle(radius){
	// this는 생성자 함수가 생성할 인스턴스를 가리킨다.
	this.radius= radius;
}

Circle.prototype.getDiameter = function () {
	// this는 생성자 함수가 생성할 인스턴스를 가리킨다.
	return 2 * this.radius;
}

// 인스턴스 생성
const circle = new Circle(5);
console.log(circle.getDiameter()); // 10
```

생성자 함수 내부의 this는 생성자 함수가 생성할 인스턴스를 가리킨다. 이처럼 this는 상황에 따라 가리키는 대상이 다르다. 

-  자바와 같은 클래스 기반 언어에서 this는 언제나 클래스가 생성하는 인스턴스를 가리킨다.
-  **자바스크립트의 this는 함수가 호출되는 방식에 따라 this에 바인딩 될 값, 즉 this 바인딩이 동적으로 결정된다.**
- strict mode 역시 this 바인딩에 영향을 준다.



## 22.2 함수 호출 방식과 this 바인딩

###  this 바인딩은 함수 호출 방식에 따라 동적으로 결정된다.

함수를 호출하는 방식
1. 일반 함수 호출
2. 메서드 호출
3. 생성자 함수 호출
4. Function.prototype.apply/call/bind 메서드에 의한 간접 호출


```js
/// this 바인딩은 함수 호출 방식에 따라 동적으로 결정된다.

const foo == function () {
	console.dir(this);
}

// 동일한 함수도 다양한 방식으로 호출할 수 있다.

// 1. 일반 함수 호출
// foo 함수를 일반적인 방식으로 호출
// foo 함수 내부의 this는 전역 객체 window를 가리킨다.
foo();

// 2. 메서드 호출
// foo 함수를 프로퍼티 값으로 할당하여 호출
// foo 함수 내부의 this는 메서드 호출한 객체 obj를 가리킨다.
const obj = { foo };
obj.foo(); // obj

// 3. 생성자 함수 호출
// foo 함수를 new 연산자와 함께 생성자 함수로 호출
// foo 함수 내부의 this는 생성자 함수가 생성한 인스턴스를 가리킨다.
new foo();

// 4. Function.prototype.apply/call/bind 메서드 의한 간접 호출
// foo 함수 내부의 this는 인수에 의해 결정된다.

const bar = { name: 'bar'};

foo.call(bar);
foo.apply(bar);
foo.bind(bar)();
```

함수 호출 방식에 따라 this 바인딩이 어떻게 결정되는지 알아보자.

## 22.2.1 일반 함수 호출

**기본적으로 this에는 전역 객체가 바인딩된다.**

```js
function foo() {
	console.log("foo's this: ", this); // window
	function bar() {
		console.log("bar's this: ", this) // window
	}
	bar();
}
foo();
```

위 예제처럼 전역 함수는 물론이고, 중첩 함수를 **일반함수로 호출하면 함수 내부의 this에는 전역 객체가 바인딩된다.** 다만 객체를 생성하지 않는 일반 함수에는 this는 의미가 없다. 따라서 stict mode에서는 일반 함수 내부의 this에는 undefined가 바인딩된다.

