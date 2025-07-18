## 3.1 타입스크립트만의 독자적 타입 시스템

- TS는 JS의 자료형에서 제시되지 않은 독자적 타입 시스템을 제공한다. 하지만 TS의 타입 시스템이 내포하고 있는 개념은 모두 JS에서 기인한 것 이다.
1. **any**
    - JS에서 존재하는 모든 값을 오류 없이 받을 수 있다. 결국 타입을 명시하지 않은 것과 동일한 효과를 보이는데, 존재하는 이유가 뭘까?
    - 대표적으로 3가지 예시를 들 수 있다.
        - 개발 단계에서 임시로 값을 지정해야 할 때
            
            ex. 추후 변경될 가능성이 높은 값이나 세부 항목에 대한 타입 지정이 안되었을 때
            
        - 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때
            
            ex. 외부 라이브러리 사용
            
        - 값을 예측할 수 없을 때 암묵적으로 사용
            
            ex. Fetch API
            
            ```jsx
            async function load() {
              const response = await fetch("https://api.com");
              const data = await response.json(); // response.json()의 리턴 타입은 Promise<any>로 정의되어 있다
              
              return data;
            }
            ```
            
            - 왜 any 타입인가?
                
                서버에서 어떤 JSON이 올 지 모르기 때문이다.
                
                ```jsx
                const data1 = await response.json(); // { name: string, age: number }일 수도
                const data2 = await response.json(); // string[]일 수도  
                const data3 = await response.json(); // { products: Product[] }일 수도
                ```
                
2. **unknown 타입**
    - any 타입과 유사하게 모든 타입의 값이 할당될 수 있다.
    - 그러나 any를 제외한 다른 타입으로 선언된 변수에는 unknown 타입의 값을 할당할 수 없다.
    
    ```jsx
    let unknownValue: unknown;
    
    unknownValue = 100; // any 타입과 유사하게 숫자이든
    unknownValue = "hello world"; // 문자열이든
    unknownValue = () => console.log("this is any type"); // 함수이든 상관없이 할당이 가능하지만
    
    let someValue1: any = unknownValue; // (O) any 타입으로 선언된 변수를 제외한 다른 변수는 모두 할당이 불가
    let someValue2: number = unknownValue; // (X)
    let someValue3: string = unknownValue; // (X)
    ```
    
    - any와 비슷해보이는데 왜 추가되었을까?
        
        ```jsx
        function handleFormSubmit(formData: unknown) {
          // any 사용 시 - 위험!
          // return formData.email.toLowerCase(); // 런타임 에러 가능
          
          // 만약 formData가 null이나 undefined 값이 들어온다면?
          // 만약 formData에 email이 문자열이 아닌 값이 들어온다면?
          
          // unknown 사용 시 - 안전!
          if (
            typeof formData === 'object' && 
            formData !== null &&
            'email' in formData &&
            typeof formData.email === 'string'
          ) {
            return formData.email.toLowerCase(); // ✅ 안전
          }
          
          throw new Error('Invalid form data');
        }
        ```
        
        “any는 무엇이든 괜찮다. unknown은 뭔지 모르지만 하나씩 테스트하면서 뭔지 알아내보자”
        
3. **void 타입**
    - 함수가 어떤 값도 반환하지 않을 때 사용한다.
    - JS에서는 함수에서 명시적인 반환문을 작성하지 않으면 undefined가 반환된다. 하지만 TS에서는 void 타입이 사용되는데 이것은 undefined가 아니다. 그냥 void를 지정하여 사용한다.
        - **하지만 런타임에서는 여전히 undefined가 반환된다.** 따라서 void는 “반환값을 사용하지 않겠다.”라는 의도를 나타낸다.
        
        ```jsx
        function sayHello(): void {
          console.log("Hello!");
          // 명시적인 return 없음
        }
        
        const result = sayHello();
        console.log(result); // undefined 출력됨!
        console.log(typeof result); // "undefined"
        ```
        
