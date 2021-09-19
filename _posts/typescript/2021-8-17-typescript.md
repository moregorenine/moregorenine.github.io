---
title: "TypeScript Tutorial"
excerpt: "TypeScript Tutorial"
categories:
    - typescript
tags:
    - typescript
last_modified_at: 2021-8-22T00:00:00+09:00
toc: true
toc_sticky: true
---

## typescript 강의
아래 youtube 강의를 정리한 문서입니다.  
[TypeScript Tutorial - The Net Ninja](https://www.youtube.com/playlist?list=PL4cUxeGkcC9gUgr39Q_yD6v-bSyMwKPUI)

## Type Basics
- 변수의 타입을 선언합니다.
- 변수에 선언된 타입 이외의 데이터를 할당할 수 없습니다.

```typescript
let character = 'mario';
let age = 30;
let isBlackBelt = false;

//character = 20; //error : 위에서 'string' type으로 초기화 되었기 때문입니다.
character = 'luigi'; //pass

//age = 'yoshi'; //error : 위에서 'number' type으로 초기화 되었기 때문입니다.
age = 40; //pass

//isBlackBelt = 'yes'; //error : 위에서 'boolean' type으로 초기화 되었기 때문입니다.
isBlackBelt = true; //pass

const circ = (diameter: number) => {
    return diameter * Math.PI;
};

//console.log(area('hello')); //error : circ함수의 diameter parameter가 'number' type으로 지정 되었기 때문입니다. 
console.log(circ(7.5));
```

## Objects & Arrays
객체와 배열에 대한 내용입니다.
```typescript
//arrays
let names = ['luigi', 'mario', 'yoshi'];

names.push('toad'); //pass
//names.push(3); //error : names 배열은 'string' type으로 초기화 되었기 때문입니다.
//names[1] = 3; //error : names 배열은 'string' type으로 초기화 되었기 때문입니다.

let numbers = [10, 20, 12, 15];

numbers.push(25); //pass
//numbers.push('shaun'); //error : names 배열은 'number' type으로 초기화 되었기 때문입니다.
//numbers[0] = 'shaun'; //error : names 배열은 'number' type으로 초기화 되었기 때문입니다.

let mixed = ['ken', 4, 'chun-li', 8, 9];

mixed.push('ryu'); //pass
mixed.push(10); //pass
mixed[0] = 3; //pass
mixed.push(true); //error : mixed 배열은 'string | number' type으로 초기화 되었기 때문에 'boolean' type의 parameter는 허용되지 않습니다.

//objects
let ninja = {
  name: 'mario',
  belt: 'black',
  age: 30
};

ninja.age = 40; //pass
ninja.name = 'ryu'; //pass
//ninja.age = '30'; //error : names 배열은 'number' type으로 초기화 되었기 때문입니다.
//ninja.skills = ['fighting', 'sneaking'] //error : 'skills'이란 property는 ninja object에 존재하지 않습니다.
                                           //error : object를 한번 선언/정의 하면 properties들을 추가 할 수 없습니다. 

ninja = {
  name: 'yoshi',
  belt: 'orange',
  age: 40, //주석처리할 경우 error 발생합니다. object가 한번 선언/정의 되면 재정의 할 경우 properties들이 정확하게 일치 해야합니다. 
  //skills: ['running'], //error : 존재하지 않는 property입니다.
};
```

## Explicit Types
위에서 살펴본 바와 같이 typescript는 변수의 선언과 동시에 값을 할당할 경우,  
typescript가 변수에 선언된 값의 type을 자동으로 변수에 지정합니다.  
만약 변수를 선언하고 값을 초기화하지 않을 경우 우리는 어떻게 변수의 type을 선언해야 할까요?
```typescript
let character: string = 'mario';
let age: number;
let isLoggedIn: boolean;

//age = 'luigi'; //error
age = 30;

//isLoggedIn = 25; //error
isLoggedIn = true;

//arrays
let ninjas: string[] = []; //[]으로 초기화 하지 않을 경우 값 할당시 error 발생합니다.

ninjas.push('ryu');
ninjas.push('chun-li');
console.log(ninjas);

//union types
let mixed: (string|number|boolean)[] = [];
mixed.push('hello');
mixed.push(false);
mixed.push(20);
console.log(mixed);

let uid: string|number;
uid = '123';
uid = 123;

//objects
let ninjaOne: object;
ninjaOne = { name: 'yoshi', age: 30 };

let ninjaTwo: {
  name: string,
  age: number,
  beltColour: string
};
ninjaTwo = { name: 'ken', age: 20, beltColour: 'black' };
```

## Dynamic (any) Types
`any` type은 모든 type의 값을 허용합니다. 즉 typescript를 사용하는 이점이 전혀 없으며 사용시 주의가 필요합니다.

```typescript
let age: any = 25;

age = true;
console.log(age);
age = 'hello';
console.log(age);
age = { name: 'luigi' };
console.log(age);

let mixed: any[] = [];

mixed.push(5);
mixed.push('mario');
mixed.push(false);
console.log(mixed);

let ninja: { name: any, age: any };

ninja = { name: 'yoshi', age: 25 };
console.log(ninja);

ninja = { name: 25, age: 'yoshi' };
console.log(ninja);
```

## Better Workflow & tsconfig
### 프로젝트 폴더 구성
`tsconfig.json` typescript 설정 파일 생성
```sh
tsc --init
```
### tsconfig.json 설정에 따른 실행
```sh
tsc //tsconfig.json 설정에 따른 실행
tsc -w //tsconfig.json 설정에 따른 watch 실행 및 감시, 소스파일 변경 후 저장시 tsc를 자동 실행합니다.
```

## Function Basics
```typescript
// let greet: Function = () => {
//   console.log('hello, world');
// }

// greet = 'hello';

// greet = () => {
//   console.log('hello, again');
// }

const add = (a: number, b: number, c/*?*/: number | string = 10): void => {
  console.log(a + b);
  console.log(c);
}

add(5, 10, 'ninja');

const minus = (a: number, b: number): number => {
  return a + b;
}

let result = minus(10,7);
console.log(result);
```

## Type Aliases
`type` 별칭을 사용하여 코드 중복을 줄일 수 있습니다.
```typescript
type StringOrNum = string | number;
type objWithName = { name: string, uid: StringOrNum };

const logDetails = (uid: StringOrNum, item: string) => {
  console.log(`${item} has a uid of ${uid}`);
}

const greet = (user: objWithName) => {
  console.log(`${user.name} says hello`);
}

const greetAgain = (user: objWithName) => {
  console.log(`${user.name} says hello Again`);
}
```

## Function Signatures
```typescript
// let greet: Function;

// example 1
let greet: (a: string, b: string) => void;

greet = (name: string, greeting: string) => {
  console.log(`${name} says ${greeting}`);
}

// example 2
let calc: (a: number, b: number, c: string) => number;

calc = (numOne: number, numTwo: number, action: string) => {
  if (action === 'add') {
    return numOne + numTwo;
  } else {
    return numOne - numTwo;
  }
}

// example 3
let logDetails: (obj: {name: string, age: number}) => void;

logDetails = (ninja: {name: string, age: number}) => {
  console.log(`${ninja.name} is ${ninja.age} years old`);
}
```

## The DOM & Type Casting
`.html file` file
```html
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TypeScript Tutorial</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>

  <div class="wrapper">
    <h1>Finance Logger</h1>

    <!-- output list -->
    <ul class="item-list">
      
    </ul>
  </div>

  <footer>
    <form class="new-item-form">
      <div class="field">
        <label>Type:</label>
        <select id="type">
          <option value="invoice">Invoice</option>
          <option value="payment">Payment</option>
        </select>
      </div>
      <div class="field">
        <label>To / From:</label>
        <input type="text" id="tofrom">
      </div>   
      <div class="field">
        <label>Details:</label>
        <input type="text" id="details">
      </div>
      <div class="field">
        <label>Amount (£):</label>
        <input type="number" id="amount">
      </div>
      <button>Add</button>
    </form>

    <a href="https://www.thenetninja.co.uk">The Net Ninja</a>
  </footer>

  <script src='app.js'></script>
</body>
</html>
```

`.ts typescript` file
```typescript
const anchor = document.querySelector('a')!;
if(anchor) {
  console.log(anchor.href);
}
console.log(anchor.href);

//const form = document.querySelector('form')!;
const form = document.querySelector('.new-item-form') as HTMLFormElement;
console.log(form.children);

// inputs
const type = document.querySelector('#type') as HTMLInputElement;
const tofrom = document.querySelector('#tofrom') as HTMLInputElement;
const details = document.querySelector('#details') as HTMLInputElement;
const amount = document.querySelector('#amount') as HTMLInputElement;

form.addEventListener('submit', (e: Event) => {
  e.preventDefault();

  console.log(
    type.value, 
    tofrom.value, 
    details.value, 
    amount.valueAsNumber
  );
});
```
selector 객체의 value 값을 가져올 때 valueAsNumber, valueAsDate 로 number나 date type으로 가져올 수 있습니다.

## Classes
`.ts typescript` file
```typescript
// classes
class Invoice {
  client: string;
  details: string;
  amount: number;

  constructor(c: string, d: string, a: number){
    this.client = c;
    this.details = d;
    this.amount = a;
  }

  format() {
    return `${this.client} owes £${this.amount} for ${this.details}`;
  }
}

const invOne = new Invoice('mario', 'work on the mario website', 250);
const invTwo = new Invoice('luigi', 'work on the luigi website', 300);

let invoices: Invoice[] = [];
invoices.push(invOne)
invoices.push(invTwo);
// invoices.push({ name: 'shaun' });

invoices.forEach(inv => {
  console.log(inv.client, /*inv.details,*/ inv.amount, inv.format());
})
```
invOne.client = 'shaun'  
invTwo.amount = 500  
같은 객체의 필드/속성 값에 직접 접근하여 수정이 가능합니다. 이를 방지하기 위한 방법을 아래의 Public, Private & Readonly 에서 소개해드립니다.

## Public, Private & Readonly
java처럼 접근 제어자가 있습니다.  
- readonly
  - class 내부/외부에서 접근이 가능합니다.
  - class 내부/외부에서 값 변경이 불가능합니다.
  - 생성자에서 값 할당이 가능합니다.
- private
  - class 내부에서만 접근 및 값 변경이 가능합니다.
- public
  - class 내부/외부에서 접근 및 값 변경이 가능합니다.

`.ts typescript` file
```typescript
// classes
class Invoice {
  // readonly client: string;
  // private details: string;
  // public amount: number;

  constructor(
    readonly client: string,
    private details: string,
    public amount: number,
  ) { }

  format() {
    return `${this.client} owes £${this.amount} for ${this.details}`;
  }

  setClient(str: string) {
    this.client = str //error : read-only 필드는 class 내부/외부에서 값 변경이 불가능합니다.
    console.log(invOne.client) //pass : read-only 필드는 class 내부/외부에서 값 접근이 가능합니다.
  }
  setDetails(str: string) {
    this.details = str //pass : pivate 필드는 class 내부에서만 접근이 가능합니다.
  }
  setAmout(num: number) {
    this.amount = num //pass : public 필드는 class 내부/외부에서 접근이 가능합니다.
  }
}

const invOne = new Invoice('mario', 'work on the mario website', 250);

// invOne.client = "shaun" //error : class 내부/외부에서 값 변경이 불가능합니다.
console.log(invOne.client) //pass : class 내부/외부에서 접근이 가능합니다.

// invOne.details = "work on the mario website" //error : pivate 필드는 class 내부에서만 접근이 가능합니다.
invOne.setDetails("work on the mario website") //pass

invOne.amount = 500 //pass : public 필드는 class 내부/외부에서 접근이 가능합니다.
```

## Modules
아래 모듈 설정은 두가지 단점이 존재합니다.
- es6를 지원하는 최신브라우저에서만 올바르게 동작합니다.
- 코드를 단일파일로 묶지 않습니다. 브라우저는 모듈 시스템을 사용하여 별도의 파일들을 로드하고 컴파일시 여러 요청을 합니다.

차후 webpack을 통해 이를 보완 가능합니다.

`tsconfig.json`
```typescript
{
  "compilerOptions": {
    /* Basic Options */
    "target": "es6", /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */
    "module": "es2015", /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */
  },
}
```
- "target": "es6"
  - 출력을 es6 으로합니다. se6가 지원되는 최신 브라우저가 대상입니다.
- "module": "es2015"
  - 일반 javascript 버젼의 module 사용합니다.

`index.html` type을 module로 지정합니다.
```html
<script type="module" src='app.js'></script>
```

`Invoice.ts` export 합니다.
```typescript
export class Invoice {
...
}
```

`app.ts` exporte된 모듈을 import 합니다.
```typescript
import { Invoice } from './classes/Invoice.js';
```

## interface
java의 interface와 비슷합니다. interface에서는 형태만 선언하고, interface를 type으로 하는 객체에서 interface에 선언된 필드 및 메소드를 구현해야 합니다.  
`app.ts`
```typescript
import { Invoice } from './classes/Invoice.js';

// interfaces
export interface IsPerson {
  name: string;
  age?: number;
  speak(a: string): void;
  spend(a: number): number;
}

const me: IsPerson = {
  name: 'shaun',
  //age: 30,
  speak(text: string): void {
    console.log(text);
  },
  spend(amount: number): number {
    console.log('I spent ', amount);
    return amount;
  },
};

console.log(me);
me.speak('hello, world');

const greetPerson = (person: IsPerson): void => {
  console.log('hello ', person.name);
}

greetPerson(me);
// greetPerson({name: 'shaun'});
```

## Interfaces with Classes
interface에서 format 메소드를 선언합니다.  
`HasFormatter.ts`
```typescript
export interface HasFormatter {
  format(): string;
}
```

`Invoice` class에서 `HasFormatter` interface를 implement합니다.  
`format` 메소드가 수행할 기능을 구현합니다.  
`Invoice.ts`
```typescript
import { HasFormatter } from '../interfaces/HasFormatter.js';

export class Invoice implements HasFormatter {
  constructor(
    readonly client: string, 
    private details: string, 
    public amount: number,
  ){}

  format() {
    return `${this.client} owes £${this.amount} for ${this.details}`;
  }
}
```
`Invoice`와 `Payment` class는 둘다 `HasFormatter` interface를 implement 했기 때문에  
`Invoice`와 `Payment` 를 HasFormatter 형으로 upcasting 할 수 있습니다.   
`app.ts`
```typescript
import { Invoice } from './classes/Invoice.js';
import { Payment } from './classes/Payment.js';
import { HasFormatter } from './interfaces/HasFormatter.js';

let docOne: HasFormatter;
let docTwo: HasFormatter;

docOne = new Invoice('yoshi', 'web work', 250);
docTwo = new Payment('mario', 'plumbing', 200);

let docs: HasFormatter[] = [];
docs.push(docOne);
docs.push(docTwo);

docs.forEach(doc => {
  console.log(doc.format())
})
```

## Generics
Generic 또한 class, interface 처럼 javascript에서는 존재하지 않는 개념 이였습니다.  
특히 타입과 관련된 Generic은 현재도 javascript에서는 없는 typescript 개념 입니다.  
타 언어에서 먼재 존재하던 기술인데요. 코딩시 type을 특정 짓지 않고 유연한 코딩을 가능하게 해주는 기술입니다.  

```typescript
const addUID = (obj: object) => {
   let uid = Math.floor(Math.random() * 100);
   return {...obj, uid};
}

let docOne = addUID({name: 'yoshi', age: 40});

console.log(docOne.name);
```
addUID 메소드는 `object` type을 인자로 받습니다. javascript에서는 object가 모든 객체의 최상의 객체이기 때문에 모든 객체들은 object로 upcasting 될 수 있습니다.  
인자로 넘겨받은 객체에 uid란 난수를 생성해 {넘겨 받은 객체와 난수}를 객체화 해서 반환해 줍니다.

그럼 변수 `docOne`는 {name: 'yoshi', age: 40, uid: 3(난수)} 란 객체를 얻을 것 입니다.

그런데 `docOne.name`을 출력하면 typescript에서 오류가 발생할 것 입니다.  
왜나하면 addUID 메소드가 반환하는 객체에 `name`이란 필드의 존재 여부를 typescript에서 알려주지 않았기 때문입니다.

```typescript
const addUID = <T>(obj: T) => {
  let uid = Math.floor(Math.random() * 100);
  return {...obj, uid};
}

let docOne = addUID({name: 'yoshi', age: 40});
let docTwo = addUID('shaun');

console.log(docOne.name);

console.log(docTwo.name);
```
`<T>`은 함수에 전달된 `T란 Type`을 capture한다는 의미입니다.(T이외의 다른 알파벳이 사용되어도 무관합니다.)  
그리고 capture된 `T type` 형태 인자 `obj`를 받겠다는 의미 입니다.  
그래서 `addUID` 메서드가 객체 반환시 `obj`에 어떤 속성이 있는지 알 수 있게 됩니다.  
docOne은 정상 작동을 합니다만,  
docTwo의 경우 {0: 's', 1: 'h', 2: 'a', 3: 'u', 4: 'n', uid: 48} 란 객체를 반환받게 됩니다.  
그래서 typescript에서는 Property 'name' does not exist on type '"shaun" & { uid: number; }' 오류를 반환할 것 입니다.  
그래서 코딩시 오류를 방지하기 위해 typescript에서 type을 capture할 시에 extends로 string 타입의 name필드의 객체를 선언해 오류를 미연해 방지하는 코딩을 하게 됩니다.

`app.ts`
```typescript
const addUID = <T extends {name: string}>(obj: T) => {
  let uid = Math.floor(Math.random() * 100);
  return {...obj, uid};
}

let docOne = addUID({name: 'yoshi', age: 40});
//let docTwo = addUID('shaun');

console.log(docOne.name);

// with interfaces
interface Resource<T> {
  uid: number;
  resourceName: string;
  data: T;
}

const docThree: Resource<object> = {
  uid: 1, 
  resourceName: 'person', 
  data: { name: 'shaun' }
};

const docFour: Resource<string[]> = {
  uid: 1, 
  resourceName: 'shoppingList', 
  data: ['bread', 'milk']
};

console.log(docThree, docFour);
```

## Enums
Enum 또한 javascript에는 없던 타언어에서 사용되는 skill입니다.  
상수 또는 키워드 세트를 저장하고 숫자 값과 연관시킬 수 있는 특별한 유형입니다.

`enum 적용 전 코드`
```typescript
interface Resource<T> {
  uid: number;
  resourceType: number;
  data: T;
}

const docOne: Resource<object> = {
  uid: 1,
  resourceType: 0,
  data: { title: 'name of the wind' }
}
const docTwo: Resource<object> = {
  uid: 10,
  resourceType: 3,
  data: { title: 'name of the wind' }
}
```
resourceType의 0이 BOOK을 뜻하고 3이 DIRECTOR를 뜻한다고 생각해봅시다. 4는 PERSON이 될 수도 있습니다.  
하지만 이런 number로 관리하는 방식이 정확히 무슨 의미인지 코드상으로는 확인할 수가 없습니다.  
이럴 경우 무의미한 number로 관리되는 부분을 의미가 있는 문자열로 표현해서 코드상를 효율적으로 관리할 수 있습니다.

`enum 적용 후 코드`
```typescript
enum ResourceType { BOOK, AUTHOR, FILM, DIRECTOR, PERSON };

interface Resource<T> {
  uid: number;
  resourceType: ResourceType;
  data: T;
}

const docOne: Resource<object> = {
  uid: 1,
  resourceType: ResourceType.BOOK,
  data: { title: 'name of the wind' }
}
const docTwo: Resource<object> = {
  uid: 10,
  resourceType: ResourceType.DIRECTOR,
  data: { title: 'name of the wind' }
}

console.log(docOne);
console.log(docTwo);
```
ResourceType.BOOK이 0으로 ResourceType.DIRECTOR가 3으로 `enum ResourceType`의 index로 변경되어 출력되는 것을 확인 할 수 있습니다.

## Tuples
`기본 사용법`
```ts
// TUPLES
let arr = ['ryu', 25, true];
arr[0] = false;
arr[1] = 'yoshi';
arr = [30, false, 'yoshi'];

let tup: [string, number, boolean] = ['ryu', 25, true];
// tup[0] = false;
tup[0] = 'ken';

let student: [string, number];
//student = [23564, 'chun-li'];
student = ['chun-li', 23564];
```

`tuples 적용 전 코드`
```ts
let doc: HasFormatter;
if (type.value === 'invoice') {
  doc = new Invoice(tofrom.value, details.value, amount.valueAsNumber);
} else {
  doc = new Payment(tofrom.value, details.value, amount.valueAsNumber);
}
```

`tuples 적용 후 코드`
```ts
let values: [string, string, number];
values = [tofrom.value, details.value, amount.valueAsNumber];

let doc: HasFormatter;
if (type.value === 'invoice') {
  doc = new Invoice(...values);
} else {
  doc = new Payment(...values);
}
```