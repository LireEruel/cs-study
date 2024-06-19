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

**함수를 호출하는 방식**

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

const obj = {
	value : 100,
	foo(){
		console.log(this) // {value : 100, foo: f}
		setTimeout(function() {
				console.log(this)// window
		}, 100)
	}
}

obj.foo();
```

위 예제처럼 전역 함수는 물론이고, 중첩 함수를 **일반함수로 호출하면 함수 내부의 this에는 전역 객체가 바인딩된다.** 다만 객체를 생성하지 않는 일반 함수에는 this는 의미가 없다. 따라서 stict mode에서는 일반 함수 내부의 this에는 undefined가 바인딩된다.

- 메서드 내엔서 정의한 중첩함수도 일반 함수로 호출되면 중첩 함수의  this에는 전역 객체가 바인딩된다.

- 콜백 함수가 일반 함수로 호출된다면 콜백 함수의 내부의 this에도 전역 객체가 바인딩된다. 어떠한 함수라도 일반 함수로 호출되면 this에 전역 객체가 바인딩된다.

- **어떠한 함수라도 일반 함수로 호출되면 this에 전역 객체가 바인딩된다.**

#### **일반 함수로 호출된 모든 함수(중첩 함수, 콜백 함수 포함) 내부의 this에는 전역 객체가 바인딩된다.**


메서드 내부의 중첩 함수나 콜백 함수의 this 바인딩을 메서드의 this 바인딩과 일치시키기 위한 방법

1. this 바인딩 변수에 할당학.

```js
const obj = {
	value : 100,
	foo(){
		const that = this;
		setTimeout(function() {
				console.log(that) // {value : 100, foo: f}
		}, 100)
	}
}
```

- 위 방법 외에도 this를 명시적으로 바인딩할 수 있는 Function.prototype.apply, Function.prototype.call, Function.prototype.bind 메서드가 있다.

2. 명시적으로 this를 바인딩하기

	```js
const obj = {
	value : 100,
	foo(){
		setTimeout(function() {
				console.log(this.value) // 100
		}.bind(this), 100)
	}
}
```

3. 화살표 함수를 사용해서 this 바인딩 일치시키기

```js
const obj = {
	value : 100,
	foo(){
		setTimeout(()=> console.log(this.value), 100); // 100
	}
}
```

### 22.2.2 메서드 호출

메서드 내부의 this에는 메서드를 호출한 객체, 즉 메서드를 호출할 때 메서드 이름 앞의 . 연산자 앞에 기술한 객체가 바인딩된다.

주의 : **메서드 내부의 this는 메서드를 소유한 객체가 아닌 메서드를 호출한 객체에 바인딩 됨.**


```js

const person = {
	name: 'Lee'
	getName() {
		return this.name;
	}
}

const anotherPerson = {
	name: "Kim"
}
const getName = person.getName;

anotherPerson.getName = person.getName;

console.log(person.getName()); // Lee
console.log(anotherPerson.getName()); // Kim
console.log(getName()) // ''
// 일반함수로 호출된 getName 함수 내부의 this.name은 브라우저 환경에서 window.name과 같다.
```


프로토타입 메서드 내부에서 사용된 this도 일반 메서드와 마찬가지로 해당 메서드를 호출한 객체에 바인딩된다.

### 22.2.3 생성자 함수 호출

생성자 함수 내부의 this에는 생성자 함수가 생성할 인스턴스가 바인딩된다.

```js

// 생성자 함수

function Circle(radius) {
	this.radius = radius;
	this.getDiameter = function () {
		return 2 * this.radius;
	}
}

const cicle1 = new Circle(5);
console.log(circle1.getDiameter()); // 10

const cicle2 = Circle(15) // new가 없으면 일반 함수 호출이다. 생성자 아니다.
```

생성자 함수는 이름 그대로 객체(인스턴스)를 생성하는 함수다. 일반 함수와 동일한 방법으로 생성자 함수를 정의하고 **new 연산자와 함께 호출하면** 해당 함수는 생성자 함수로 동작한다.


### 22.2.4 Function.prototype.apply / call / bind 메서드에 의한 간접 호출

apply / call / bind 메서드는 Function.prototype의 메서드다. 즉 , 이들 메서드는 모든 함수가 상속받아 사용할 수 있다.


Function.prototype.apply, Function.prototype.call 메서드는 this로 사용할 객체와 인수 리스트를 인수로 전달받아 함수를 호출한다.

```js

function getThisBinding() {
	return this;
}

const thisArg = {a : 1}
console.log(getThisBinding()); // window

console.log(getThisBinding.apply(this.Arg)) // {a:1}
console.log(getThisBinding.call(this.Arg)) // {a:1}
```


**apply와 call 메서드의 본질적인 기능은 함수를 호출하는 것이다.**  apply와 call 메서드는 함수를 호출하면서 첫 번째 인수로 전달한 특정 객체를 호출한 함수의 this에 바인딩한다.

bind 메서드는 메서드의 this와 메서드 내부의 중첩 함수 또는 콜백 함수의 this가 불일치하는 문제를 해결하기 위해 유용하게 사용된다.

```js

const person = {
	name: 'Lee',
	foo(callback) {
		setTimeout(callback, 100)
	}
};

person.foo(function () {
	console.log(`Hi! my name is ${this.name}.`)// Hi my name is .
	// 일반 함수로 호출된 콜백 함수 내부의 this.name은 window.name과 같다.
})
```

위 문제를 bind로 쉽게 해결 가능하다

```js

const person = {
	name: 'Lee',
	foo(callback) {
		setTimeout(callback.bind(this, 100)
	}
};

person.foo(function () {
	console.log(`Hi! my name is ${this.name}.`)// Hi my name is Lee.
	// 일반 함수로 호출된 콜백 함수 내부의 this.name은 window.name과 같다.
})
```

## this 바인딩 정리

| 함수 호출 방식                                             | this 바인딩                                                |
| ---------------------------------------------------- | ------------------------------------------------------- |
| 일반 함수 호출                                             | 전역 객체                                                   |
| 메서드 호출                                               | 메서드를 호출한 객체                                             |
| 생성자 함수 호출                                            | 생성ㅅ자 함수가 (미래에) 생성할 인스턴스                                 |
| Function.prototype.apply/call/bind 메서드에 의한 간접 호출<br> | Function.prototype.apply/call/bind 메서드에 첫 번째 인수로 전달한 객체 |
