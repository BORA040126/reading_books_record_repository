## 🌈 Chapter 3 : 네이티브

- 가장 많이 사용하는 네이티브들
> `String`, `Number`, `Boolean`, `Array`, `Object`, `Function`, `RegExp`, `Date`, `Error`, `Symbol`
- 네이티브는 내장 함수이다.

```javascript
var a = new String('abc');
typeof a; // object
a instanceof String; // true
Object.prototype.toString.call(a); // "[object String]"
```

- `new String('abc')` 생성자의 결과는 원시 값 `abc`를 감싼 **객체 레퍼**이다.
- `typeof` 연산자로 이 객체의 타입을 확인해보면 자신이 감싼 원시 값의 타입이 아닌 `object`의 하위 타입에 가깝다.

```javascript
console.log(a); // String {"abc"}
```

- `new String('abc')`은 `abc`를 감싸는 문자열 래퍼를 생성하며 원시 값 `abc`는 아니다.

### 🎯 내부 `[[Class]]`

- `typeof`가 `object`인 값에는 `[[Class]]`라는 내부 프로퍼티가 추가로 붙는다.
- 이 프로퍼티는 **직접 접근할 수 없고** `Object.prototype.toString()`라는 메서드에 값을 넣어 호출함으로써 존재를 엿볼 수 있다.

```javascript
Object.prototype.toString.call([1, 2, 3]); // "[object Array]"

Object.prototype.toString.call(/regex-literal/i); // "[object RegExp]"
```

- 내부 `[[Class]]` 값이, 배열은 `Array` 정규식은 `RegExp`이다.
- 대부분 내부 `[[Class]]`는 해당 값과 관련된 내장 네이티브 생성자를 가리키지만, 그렇지 않은 경우도 존재한다.

```javascript
Object.prototype.toString.call(null); // "[object Null]"
Object.prototype.toString.call(undefined); // "[object Undefined]"
```

- 위와 같이 `Null()`, `Undefined()` 같은 네이티브 생성자는 없지만 내부 `[[Class]]` 값은 `Null`, `Undefined` 이다.
- 하지만 그 밖의 문자열, 숫자, 불리언 같은 단순 원시 값은 **박싱(Boxing)** 과정을 거친다.

```javascript
Object.prototype.toString.call('abc'); // "[object String]"
Object.prototype.toString.call(42);  // "[object Number]"
Object.prototype.toString.call(true); // "[object Boolean]"
```

- 내부 `[[Class]]` 값이 각각 `String`, `Number`, `Boolean`으로 표시된 것으로 보아 **단순 원시 값은 해당 객체 레퍼로 자동 박싱**됨을 알 수 있다.


### 🎯 래퍼 박싱하기
- 원시 값엔 프로퍼티나 메서드가 없으므로 `.length`, `.toString()`으로 접근하려면 원시 값을 객체 래퍼로 감싸줘야 한다.
- 자바스크립트는 **원시 값을 알아서 박싱(래핑)** 하므로 다음과 같은 코드가 가능하다.

```javascript
var a = 'abc';

a.length; // 3
a.toUpperCase(); // 'ABC'
```

- 따라서 루프 조건 `i < a.length` 처럼 빈번하게 문자열 값의 프로퍼티/메서드를 사용해야 한다면 자바스크립트 엔진이 암시적으로 객체를 생성할 필요가 없도록 처음부터 값을 객체로 갖고 있는 것이 이치에 맞는 것처럼 보이지만, 좋은 생각이 아니다.
- 개발자가 직접 객체 형태로(최적화되지 않은 방향으로) 선 최적화(pre-Optimize)하면 프로그램이 더 느려질 수 있다.
- 직접 객체 형태로 써야 할 이유는 거의 없고, 필요시 엔진이 알아서 암시적으로 박싱하게 하는 것이 낫다.

#### 📚 객체 래퍼의 함정
- 다음 예제는 `Boolean`으로 래핑한 값이 있다.

```javascript
var a = new Boolean(false);

if(!a) {
  console.log('Oops'); // 실행 X
}
```
- `false`를 객체 래퍼로 감쌌지만 문제는 **객체가 `truthy`란 점이다.**
- 그래서 예상과 달리, 안에 들어있는 `false`값과 반대의 결과이다.
- **수동으로 원시 값을 박싱하려면 `Object()` 함수를 이용한다.**

```javascript
var a = 'abc';
var b = new String(a);
var c = Object(a);

typeof a; // "string"
typeof b; // "object"
typeof c; // "object"

b instanceof String; // true
c instanceof String; // true

Object.prototype.toString.call(b); // "[object String]"
Object.prototype.toString.call(c); // "[object String]"
```

- 하지만 **객체 레퍼로 직접 박싱하는 건 권하지 않는다.**

### 🎯 언박싱

