# [아이템 14] 타입 연산과 제너릭 사용으로 반복 줄이기
### 요약
>타입이 중복되지 않도록 type.ts 파일로 관리하기
>
>중복되는 타입은 변수로 지정하기
>
>유틸리티 타입으로 기존 타입, 자바스크립트 값 활용하기
>
>중요! interface에서는 Mapped Type 선언이 불가능하다.

## 1. 중복을 새로운 선언으로 해결하기
### 중복되는 타입 선언은 `type 타입명 = 내용`으로 줄이자.

#### X
```ts
interface Props {
  onMouseOver?: () => void;
  onMouseOut?: () => void;
  onTouchStart?: () => void;
  onFocus?: () => void;
  onBlur?: () => void;
}
```

#### O
```ts
type Handler = () => void;

interface ProfileProps {
  onMouseOver?: Handler;
  onMouseOut?: Handler;
  onTouchStart?: Handler;
  onFocus?: Handler;
  onBlur?: Handler;
}
```
    
## 2. 중복을 기존의 타입을 이용해서 해결하기
### interface의 중복은 `extends`를 이용하자.
#### X
```ts
interface Props {
  onMouseOver?: () => void;
  onMouseOut?: () => void;
}

interface newProps {
  onMouseOver?: () => void;
  onMouseOut?: () => void;
  onTouchStart?: () => void;
  onFocus?: () => void;
  onBlur?: () => void;
}
```

#### O
```ts
interface Props {
  onMouseOver?: () => void;
  onMouseOut?: () => void;
}

interface newProps extends Props {
  onTouchStart?: () => void;
  onFocus?: () => void;
  onBlur?: () => void;
}
```

###  객체 타입은 인덱싱을 이용하자.
 `{새로운 속성 타입 : 기존interface["재사용할 속성"]}`의 형태로 사용한다.

#### O
```ts
interface Props {
  onMouseOver?: () => void;
  onMouseOut?: () => void;
}

interface newProps {
  onTouchStart?: Props["onMouseOver"];
}
```
   
###  타입 매핑을 이용하자.
**무조건 type으로!! interface는 안된다!!**
`{[k in {반복문을 실행할 유니온 타입}]: 기존interface[k]}`의 형태로 사용한다.

#### X
```ts
type Handler = () => void;

interface ProfileProps {
  onMouseOver?: Handler;
  onMouseOut?: Handler;
  onTouchStart?: Handler;
  onFocus?: Handler;
  onBlur?: Handler;
}
```

#### O
```ts
type Handler = () => void;

type HandlerName = "onMouseOver" | "onMouseOut" | "onTouchStart" | "onFocus" | "onBlur"

type Props = {
  [k in HandlerName ]: Handler;
}
```

### 유틸리티 타입을 이용하자.
**무조건 type으로!! interface는 안된다!!**
`pick`, `partial`

#### pick
```ts
interface ProfileProps {
  onMouseOver?: Handler;
  onMouseOut?: Handler;
  onTouchStart?: Handler;
  onFocus?: Handler;
  onBlur?: Handler;
}

type NewProps = Pick<ProfileProps, "onBlur">

// === {
  onBlur?: Handler;
}

```

#### partial
```ts
interface ProfileProps {
  onMouseOver: Handler;
  onMouseOut: Handler;
  onTouchStart: Handler;
  onFocus: Handler;
  onBlur: Handler;
}

type NewProps = Partial<ProfileProps>

// === {
  onMouseOver?: Handler;
  onMouseOut?: Handler;
  onTouchStart?: Handler;
  onFocus?: Handler;
  onBlur?: Handler;
}

```

### keyof를 이용하자.
`keyof 객체 타입`의 형태로 객체 타입의 속성만 유니온 타입으로 가져올 수 있다.

```ts
interface ProfileProps {
  index: number;
  id: number;
  name: string;
  friends: friend[];
}

type KeyProps = keyof ProfileProps

// === "index" | "id | "name" | "friends"
```

`객체 타입[keyof 객체타입]`의 형태로 객체 타입의 값을 유니온 타입으로 가져올 수 있다.

```ts
interface ProfileProps {
  index: number;
  id: number;
  name: string;
  friends: friend[];
}

type KeyProps = ProfileProps[keyof ProfileProps]

// === number | string | friend[]
```

### 3. 중복을 기존의 값을 이용해서 해결하기
#### `typeof 값`으로 기존 값 형태를 타입으로 바꾸어 이용하자.
```ts
const object = {
  보고싶은사람: "손상희",
  하고싶은말: "사랑해",
}

type Object = typeof object

// ===   {
  보고싶은사람: string,
  하고싶은말: string,
}

const object = {
  보고싶은사람: "손상희",
  하고싶은말: "사랑해",
} as const

type Object = typeof object

// === {
    보고싶은사람: "손상희";
    하고싶은말: "사랑해";
}
```

#### `ReturnType<함수>`로 함수와 메서드의 리턴값을 이용하자.
```ts
type Func = (str: "있을때") => "잘하자"

type ReturnFunc = ReturnType<Func>

// === "잘하자"
```

```ts
const func = (str: "있을때"): "잘하자" => {
  return "잘하자"
}

type ReturnFunc = ReturnType<typeof func>

// === "잘하자"
```

### 4. 제너릭을 사용해서 중복을 제거하기
#### 매핑타입과 타입단언을 이용해서 `{[Property in keyof 기존타입 as 'newName${타입변환제너릭}']}`의 형태로 속성이름을 바꿀 수 있다.
#### 속성이름을 바꿀 때는 `Capitalize<>`, `Uncapitalize<>`, `Uppercase<>`, `Lowercase<>` 를 사용해서 대소문자를 변경할 수 있다.

```ts
type APIData = {
  name: string;
  id: number;
};

type newAPIData = {
  [Property in keyof APIData as `user${Capitalize<Property>}`]: APIData[Property];
};

// === {
    userName: string;
    userId: number;
}
```

#### 이를 이용해서 제네릭을 이용한 타입 변환기, Formatter를 만들 수 있다. 
#### `Capitalize<Key & string>` 에서 Capitalize의 제네릭 인자가 string 의 부분집합이어야 하므로 `& string`을 추가한다.

```ts
type APIData = {
  name: string;
  id: number;
};

type Formatter<T> = {
  [Key in keyof T as `format${Capitalize<Key & string>}`]: T[Key];
}

type newAPIData = Formatter<APIData>

// === {
    formatName: string;
    formatId: number;
}
```
