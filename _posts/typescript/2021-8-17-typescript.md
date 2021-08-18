---
title: "TypeScript Tutorial"
excerpt: "TypeScript Tutorial"
categories:
    - typescript
tags:
    - typescript
last_modified_at: 2021-8-17T14:00:00+09:00
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

## Better Workflow & tsconfig
```typescript
```

## Better Workflow & tsconfig
```typescript
```