- 객체 래퍼의 원시 값은 `valueOf()` 메서드로 추출한다.

```javascript
var a = new String('abc');
var b = new Number(42);
var c = new Boolean(true);

a.valueOf(); // 'abc'
b.valueOf(); // 42
c.valueOf(); // true
```

- 이때도 암시적인 언박싱이 일어난다. (자세한 사항 강제변환은 4장에서)

```javascript
var a = new String('abc');
var b = a + ''; // 'b'에는 언박싱된 원시 값 'abc'이 대입된다.

typeof a; // "object"
typeof b; // "string"
```

### 🎯 네이티브, 나는 생성자다.
- 배열, 객체, 함수, 정규식 값은 리터럴 형태로 생성하는 것이 일반적이지만, **리터럴은 생성자 형식으로 만든 것과 동일한 종류의 객체를 생성한다.**
- 필요해서 쓰는 게 아니라면 **생성자는 가급적 쓰지 않는 편이 좋다.**

#### 📚 Array()

```javascript
var a = new Array(1, 2, 3);
a; // [1, 2, 3]

var b = [1, 2, 3];
b; // [1, 2, 3]
```

> `Array()` 생성자 앞에 `new`를 붙이지 않아도 된다. 붙이지 않아도 붙인 것처럼 작동한다. 즉, `Array(1, 2, 3)`와 `new Array(1, 2, 3)` 와 같다.

- `Array` 생성자는 인자로 숫자를 하나만 받으면 그 숫자를 원소로하는 배열을 생성하는 게 아니라 배열의 크기를 미리 정하는 기능이다.
- 하지만 **배열의 크기를 미리 정한다는 것 자체가 밀이 안 된다.**
- 그렇게 하려면 빈 배열을 만들고 나중에 `length` 프로퍼티에 숫자 값을 할당하는 게 맞다.

> 빈 슬롯을 한 군데 이상 가진 배얼을 구멍 난 배열(Sparse Array)라고 한다.

```javascript
var a = new Array(3);
var b = [undefined, undefined, undefined];
var c = [];
c.length = 3;

a; // [empty × 3]
b; // [undefined, undefined, undefined]
c; // [empty × 3]
```

- 브라우저 개발자 콘솔 창 마다 객체를 나타내는 방식이 제각각이라 혼란이 가중된다.
- `a`와 `b`가 어떨 때는 같은 값처럼 보이다가도 그렇지 않을 때도 있다.

```javascript
a.join('-'); // "--"
b.join('-'); // "--"

a.map(function(v, i){return i;}); // [empty × 3]
b.map(function(v, i){return i;}); // [0, 1, 2]
```

- `a.map()`은 `a`에 슬롯이 없기 때문에 `map()` 함수가 순회할 원소가 없다.
- `join()`은 구현 로직이 다음과 같기 때문에 다르다.

```javascript
function fakeJoin(arr, connector) {
  var str = '';
  for(var i = 0; i < arr.length; i++) {
    if(i > 0) {
      str += connector;
    }
    if(arr[i] !== undefined) {
      str += arr[i];
    }
  }
  return str;
}

var a = new Array(3);
fakeJoin(a, '-'); // "--"
```

- `join()`은 슬롯이 있다는 가정하에 `length`만큼 루프를 반복한다.
- `map()` 함수는 빈 슬롯 배열이 입력되면 예기치 않은 결과가 빚어지거나 실패의 원인이 된다.
- 진짜 `undefined` 값 원소로 채워진 배열은 어떻게 생성할까?

```javascript
var a = Array.apply(null, { length: 3 });
a; // [undefined, undefined, undefined]
```

- `apply()`는 모든 함수에서 사용 가능한 유틸리티이다.
- 첫 번째 인자 `this`는 객체 바인딩으로, `null`로 세팅했다.
- 두 번째 인자는 인자의 배열(또는 배열 비슷한 유사 배열)로, 이 안에 **포함된 원소들이 펼쳐져(Spread) 함수의 인자로 전달된다.**
- 따라서 `Array.apply()`는 `Array()` 함수를 호출하는 동시에 `{ length: 3 }` 객체 값을 펼처 인자로 넣는다.
- `apply()` 내부에서는 아마 `0`에서 `length` 직전까지 루프를 순회할 것이다.
- 인덱스별로 객체에서 키를 가져온다. (`arr[0], arr[1], arr[2]`)
- 물론 이 세 프로퍼티도 모두 `{ length: 3 }` 객체에는 존재하지 않기 때문에 모두 `undefined`를 반환한다.
- 즉, `Array()`를 호출하면 `Array(undefined, undefined, undefined)`처럼 되어, 빈 슬롯이 아닌, `undefined`로 채워진 배열이 탄생한다.
- **절대로 빈 슬롯 배열을 애써 만들어서 사용하지 말자.**