# GraphQl

## 정의
GraphQL은 API를 위한 쿼리 언어로, Rest Api와 비교하여 요구 사항에대해 유연한 대처가 가능하고<br> 언더 페칭, 오버 페칭등을 방지할 수 있는 언어이다.

<br>

## 개념

- ### Schema

    스키마는 데이터의 구조를 명시하며, 클라이언트가 어떤 쿼리를 할 수 있는지, 서버에서 어떤 유형의 데이터를 페치할 수 있는지, 그리고 이러한 유형 간의 관계가 무엇인지 정의한다.

    ### Type : 데이터 정의

    ```graphql
    type Member {
        id: ID!
        name: String!
        age: Int!
    }
    ```
    ### Query : 클라이언트가 피룡한 데이터를 명시하여 서버에 요청

    ```graphql
    type Query {
        getMember(id: ID!): Member
        getMembers: [Member]
    }
    ```

    ### Mutation : 데이터 생성, 업데이트, 삭제 요청 처리

    ```graphql
    type Mutation {
        save(name: String!, age: Int!): Member
    }
    ```

- ### 다양한 방식

    GraphQL은 편의를 위해 다양한 사용방법을 제공합니다.

    <br>

    ### Union : GraphQL 스키마에서 하나의 필드가 둘 이상의 타입 중 하나를 반환할 수 있도록 정의하는 타입입니다.

    ```graphql
    union SearchResult = Book | Author

    type Book {
        title: String!  
    }

    type Author {
        name: String!
    }

    type Query {
        search(contains: String): [SearchResult!]
    }
    ```

    ### Interface : Interface는 Java와 비슷하게 공통된 필드를 추상화시켜 여러 Type에서 상속받아 사용할 수 있게 합니다.

    ```graphql
    interface Character {
        id: ID!
        name: String!
    }

    type Human implements Character {
        id: ID!
        name: String!
        homePlanet: String
    }

    type Droid implements Character {
        id: ID!
        name: String!
        primaryFunction: String
    }

    type Query {
        character(id: ID!): Character
    }   
    ```

    ### Enum : Enum또한 Java와 비슷하게 사전에 정의한 값을 사용하는 타입입니다.
    
    ```graphql
    enum State {
        FINISH
        INPROGRESS
    }

    type Character {
        id: ID!
        name: String!
        status: [State!]!
    }
    ```

    ### Fragment :  fragment는 쿼리에서 공통적으로 사용되는 필드 집합을 재사용할 수 있도록 정의하는 방법입니다.

    ```graphql
    fragment CharacterFields on Character {
        id
        name
        appearsIn
    }

    {
        hero(episode: NEWHOPE) {
            ...CharacterFields
        }
        villain(episode: NEWHOPE) {
            ...CharacterFields
        }
    }
    ```

    ### Directive : 쿼리나 스키마의 특정 부분을 동적으로 수정할 수 있도로 하는 것입니다. QueryDsl의 BooleanExpression 과 비슷함.

    - @include(if: Boolean!): 조건이 참일 때만 필드를 포함.
    - @skip(if: Boolean!): 조건이 참일 때 필드를 생략.

    ```graphql
    query Hero($withFriends: Boolean!) {
        hero {
            name
            friends @include(if: $withFriends) {
            name
            }
        }
    }
    ```

    ### Instrumentation : 쿼리 실행을 관찰하고 런타임 동작을 변경할 수 있는 방법입니다.

    ```java
    //ex)
    @Configuration
    public class InstrumentationConfiguration {
        @Bean
        public Instrumentation someFieldCheckingInstrumentation() {
            return new FieldValidationInstrumentation(env -> {
                // ... 
            });
        }
    }
    ``` 
<br>

## 동작 순서

1. 요청 수신
2. 쿼리 파싱 & 검증(GraphQl 구문 오류, Validation Schema Check)
3. 리졸버 선택 & 실행
4. 응답 객체 생성 후 반환

<br>

## 에러 핸들링

GraphQL 자체는 에러가 발생해도 구문 에러가 아닌이상 200이 반환되며 응답 필드에 error field가 추가되어 나갑니다.

<br>

## 장단점

- ### 장점
1. 하나의 엔드 포인트로 다양한 쿼리와 변화를 처리 할 수 있습니다.
2. 오버페칭, 언더페칭을 방지 합니다.
3. 불필요한 DTO class를 줄입니다.

- ### 단점
1. 캐싱이 Rest Api 에 비해 어렵습니다.
2. 파일 요청이 어렵습니다.
3. 지원 필드가 한정적입니다.