4. **never 타입**
    
    값을 반환할 수 없는 타입을 말한다. 여기서 값을 반환할 수 있는 것과 없는 것을 명확히 구분해야 한다.
    
    - 값을 반환할 수 없는 것
        - 에러를 던지는 경우
        
        ```jsx
        function generateError(res: Response): never {
          throw new Error(res.getMessage());
        }
        ```
        
        - 무한히 함수가 실행되는 경우
        
        ```jsx
        function checkStatus(): never {
          while (true) {
            // ...
          }
        }
        ```
        
    - never 타입은 모든 타입의 하위 타입이다. 즉 자신을 제외한 어떤 타입도 never 타입에 할당될 수 없다. 심지어 any도!
5. **Array 타입**
    - JS의 배열은 어떤 값이든 배열의 원소로 허용한다. 하지만 이 특징으타입스크립트의 정적 타이핑과 잘 부합하지 않는다.
        
        ```jsx
        const array = [1, "string", fn];
        ```
        
    - 따라서 명시적인 타입을 선언하여 해당 타입의 원소를 관리하는 것을 강제한다.
        
        ```jsx
        const array: number[] = [1, 2, 3];
        ```
        
    - 배열의 길이까지 제한하려면 튜플을 사용하면 된다.
6. enum 타입
    - TS에서 지원하는 특수한 타입이다. enum을 사용해서 열거형을 정의할 수 있으며 JS의 객체와 모양이 닮았다.
        
        ```jsx
        enum ProgrammingLanguage {
          Typescript = "Typescript",
          Javascript = "Javascript",
          Java = 300,
          Python = 400,
          Kotlin, // 401
          Rust, // 402
          Go, // 403
        }
        ```
        
    - 주로 문자열 상수를 생성하는데 사용된다.
        
        ```jsx
        enum ItemStatusType {
          DELIVERY_HOLD = "DELIVERY_HOLD", // 배송 보류
          DELIVERY_READY = "DELIVERY_READY", // 배송 준비 중
          DELIVERING = "DELIVERING", // 배송 중
          DELIVERED = "DELIVERED", // 배송 완료
        }
        
        const checkItemAvailable = (itemStatus: ItemStatusType) => {
          switch (itemStatus) {
            case ItemStatusType.DELIVERY_HOLD:
            case ItemStatusType.DELIVERY_READY:
            case ItemStatusType.DELIVERING:
              return false;
            case ItemStatusType.DELIVERED:
            default:
              return true;
          }
        };
        ```
        
    - 주의할 점은 숫자로 이뤄져있거나 TS가 자동으로 추론한 열거형은 안전하지 않은 결과를 낳을 수 있다.
        
        ```jsx
        ProgrammingLanguage[200]; // undefined를 출력하지만 별다른 에러를 발생시키지 않는다
        
        // 다음과 같이 선언하면 위와 같은 문제를 방지할 수 있다
        const enum ProgrammingLanguage {
          // ...
        }
        ```
        
    - 하지만 const enum으로 열거형을 선언하더라도 숫자 상수로 관리되는 열거형은 선언한 값 이외의 값을 할당하거나 접근할 때 이를 방지하지 못한다.
        
        ```jsx
        const enum NUMBER {
          ONE = 1,
          TWO = 2,
        }
        
        const myNumber: NUMBER = 100; // NUMBER enum에 정의되지 않은 값이지만, 숫자형 enum은 컴파일 시점에 값이 인라인 처리되어 타입 오류가 발생하지 않을 수 있다.
        ```
        
        - 음?
            
            ```tsx
            const enum NUMBER {
              ONE = 1,
              TWO = 2,
            }
            
            const myNumber: NUMBER = 100; // error: Type '100' is not assignable to type 'NUMBER'.
            ```
            
            - 컴파일 순서
                1. 타입 검사 단계: TS가 타입 오류 확인
                2. 트랜스파일 단계: JS로 변환하면서 인라인 처리
                
                위 코드는 1단계에서 타입 에러가 발생해서 컴파일이 중단됨. 인라인 처리는 타입 검사 통과 후에만 발생하는데 책에는 컴파일 시점에 값이 인라인 처리되어 타입 오류가 발생하지 않을 수 있다고 되어 있음🤨
                
    - 반면 문자열 상수 방식으로 선언한 열거형은 미리 선언하지 않은 멤버로 접근을 방지한다.
        
        ```jsx
        const enum STRING_NUMBER {
          ONE = "ONE",
          TWO = "TWO",
        }
        
        const myStringNumber: STRING_NUMBER = "THREE"; // Error
        ```
        
    - 이외에도 일부 번들러에서 트리쉐이킹 과정 중 즉시 실행 함수로 변환된 값을 사용하지 않는 코드로 인식하지 못하는 경우가 발생할 수 있다.
        
        따라서 const enum 또는 as const assertion을 사용해서 유니온 타입으로 열거형과 동일한 효과를 얻는 방법이 있다.
        

## 3.2 타입 조합

1. **교차 타입**
    - 여러가지 타입을 결합해서 하나의 단일 타입으로 만들 수 있다.
2. **유니온 타입**
    - 타입 A와 타입 B 중 하나가 될 수 있는 타입을 말한다.
3. **인덱스 시그니처**
    - 특정 타입의 속성 이름은 알 수 없지만 속성값의 타입을 알고 있을 때 사용하는 문법
    
    ```tsx
    type ObjectType = {
      [key: string]: ValueType;
    }
    ```
    
4. **인덱스드 엑세스 타입**
    - 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용된다.
    
    ```tsx
    type Example = {
      a: number;
      b: string;
      c: boolean;
    };
    
    type IndexedAccess = Example["a"];
    type IndexedAccess2 = Example["a" | "b"]; // number | string
    type IndexedAccess3 = Example[keyof Example]; // number | string | boolean
    
    type ExAlias = "b" | "c";
    type IndexedAccess4 = Example[ExAlias]; // string | boolean
    ```
    
    - 배열 요소 타입 조회를 위해 사용하는 경우도 있다.
    
    ```tsx
    const PromotionList = [
      { type: "product", name: "chicken" },
      { type: "product", name: "pizza" },
      { type: "card", name: "chee-up" },
    ];
    
    type ElementOf<T> = typeof T[number];
    
    // type PromotionItemType = { type: string; name: string }
    type PromotionItemType = ElementOf<PromotionList>;
    ```
    
5. **맵드 타입**
    - 기존 타입의 각 속성을 순회하면서 새로운 타입을 생성해준다. 배열의 map 메서드와 비슷하다.
    
    ```tsx
    const BottomSheetMap = {
      RECENT_CONTACTS: RecentContactsBottomSheet,
      CARD_SELECT: CardSelectBottomSheet,
      SORT_FILTER: SortFilterBottomSheet,
      PRODUCT_SELECT: ProductSelectBottomSheet,
      REPLY_CARD_SELECT: ReplyCardSelectBottomSheet,
      RESEND: ResendBottomSheet,
      STICKER: StickerBottomSheet,
      BASE: null,
    };
    
    export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;
    
    // 불필요한 반복이 발생한다
    type BottomSheetStore = {
      RECENT_CONTACTS: {
        resolver?: (payload: any) => void;
        args?: any;
        isOpened: boolean;
      };
      CARD_SELECT: {
        resolver?: (payload: any) => void;
        args?: any;
        isOpened: boolean;
      };
      SORT_FILTER: {
        resolver?: (payload: any) => void;
        args?: any;
        isOpened: boolean;
      };
      // ...
    };
    
    // Mapped Types를 통해 효율적으로 타입을 선언할 수 있다
    type BottomSheetStore = {
      [index in BOTTOM_SHEET_ID]: {
        resolver?: (payload: any) => void;
        args?: any;
        isOpened: boolean;
      };
    };
    ```
    
6. 템플릿 리터럴 타입
    - JS의 템플릿 리터럴 문자열을 사용하여 문자열 리터럴 타입을 선언할 수 있는 문법이다.
    
    ```tsx
    type Stage =
      | "init"
      | "select-image"
      | "edit-image"
      | "decorate-card"
      | "capture-image";
    type StageName = `${Stage}-stage`;
    // ‘init-stage’ | ‘select-image-stage’ | ‘edit-image-stage’ | ‘decorate-card-stage’ | ‘capture-image-stage’
    ```
    
7. 제네릭
    - **타입을 미리 정하지 않고 사용할 때 타입을 결정할 수 있게 해준다.**
    - 함수의 매개변수와 비슷한 개념으로 “타입의 매개변수” 라고 생각하면 된다.
    - 특정 타입에만 존재하는 멤버를 참조하려고 하는 경우 에러가 발생한다. 따라서 제네릭 꺾쇠괄호 내부에 ‘length 속성을 가진 타입만 받는다’라는 제약을 걸어줌으로써 length 속성을 사용할 수 있다.
    
    ```tsx
    interface TypeWithLength {
      length: number;
    }
    
    function exampleFunc2<T extends TypeWithLength>(arg: T): number {
      return arg.length;
    }
    ```
    

## 3.3 제네릭 사용법

1. 함수의 제네릭
    - 어떤 함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 제네릭을 사용할 수 있다.
    
    ```tsx
    function ReadOnlyRepository<T>(target: ObjectType<T> | EntitySchema<T> | string):
    Repository<T> {
      return getConnection(“ro”).getRepository(target);
    }
    
    // 문자열로 사용
    const userRepo2 = ReadOnlyRepository<User>("User");
    const user = await userRepo2.findOne({ where: { id: 1 } });
    ```
    
2. 호출 시그니처의 제네릭
    - 함수 타입을 정의할 때 사용하는 제네릭이다.
    - 함수의 매개변수와 반환 타입을 미리 선언하는 것을 말한다.
    - 언제 사용하면 좋을까?
        - API 요청/응답
        
        ```tsx
        // API 클라이언트 타입 정의
        interface SimpleApiClient {
          get<T>(url: string): Promise<T>;
          post<TRequest, TResponse>(url: string, data: TRequest): Promise<TResponse>;
        }
        
        // 구현
        const apiClient: SimpleApiClient = {
          get: async <T>(url: string): Promise<T> => {
            const response = await fetch(url);
            return response.json();
          },
          
          post: async <TRequest, TResponse>(url: string, data: TRequest): Promise<TResponse> => {
            const response = await fetch(url, {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify(data)
            });
            return response.json();
          }
        };
        
        // 사용 예시
        interface User {
          id: number;
          name: string;
        }
        
        interface CreateUserRequest {
          name: string;
          email: string;
        }
        
        // 각 호출마다 다른 타입 지정
        const user = await apiClient.get<User>('/api/users/1');
        const users = await apiClient.get<User[]>('/api/users');
        
        const newUser = await apiClient.post<CreateUserRequest, User>('/api/users', {
          name: '김철수',
          email: 'kim@example.com'
        });
        ```
        
        - 이벤트 시스템
        
        ```tsx
        // 이벤트 핸들러 타입 정의
        interface EventHandler {
          on<T>(eventName: string, callback: (data: T) => void): void;
          emit<T>(eventName: string, data: T): void;
        }
        
        // 구현
        const eventHandler: EventHandler = {
          on: <T>(eventName: string, callback: (data: T) => void): void => {
            // 이벤트 리스너 등록 로직
            console.log(`${eventName} 이벤트 등록됨`);
          },
          
          emit: <T>(eventName: string, data: T): void => {
            // 이벤트 발생 로직
            console.log(`${eventName} 이벤트 발생:`, data);
          }
        };
        
        // 사용 - 각 이벤트마다 다른 타입
        eventHandler.on<string>('userLogin', (username) => {
          console.log(`${username}님이 로그인했습니다`);
        });
        
        eventHandler.on<number>('scoreUpdate', (score) => {
          console.log(`점수가 ${score}점으로 업데이트되었습니다`);
        });
        
        eventHandler.on<User>('userRegistered', (user) => {
          console.log(`새 사용자: ${user.name}`);
        });
        
        // 이벤트 발생
        eventHandler.emit<string>('userLogin', '홍길동');
        eventHandler.emit<number>('scoreUpdate', 100);
        eventHandler.emit<User>('userRegistered', { id: 1, name: '김철수' });
        ```
        
        - **호출 시그니처 제네릭의 특징**
            1. **하나의 함수로 여러 타입 처리**: 같은 함수를 다양한 타입으로 호출 가능
            2. **호출 시 타입 결정**: 함수를 호출할 때마다 타입을 새로 지정
            3. **유연성**: 매번 다른 타입 조합으로 사용 가능
3. 제네릭 클래스 
    - 외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스이다.
    
    ```tsx
    class Box<T> {
      private value: T;
      
      constructor(value: T) {
        this.value = value;
      }
      
      getValue(): T {
        return this.value;
      }
      
      setValue(value: T): void {
        this.value = value;
      }
    }
    
    // 사용 - 클래스 생성할 때 타입 지정
    const stringBox = new Box<string>("안녕하세요");
    const numberBox = new Box<number>(42);
    const booleanBox = new Box<boolean>(true);
    
    console.log(stringBox.getValue()); // "안녕하세요"
    console.log(numberBox.getValue());  // 42
    console.log(booleanBox.getValue()); // true
    
    // 타입 안전성 보장
    stringBox.setValue("새로운 문자열"); // ✅ 정상
    // stringBox.setValue(123);          // ❌ 컴파일 에러
    ```
    
4. 제한된 제네릭
    - 타입 매개변수에 대한 제약 조건을 설정하는 기능을 말한다.
    - 예시
        - extends 키워드 사용
        
        ```tsx
        // T는 반드시 length 속성을 가져야 함
        function getLength<T extends { length: number }>(item: T): number {
          return item.length;
        }
        
        // 사용
        console.log(getLength("hello"));      // 5 (string은 length 속성 있음)
        console.log(getLength([1, 2, 3]));    // 3 (array는 length 속성 있음)
        // console.log(getLength(123));       // ❌ 에러 (number는 length 속성 없음)
        ```
        
        - 키 제약 조건 (keyof 사용)
        
        ```tsx
        function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
          return obj[key];
        }
        
        const person = {
          name: "홍길동",
          age: 25,
          city: "서울"
        };
        
        // 사용
        const personName = getProperty(person, "name");  // string
        const personAge = getProperty(person, "age");    // number
        // const invalid = getProperty(person, "height"); // ❌ 에러 (없는 속성)
        ```
        
5. 확장된 제네릭
    - 제네릭의 유연성을 잃지 않으면서 타입을 제약해야 할 때는 타입 매개변수에 유니온 타입을 상속해서 선언하면 된다.
        
        ```tsx
        <Key extends string | number>
        ```
        
6. 제네릭 예시
    - 현업에서는 API 응답 값의 타입을 지정할 때 가장 많이 사용한다.
    
    ```tsx
    export interface MobileApiResponse<Data> {
      data: Data;
      statusCode: string;
      statusMessage?: string;
    }
    
    export const fetchPriceInfo = (): Promise<MobileApiResponse<PriceInfo>> => {
      const priceUrl = "https: ~~~"; // url 주소
    
      return request({
        method: "GET",
        url: priceUrl,
      });
    };
    
    ```
    
    - 굳이 필요하지 않은 곳에는 제네릭을 사용하면 오히려 독이된다.
        - 제네릭을 굳이 사용하지 않아도 되는 타입
            
            ```tsx
            type GType<T> = T;
            type RequirementType = "USE" | "UN_USE" | "NON_SELECT";
            interface Order {
              getRequirement(): GType<RequirementType>;
            }
            ```
            
        - any 사용하기
            
            ```tsx
            type RequirementType = "USE" | "UN_USE" | "NON_SELECT";
            interface Order {
              getRequirement(): RequirementType;
            }
            ```
            
        - 가독성을 고려하지 않은 사용
            
            ```tsx
            type ReturnType<T = any> = {
              // ...
            };
            ```
