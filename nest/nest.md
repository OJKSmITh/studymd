<img src="../img폴더/nestjs.png"/>

# NestJs
Node js 서버 측 애플리케이션 구축을 위한 프레임 워크라고 생각하면 된다.

근데 왜 프레임 워크가 나오게 됐는가? 결국에는 유지보수 문제이다. express의 장점이자 단점은 결국 **무한한 자유성**이다. 

이를 통해 정말 획기적인 결과가 나올 수도 있지만, 반대로 다른사람이 코드를 읽거나, 유지보수하는데는 어려움이 있을 수 밖에 없다. 

NestJs는 ts 빌드를 완벽하게 지원하며 사용자에게 편리성을 강요하지만, 반대로 이는 express의 기본적인 지식과 + nestjs 지식이 필요하다는 것이다. 

# NestJs 시작하기
필자는 m1 맥북을 사용중에 있으며 떄문에 NestJs CLI 로 설치를 하였다.
```sh
npm i -g @nestjs/cli
nest --version -> 버전 확인용
nest new [폴더이름]
```
아롷개 설치를 하고 나면 디폴트 값들이 설정이 되어있다. 

CRUD를 통해서 좀 더 자세히 Nest에 대해서 알아보기 전에 Nest 기본 흐름에 대해서 알아보자. 

# Nest JS 기본 로직
main.ts -> app.module(루트 모듈) -> app.controler -> app.service

여기서 루트 모듈인 모듈은 @Module 데코레이터로 주석이 달린 클래스로 Nest가 애플리케이션 구조를 구성하는데 사용하는 `메타데이터`를 제공합니다.

☒ 메타데이터는 다른 **데이터를 구조화하는 데이터**로 이해하면 됩니다. 

# Provider
프로바이더는 Nest의 기본 개념으로 많은 기본 Nest class는 서비스, 레파지토리, 팩토리, 헬퍼 등등의 프로바이더로 취급이 가능합니다. 

프로바이더의 주요 아이디어는 **의존성을 주입할수 있다는 것**입니다. 이 뜻은 객체가 서로 다양한 관계를 만들 수 있다는 것을 의미하고, 객체의 인스턴스를 연결해주는 기능은 Nest 런타입 시스템에 위임될 수 있다는 것인데...

사실 이 말만 보면 무슨 말인지 이해하기 어렵습니다.

이를 이해하기 위해서는 3가지 선행지식이 필요합니다.
- 계층형 구조(Layered Architecture)
- 제어 역전(IoC, Inversion of Control)
- 의존성 주입(Dependency Injection)
자 이걸 하나하나 뜯어봅시다.

## 계층형구조(Layered Architecture)
하나의 ts 파일에 모든 카카오 로그인 로직이 있다고 해봅시다

그러면 인증받는거, 리다이렉트 url 넣는것, 그리고 token 발급하는거... 참 많은 로직이 한 페이지에 들어가고

나중에 그 코드를 보면 어떤 코드인지 이해하기 어려울 것입니다.

즉 더 크고, 복잡한 코드를 만드는게 어려워진다는 말입니다. 이런 문제 해결을 위해서 등장한 것이 소프트웨어 업계에서 `계층형구조`라는 기법이 등장하게 됐습니다.

사실 말이 복잡하지 결국 복잡한 작업을 작업을 나누어서 개발하자는 것입니다. 한 사람의 풀스택 개발자가 큰 웹서비스 전체를 만들기 어렵지만 프론트와 백엔드를 나누면 좀 더 쉽고 빠르게 만들 수 있다는 것과 동일합니다.

몇개의 계층으로 구분하냐에 따라 다르지만 보통 **3계층 구조**를 많이 사용합니다. 영어로는 `3-Tier Architecture`라고 합니다. 3계층은 아래와 같이 나눕니다.

- Presentation Tier: 사용자 인터페이스 혹은 외부와의 통신을 담당합니다. 
- Application Tier: logic Tier 라고 하기도 하고 Middle Tier라고 하기도 합니다. 주로 비즈니스 로직을 여기서 구현하고 Presentation Tier와 Data tier 사이를 연결합니다.
- Data Tier: 데이터베이스에 데이터를 읽고 쓰는 역활을 담당합니다. 

Nest에서 Presentation Tier는 **외부의 입력**을 받아들이는 컨트롤러 이며 서비스는 ApplicationTier에 해당합니다. 서비스에는 주로 비즈니스 로직이 들어가고, Nest는 이렇게 컨트롤러와 그 하위 계층을 프로바이더 라는 이름으로 구분하며 이는 

**응집도는 높이고 결합도는 낮추는 소프트웨어 설계입니다**.

## 제어역전
제어 역전을 한다디로 표현하면 **<span style="color:red">나 대신 프레임워크가 제어한다</span>** 입니다. 

제어역전을 알기 위해서는 의존성이라는 개념이 필요한데요

타입스크립트를 비롯한 많은 언어에서는 클래스를 사용하려면 new 같은 키워드로 인스턴스화 시켜야 합니다. 모름지기 사람이라면 붕어빵을 먹지 붕어빵틀로 먹지는 않는것과 같습니다. 

코드를 통해 더 살펴봅시다.

```ts
const sword = new Sword();
```
warrior 클래스에서 Sword 클래스를 위와같이 인스턴스화 했다고 해봅시다. 근데 이제 sword 뿐만 아니라 몽둥이도 사용하도록 하자고 합니다.

sword를 사용하는 곳이 백만곳이 넘는다면 똑같이 몽둥이도 일일히 백만개를 쳐줘야 하는 아찔한 상황입니다. 

이런 문제를 해결하기 위해서 등장한 것이 바로 **인터페이스** 입니다.

프로그래밍에서 인터페이스란 **규약**입니다.

인터페이스를 구현하려면 인터페이스가 원하는 규역을 따라야 하고, 반대급부로 프로그래머는 내가 호출하는 클래스가 무엇인지 정확하게 알 필요가 없습니다. 다만 특정 기능이 동작 가능하다는 사실만 알고 개발하면됩니다.

다시 코드를 통해 살펴봅시다.

```ts
inferfae Weaponable {
    swing(): void;
}
inferfae Playable {
    attack(): void;
}

class Warrior implements Playable {
  private Weaponable weapon;

  constructor Warrior(private readonly Weaponable _weapon) {
    weapon = _weapon;
  }

  public void attack() {
    weapon.swing();
  }
}

class Mongdungee implements Weaponable {
  public void swing() {
    console.log('Mongdungee Swing!');
  }
}
```
몽둥이를 쥔 전사 클래스를 인스턴스화 시켜봅시다.
```ts
Warrior warrior = new Warrior(new Mongdungee())
```
이렇게 보면 굉장히 쉬워보입니다. 하지만 클래스 계층 구조가 복잡한 프로그램에서 사람이 직접 저 몽둥이를 넘긴다거나, 여러 전사에게 같은 몽둥이를 넘겨야 하는 상황이 좋지는 않습니다. 

이 경우 사용하는 것이 바로 `제어역전`입니다. Nest는 제어 역전을 추상화해서 그 동작이 잘 보이지 않기 때문에 `typedi`를 통해서 이를 살펴봅시다.

```ts
import "reflect-metadata";
import { Container, Service } from "typedi";

inferfae Weaponable {
    swing(): void;
}
inferfae Playable {
    attack(): void;
}

@Service()
class Mongdungee implements Weaponable {
  public void swing() {
    console.log('Mongdungee Swing!');
  }
}

@Service()
class Warrior implements Playable {
  // 아래 코드 중요!
  constructor(private readonly weapon: Weaponable) {}

  public void attack() {
    this.weapon.swing();
  }
}


const playerInstance = Container.get<Warrior>(Warrior);
playerInstance.attack();  // "Mongdungee Swing!"
```
코드에서도 `new`가 존재하지 않습니다. 하지만 잘 동작하는데, 그 이유는 바로 `typedi`의 Container가 알아서 클래스의 인스턴스를 생성했기 때문입니다. 

좀 더 자세히 설명해보자면 `@Service()`은 `typedi`에게 `service 프로바이더` 임을 알리고 등록이 됩니다.

이후 사용자가 Warrior클래스를 생성하는데 `Warrior` 클래스에서는 `Weaponable` 객체가 필요하고 등록된 유일한 Weaponable 객체는 Mongdungee 이기 때문에 

그 하나를 자동으로 생성해서 `Container`가 넣어줍니다. 이렇게 유저가 생성하는것이 아니라 `typedi`

이처럼 제어권을 내가 아닌 프레임워크에게 넘기는것이 바로 제어역전이라고 합니다. 

여기서 바로 **<span style="color:red">라이브러리와 프레임워크의 결정적인 차이</span>**가 등장하는데

바로 라이브러리는 작성자가 필요할때 라이브러리를 실행시킬수 있는반면, 프레임워크는 프레임 워크 스스로 필요할때 코드를 알아서 실행시킨다는 것입니다. 

## 의존성 주입(DI/ Dependenct Injection)
제어 역전은 나 대신 프레임워크가 제어한다라면 의존성 주입은 프레임워크가 주체가 되어 네가 필요한 클래스 등을 너 대신 내가 관리한다는 개념이라고 생각하면 됩니다. 

좀 더 자세히 설명한다면 의존성 주입은 다른곳에서 객체를 만들어서 그 객체를 가지고 다른 곳에서 사용하는 즉 하나를 만들어서 사용한다고 생각하면 됩니다. 

이렇게 들으면 어? **제어 역전이랑, 의존성 주입이랑 뭔 차인가요**? 라고 할 수 있습니다. 

멀리서 보면 제어역전은 추상적인 개념이고, 이를 구체적으로 구현한게 바로 DI라고 할 수 있는 것입니다. 

## Re Provider
다시 공식문서로 들어가면

프로바이더는 Nest의 기본 개념입니다. 많은 기본 Nest 클래스는 서비스, 레파지토리, ㄷ팩토리, 헬퍼 등등의 프로바이더로 취급하고, 프로바이더의 주요 아이디어는 **의존성을 주입할 수 있다는 것**입니다. 

이 뜻은 객체가 서로 다양한 관계를 만들 수 있다는 것을 의미하고, 객체의 인스턴스를 연결해주는 기능은 Nest 런타임 시스템에 위임될 수 있습니다. 

이를 우리가 봤던 코드로 보자면 `Warrior`의 Mongdungee가 바로 프로바이더 입니다. **어떤 컴포넌트가 필요하며 의존성을 <span style="color:red">주입당하는 객체</span>를 프로바이더라고 생각하면 됩니다**.

그리고 Nest 프레임워크 내부에서 알아서 컨테이너를 만들어서 관리해준다는 말로 이해하시면 되겠습니다. 

## Provider 예제 - Service
간단한 캣츠 서비스부터 만들어 봅시다. CatsController가 사용하도록 설계되었으므로 프로바이더로 정의하기 좋습니다.

```ts
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}

cli 사용할 시 `nest g service cats` 사용하면 됩니다.
```
`CatsService`는 하나의 속성과 두 개의 메소드를 가진 기본 클래스입니다. 유일한 새로운 기능은 @Injectable()데코레이터를 사용한다는 것 뿐입니다. @Injectable() 데코레이터는 메타 데이터를 첨부하여 CatsServicerk Nest IoC 컨테이너 에서 관리할 수 있는 클래스입니다.

이 예제에서는 Cat 인터페이스도 사용하는데 아마 다음과 같은 코드일 것입니다.

```ts
// interfaces/cat.interface.ts
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```
자 그러면 이제 이 코드를 CatsController 안에서 사용해 봅시다.

```ts
// cats.controller.ts
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

CatsService는 컨트롤러 클래스 생성자를 통해서 **주입**됩니다. 생성자의 파라미터에 주목해봅시다. 보통 Java에서 객체의 생성자는 아래와 같습니다.

```java
public class User {
  private String name;
  constructor(String name) {
    this.name = name;
  }
}
```

```ts
class User{
    constructor(private name:string){}
}
```

자 여기서 또 한가지 알아야 할것은 의존성을 주입하는 방법은 총 3가지라는 것입니다.
- 생성자를 이용한 의존성 주입(Constructor Injection)
- 수정자를 이용한 의존성 주입(Setter Injection)
- 필드를 이용한 의존성 주입(Field Injection)
Nest에서는 주로 **생성자를 이용한 의존성 주입**을 권장합니다. 필드를 이용한 의존성 주입은 한 두 스크롤 아래 `속성기반 주입이라는 항목으로 소개`합니다.

## 범위
프로바이더는 일반적으로 Nest 프로그램의 수명 주기와 동기화 된 수명(범위)을 갖습니다. Nest 프로그램이 `☒부트 스트랩` 될때 모든 종속성을 해결해야 하기 떄문에 **모든 프로바이더가 인스턴스화** 됩니다. 

마찬가지로 Nest 프로그램이 종료되면 각 프로바이더가 메모리에서 삭제 됩니다. 그러나 프로바이더의 수명을 요청 단위로 제한하는 방법도 있습니다. 다만 성능에 문제가 될 수 있기 때문에 특수한 상황이 아니라면 기본 설정된 수명 주기를 사용합시다. 

## 선택적 프로바이더
떄때로 반드시 해결할 필요가 없는 종속성이 있을 수 있습니다. 예를들어 클래스는 configuration 객체에 의존할 수 있지만 해당 인스턴스가 없는 경우 기본값을 사용하는 경우입니다.

이러한 경우 에러가 발생하지 않으므로 종속성이 선택사항이 됩니다.

```ts
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

## 프로바이더 등록
우리는 프로바이터(`catsService`)를 정의했고, 그 서비스의 소비자(CatsController)를 가지고 있으므로, 주입을 수행할 수 있도록 Nest에 서비스를 등록해야 합니다. 모듈파일(app.module.ts)를 편집하고 서비스를 

@Module() 데코레이터의 Prociders 배열에 추가하면 됩니다. 



## module
모듈에 대해서 좀 더 설명을 가지자면 `@Module` 데코레이터는 아래 속성을 가지는 객체가 필요합니다. 이 객체들이 바로 모듈을 구성하는 요소들입니다.

- providers(프로바이더): Nest 인젝터(Injector: **의존성**을 주입하는 Nest 내부 모듈)가 인스턴스화 시키고 적어도 이 모듈 안에서 공유하는 프로바이더를 말합니다.
- controllers(컨트롤러): 이 모듈안에서 정의된, 인스턴스화 되어야 하는 컨트롤러의 집합
- imports: 해당 모듈에서 필요한 모듈의 집합, 여기에 들어가는 모듈은 프로바이더를 노출하는 모듈입니다.
- exports: 해당 모듈에서 제공하는 프로바이더의 부분집합이며, 이 모듈을 가져오는 다른 모듈에서 사용할 수 있도록 노출할 프로바이더

모듈은 기본적으로 프로바이더를 캡슐화합니다. 즉, 현재 모듈에 직접 속하지 않거나, 가져온 모듈에서 노출하지 않는 프로바이더를 주입할 수 없습니다. 따라서 모듈에서 노출한 프로바이더를 모듈의 공용 인터페이스 도는 API로 간주 할 수 있습니다.

## 기능모듈
CatsController와 CatsService는 둘다 고양이를 다루니 같은 도메인에 속하게 됩니다. 밀접하게 관련되어 있으므로 기능 모듈로 이동하는것이 좋습니다. 기능 모듈은 특정 기능과 관련된 코드를 구성하여 코드를 체계적으로 유지하고 명확한 경계를 설정합니다.

이는 app이나 팀의 규모가 커짐에 따라 커지는 복잡성을 관리하는데 도움을 주고 SOLID 원칙으로 개발하는데 도움이 됩니다.

```ts
// cats/cats.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```
우리는 CatsModules을 정의했습니다. 그리고 이 모듈과 관련된 모든 것을 cats 디렉토리로 옮겼습니다. 마지막으로 해야할 일은 이 모듈을 루트 모듈로 가져오는 것입니다. 

```ts
// app.module.ts

import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

## 공유 모듈
Nest에서 모듈은 기본적으로 singleton 입니다. 이는 Nest 고유의 특성이 아니라 Node의 특성입니다. 이런 특성 떄문에 Nest에서는 여러 모듈간에 쉽게 공급자의 동일한 인스턴스를 공유할 수 있습니다. 

모든 모듈은 자동으로 공유 모듈입니다. 즉 공유가 가능한 모듈이라는 뜻이며 일단 생성되면 모든 모듈에서 재사용 할 수 있다는 뜻입니다. 다른 여러 모듈간에 CatsService의 인스턴스를 상황을 가정해보겠습니다.

그렇게 하려면 먼저 아래와 같이 모듈의 exports 배열에 CatsService 프로바이더를 추가하여 프로바이더를 노출해야 합니다. 

```ts
//cats.module.ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```
이제 CatsModule 을 가져오는 모듈에서는 CatsService에 접근이 가능하며, 이 모듈을 **가져오는 다른 모듈과 동일한 인스턴스를 <span style="color:red">공유</span>**합니다. 

## 모듈 다시 내보내기
위에서 볼 수 있듯이 모듈은 내부의 프로바이더를 노출할 수 있습니다. 또한 가져온 모듈은 다시 내보낼 수 있습니다. CommonModule을 가져와서 이를 바로 노출합니다. 그럼 CoreModule을 가져오는 다른 모듈에서는 CommonModule을 사용할 수 있게 됩니다. 

## 의존성 주입
모듈 클래스도 프로바이더를 주입할 수 있습니다. 예를들어 설정관련 목적입니다.

```ts
// cats.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
```
그러나 모듈 클래스 자체는 순환 종속성으로 인해 프로바이더로 주입할 수 없습니다. 

## 전역 모듈
모든 곳에서 동일한 모듈을 가져오는 일은 굉장히 중복적인 일입니다. 별도의 절차없이 모듈을 전역으로 뿌리는 방법도 분명히 존재합니다. 

Nest 가 Angular에게 영감을 받아서 만들어졌지만 하지만 Angular와는 달리 Nest는 전역적으로 프로바이더를 등록할 수 없습니다. 

Nest에서는 프로바이더는 모듈을 벗어날 수가 없기 떄문입니다. 해당 프로바이더를 다른 곳에서 사용하려면 해당 프로바이더가 속한 모듈을 먼저 가져오지 않으면 안됩니다.

어디서나 사용할 수 있어야 하는 프로바이더를 제공하려면 `@Global()`데코레이터를 사용하여 해당 모듈을 전역 모듈로 만드시기를 바랍니다. 

```ts
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```
`@Global()`데코레이터는 모듈을 전역적으로 사용할 수 있도록 만듭니다. 전역 모듈은 일반적으로 루트 또는 코어 모듈에 의해 단 한번만 등록되어야 합니다. 

위의 예에서 CatsService 프로바이더는 어디서나 사용이 가능하며 CatsService 서비스를 주입하려는 모듈은 CatsModule을 모듈의 import 배열에 추가할 필요가 없습니다. 

# 미들웨어 
미들웨어는 express의 미들웨어와 같은 개념입니다. 따라서 클라이언트로부터 들어온 요청을 각 컨트롤러의 요청 핸들러가 처리하기 이전에 코드를 실행할 수 있는 기능입니다. 

미들웨어 함수는 app의 요청-응답 주기에서 요청 및 응답 객체에 접근할 수 있으며 next()라는 미들웨어 함수가 가능합니다. 

미들웨어 기능은 다음과 같습니다.
- 어떠한 코드를 실행할 수 있습니다.
- 요청 및 응답 개체를 변경할 수 있습니다.
- 요청-응답 주기를 종료합니다.
- 스택의 다음 미들웨어 기능을 호출합니다.
- 현재 미들웨어 기능이 요청-응답 주기를 종료하지 않는 경우, 다음 미들웨어 기능으로 제어를 전달하기 위해 next()를 호출해야 합니다. 그렇지 않으면 요청이 보류 됩니다. 

미들웨어는 하나의 함수를 통해 구현되거나 `@Injectable()` 데코레이터가 있는 클래스로 구현이 가능합니다. 

클래스로 구현하려면 `NestMiddleware` 인터페이스를 구현해야 합니다. 먼저 클래스를 사용해서 간단한 미들웨어를 만들어봅시다. 

```ts
// logger.middleware.ts;

import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```
위 미들웨어의 용도는 우선은 콘솔에 Request 라는 문자열을 찍고 끝나기만 합니다. 

## 의존성 주입
Nest미들웨어는 의존성 주입을 완벽하게 지원합니다. 프로바이더 및 컨트롤러와 마찬가지로 동일한 모듈 내에서 사용할 수 있는 의존성을 주입할 수 있습니다. 항상 그렇듯이 이작업은 생성자를 통해 수행됩니다. 

## 미들웨어 적용
`@Module()` 데코레이터를 설정하는 속성에는 imports, exports, providers 등만 있지 미들웨어를 설정하기 위한 속성은 없습니다. 대신 모듈 클래스의 configure() 메서드를 사용하여 설정할 수 있습니다. 

미들웨어를 포함하는 모듈은 NestModule 인터페이스를구현해야 합니다. 이제 Logger 미들웨어를 AppModule 수준에서 설정하겠습니다.

```ts
// app.module.ts

import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('cats');
  }
}
```
이 예제 이전에 단순히 `Request...`라고 콘솔을 찍던 미들웨어가 생각나실 겁니다. 이 미들웨어를 클라이언트에서 /cats 경로의 리소스를 요청할 경우 LoggerMiddleware를 적용하도록 설정했습니다. 

특정 HTTP 메서드만 적용할 수도 있습니다. 예를들어 /cats 경로의 `GET`메서드만 적용하고 싶으시다면 아래와 같이 `RequestMethod`열거형을 불러와서 적용합니다.

```ts
// app.module.ts

import { Module, NestModule, RequestMethod, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes({ path: 'cats', method: RequestMethod.GET });
  }
}
```

## Route wildcards
패턴 기반 라우팅 역시 지원하고 있습니다. 예를들어 별표(*)가 와일드 카드로 사용되고 다음과 같은 문자의 조합과 일치한다고 해봅시다. 

`forRoutes({ path: 'ab*cd', method: RequestMethod.ALL})`

`ab*cd`의 라우팅 경로는 abcd, ab_cd, abce 등과 일치합니다. 하지만 그렇다고 nest에서 모든 정규 표현식을 지원하는것은 아닙니다. 

Nest에서 지원한느 패턴 기반 라우팅은 정규표현식의 부분집합, 즉 모든 정규표현식의 기능을 지원하지 않습니다. 라우팅 경로에서 문자 ?, +, * 및 ()를 사용할 수 있으며

하이픈이나 점은 문자 그대로 해석합니다. 

## 미들웨어 소비자(MiddlewareConsumer)
Nest가 제공하는 헬퍼 클래스입니다. 말 그대로 도와주는 클래스이고, 어떤 특정 클래스의 작업을 도와주는 클래스를 통칭하는 말입니다. 

절대 전면에 나서지 않는 도우미와 같은 존재입니다. 어쨌든 `MiddlewareConsumer`는 말 그대로 미들웨어를 잘 사용하기 위한 유용한 기능을 제공해주는데, 이 기능은 모두 플루언트 스타일로 체이닝을 사용할 수 있습니다.

forRoutes() 메서드에는 단일 문자열, 다중문자열, RouteInfo 개체, 컨트롤러 클래스 및 여러 컨트롤러 클래스가 포함될 수 있습니다. 대부분의 경우 쉼표로 구분된 컨트롤러 목록을 전달하기만 하면 됩니다. 

```ts
// app.module.ts

import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';
import { CatsController } from './cats/cats.controller.ts';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes(CatsController);
  }
}
```
## 경로제외
특정 경로 한 두개에 대해서 미들웨어를 적용시키고 싶지 않을떄도 있을 수 있습니다. exclude() 메서드를 사용하면 특정 경로만 쉽게 제외가 가능합니다. 이 메서드는 다음과 같이 제외할 경로를 식별하는 단일 문자열, 다중 문자열 또는

RouteInfo 객체가 사용이 가능합니다. 

```ts
consumer
  .apply(LoggerMiddleware)
  .exclude({ path: 'cats', method: RequestMethod.GET }, { path: 'cats', method: RequestMethod.POST }, 'cats/(.*)')
  .forRoutes(CatsController);
```
위와 같은 방식으로 다중 경로도 가능합니다.

## 함수형 미들웨어
지금까지 만진 LoggerMiddleware 클래스도 존재하지만 클래스의 멤버도 없고, 메서드 및 종속성도 존재하지 않는다면 간단한 미들웨어는 클래스 대신 간단한 함수로 정의합시다. 
```ts
// logger.middleware.ts

import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
}
```

그리고 이를 `AppModule`에서 사용하면 됩니다. 
```ts
// app.module.ts
consumer.apply(logger).forRoutes(CatsController);
```

## 함수형 미들웨어 여러개 적용하기
```ts
consumer.apply(cors(), helmet(), logger).forRoutes(CatsController);
```

## 전역 미들웨어 
미들웨어를 등록된 모든 경로에 한번에 바인딩 하려면 INestApplication 인스턴스에서 제공하는 use() 메서드를 사용할 수 있습니다.

```ts
const app = await NestFactory.create(AppModule);
app.use(logger);
await app.listen(3000);
```


<a link="https://www.wisewiredbooks.com/nestjs/overview/04-provider.html">참조주소</a>

# 2일차
## CRUD
### Board 모듈 생성하기
```sh
명령어 :nest g module boards

nest : useing nest cli
g: generate
module: module: 생성
boards: modul의 이름
```
이런식으로 명령어를 통해서 자동으로 만들 수 있다는 것이 장점

### Board controller 생성하기
```sh
명령어 :nest g controller boards --no--spec
--no--spec: 원래 테스트 코드까지 생성하지만, 우리는 그냥 만들것이다. 
```

### Board Service 만들기
기본적으로 Repository가 없다면 이 부분에서 데이터베이스 관련 로직을 처리함, DB에서 데이터를 가져오거나 데이터베이스 안에 게시판 생성시 생성한 게시판 정보를 넣는등의 로직을 처리합니다. 

```sh
nest g service boards --no--spec
```
서비스가 만들어지고 업데이트가 됨(module에!)

CLI를 통해서 생성하면 파일이 생성되고 `Injectable 데코레이터`가 있으며 NestJs는 이것을 이용해서 다른 컴포넌트에서 이 서비스를 사용할 수 있게 만듭니다.

board Service를 boardController에서 사용할 수 있게 하려면 ts에서는 다음과 같이 작성하면 됩니다.

```ts
import { Controller } from '@nestjs/common';
import { BoardsService } from './boards.service';

@Controller('boards')
export class BoardsController {
    constructor(private boardsService:BoardsService){}
}
```
이렇게 하면 의존성 주입이 가능해짐, 근데 이 코드가 처음부터 동작하는것은 아닙니다. 원래는 아래와 같이 작성했어야 합니다.

```js
import { Controller } from '@nestjs/common';
import { BoardsService } from './boards.service';

@Controller('boards')
export class BoardsController {
    boardsService: BoardsService
    constructor(boardsService:BoardsService){
        this.boardService = boardSerivce
    }
}
```
원래 js에서는 위와 같이 작성해야 했지만, 이후 private, public등의 **접근제한자**가 등장하게 되면서 코드를 줄여서 사용이 가능해졌다는 것

즉 생성자 안에서 접근제한자를 쓰면 암묵적으로 클래스 프로퍼티가 가능하다는 것이다. 이런식으로 service클래스 주입이 가능해졌다는 것


### 프로바이더 등록하기
프로바이더를 그냥 사용할 수 있는 것은 아니고, 이것을 Nest에 등록해야만 사용이 가능해집니다. 등록을 위해서는 module 파일에서 작성이 가능합니다.

module 파일 안에 해당 모듈에서 사용하고자 하는 Provider를 넣으면 됩니다. 

```ts
import { Module } from '@nestjs/common';
import { BoardsController } from './boards.controller';
import { BoardsService } from './boards.service';

@Module({
  controllers: [BoardsController],
  providers: [BoardsService]
})
export class BoardsModule {}

providers: [BoardsService]
```
이런식으로 정의한 파일을 넣으면 됩니다. 

### boardStatus 정의하기
```ts
export enum BoardStatus {
    PUBLIC= "PUBLIC",
    PRIVATE= "PRIVATE"
}
```
enum은 열거형 타입으로 PUBLI, PRIVATE 등의 값들을 지정할 수 있지만, 없는경우는 0,1 순서대로 나열되며 숫자가 있을 경우에는 하나만 지정해도 다음 숫자가 자동으로 지정된다는 것들이 있음

### DTO
Data transfer object

계층간 데이터 교환을 위한 객체, DB에서 데이터를 얻어 Servicesk Controller 등으로 보낼때 사용하는 객체를 말합니다.

DTO는 데이터가 네트워크를 통해 전송되는 방법을 정의하는 객체를 말하고

interface나 class를 통해서 정의가 가능합니다. 하지만 공식문서에서는 **클래스를 이용하는것을 추천하고 있다는 사실을** 명심합시다.

클래스를 추천하는 이유는 클래스는 인터페이스와 다르게 **런타임**에서 작동하기 때문에 파이프 같은 기능을 이용할때 . 더 유용하며, 그래서 클래스를 이용해서 DTO 를 작성한다는 것입니다. 

근데 왜 굳이 DTO를 거쳐야 해? 라는 것에 대한 물음으로는 **데이터 유효성** 체크를 위해서 효율적이며, 안정적인 코드를 만들어주기때문이라는 것

Board 를 위한 Property들을 여러곳에서 사용하고 있으며, 떄문에 하나씩 정의하는 것보다 한번에 정의하고 그것을 바꾸는것이 더 효율성이 좋다는 것

### NestJs 파이프
파이프란 @Injectable() 데코레이터로 주석이 달린 클래스를 말합니다. 파이프는 data transformation(데이터 변형)과 data validation(유효성 처리)을 위해서 사용됩니다.

파이프는 컨트롤러 경로처리기에 의해 처리되는 인수에 대해 작동하며, 메소드가 **호출되기 직전에 파이프를 삽입**하고 파이프는 메소드로 향하는 인수를 수신하고 이에 대해 작동한 다는 것이고

**미들웨어 느낌**으로 사용한다고 생각하면 이해가 편할거 같습니다. 

파이프가 있으면 title/description에 대해서 유효성 체크를 하고 보낸다는 뜻으로 해석하면 되는데 유효성에 맞지 않으면 실패하게 되어서 Error 를 띄운다고 생각하면 됩니다. 

파이프 코드는 다음과 같습니다.

```ts
import { ArgumentMetadata, BadRequestException, PipeTransform } from "@nestjs/common";
import { BoardStatus } from "../board-status.enum";

export class BoardStatusValidationPipe implements PipeTransform {
    readonly StatusOptions =[
        BoardStatus.PRIVATE,
        BoardStatus.PUBLIC
    ]
    
    transform(value: any) {
        value = value.toUpperCase()

        if(!this.isStatusValid(value)){
            throw new BadRequestException(`${value} isn't in the status options`)
        }

        return value;
    }

    private isStatusValid(status:any){
        const index = this.StatusOptions.indexOf(status)
        return index !== -1;
    }
}
```
파이프를 넣게 되면 transform 이라는 함수가 자동으로 실행이 되게 되고 그러면서 밑에 있는 함수를 통해 검증한다고 생각하면 됩니다.

이런식으로 검증하는 코드의 파이프를 Data validation파이프라고 합니다. **입력 데이터를 평가하고 유효한 경우 변경되지 않은 상태로 전달하면 됩니다**.

그렇지 않으면 데이터가 올바르지 않을떄 예외를 발생시킨다고 생각하면 됩니다.

다른 종류의 파이프로는 Data transformation 파이프가 있는데

**입력 데이터를 원하는 형식으로 변환**해줍니다. 그렇지 않으면 데이터가 올바르지 않을떄 예외를 발생시킨다고 생각하면 될거 같습니다. 

이러한 종류의 경우 모든 파이프는 위의 두 경우에서 **라우트 핸들러가 처리하는 인수**에서 작동하며 파이프 메소드를 바로 직전에 작동해서 메소드로 향하는 인수에 변환할 것이 있으면

변환하고 유효성 체크를 위해서도 호출된다고 생각하면됩니다.

### 파이프 사용법
총 세가지 방법의 사용법이 존재합니다.

1. Handler-level Pipes
  
핸들러 레벨에서 `@UserPipes()` 데코레이터를 이용해서 사용할 수 있으며, 이 파이프는 **모든 파라미터**에 적용이 됩니다.(title, description)

2. Parameter-level Pipes

특정한 파라미터에게만 적용이 되는 파이프입니다.
```ts
  @Post()
    createBoard(
      @Body('title', ParameterPipe) title
      @Body('description') description
    ) : Board{
          return this.boardsService.createBoard(createBoardDto)
    }
```
위와 같은 파이프의 경우 title에만 파이프가 작동합니다. 

3. Global-level Pipes

애플리케이션 레벨의 파이프입니다. 클라이언트에서 들어오는 모든 요청에 적용이 됩니다. 가장 상단 영역인 `main.ts`에 들어가면 됩니다.
```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(GlobalPipes);
  await app.listen(3000);
}
bootstrap();
```
이런식으로 사용할 경우 클라이언트에서 들어오는 모든 요청에 적용이 됩니다. 

### Built-in Pipes
Nest Js에서는 사용자의 편의를 위해서 제공하는 6가지 파이프가 있습니다.
- ValidationPipe
- ParseIntPipe
- ParseBollPipe
- ParseArrayPipe
- ParseUUIDPipe
- DefaultValuePipe

이 중 한가지를 맛보기로 살펴보자면

```ts
@Get('id')
findONe(@Param('id',parseIntPipe) id:number){
  return
}
```
이런 코드가 있는 가운데 파라미터 값으로 `abc` 라는 문자 값이 온다고 가정해봅시다. 그렇다면 에러를 발생시키고

우리는 이런 파이프등을 통해서 유효성을 체크할 수 있게 됩니다.

### Pipe를 통한 유효성 체크해보기
게시물을 생성할때 파이프를 써보는 예제입니다.
```ts
import {IsNotEmpty} from "class-validator"

export class CreateBoardDto {
    @IsNotEmpty()
    title: string;

    @IsNotEmpty()
    description: string;
}
```
이런식으로 DTO객체를 만듭니다. 다음으로 실 적용은 위에서 보여드렸던

```ts
@Post()
    @UsePipes(ValidationPipe)
    createBoard(
        @Body() createBoardDto: CreateBoardDto
        
        ):Board{
            return this.boardsService.createBoard(createBoardDto)
    }
```
코드에서 적용되는 예제인데 코드를 해석하자면 만약 @Body의 내용이 createBoardDto 매개변수로 바인딩하고, 이때 ValidationPipe는 다음과 같은 작업을 수행합니다.

1. 요청 본문의 데이터를 `CreateBoardDto`에 선언된 타입으로 변환합니다. 예를들어 `CreateBoardDto`에서 문자열로 선언된 필드는 문자열로 변환됩니다.
2. 유효성 검사: `CreateBoardDto`에 정의된 유효성 검사규칙(class-validator라이브러리를 통해서 선언될 될 수 있음)을 사용하여 요청 데이터를 검사합니다.
예를들어 필수 필드의 존재 여부 문자열 길이, 숫자 범위 등이 유효성 검사 대상이 될 수 있습니다.
3. 오류처리: 유효성 검사에서 오류가 발견되면, NestJS는 클라이언트에게 오류 응답을 보냅니다. 이 응답은 일반적으로 유효하지 않은 필드와 관련된 정보를 포함합니다. 

### 에러 표출해보기 
에러 표출을 위해서는 예외 인스턴스(미리 만들어져 있음)를 사용하면 됩니다.
```ts
getBoardById(id:string):Board{
        const found = this.boards.find(board => board.id ===id);

        if(!found){
            throw new NotFoundException(`Can't find Board with id ${id}`)
        } 
        return found
    }
```
이런식으로 에러를 던져주면 잘 작동하는 모습을 볼 수 있습니다. 

### public/private/readonly
public: 접근가능, 변경가능
private: 내부만 접근가능, 내부에서 변경가능
readonly: 접근 가능, 변경불가

### ORM이란
객체와 관계형 데이터베이스의 데이터를 자동으로 변형 . 및연결하는 작업, ORM을 이용한 개발은 객체와 데이터베이스의 변형에 유연하게 사용이 가능하며

객체지향 프로그래밍은 클래스를 사용하고, 관계형 데이터베이스는 테이블을 사용합니다. 이걸 매핑하는 것을 ORM 이라고 할 수 있습니다.

### TypeORM 특징과 이점
모델을 기반으로 Db table을 자동으로 생성합니다. 데이터베이스에서 개체를 쉽게 삽입, 업데이트 및 삭제가 가능합니다.

테이블간의 매핑(일대일, 일대다 . 및 다대다)를 만듭니다.

간단한 CLI 명령을 제공합니다. TypeORM은 간단한 코딩으로 ORM 프레임워크를 사용하기 쉽습니다. TypeORM은 다른 모듈과 쉽게 통합이 가능합니다. 

설치방법은 다음과 같습니다.

```sh
@nestjs/typeorm
typeorm
pg
```

### 게시물을 위한 Entity 생성하기
왜 Entitiy를 사용해야 할까요?

원래 ORM 없이 데이터베이스 테이블을 생성할떄를 먼저보면 다음과 같습니다.

```ts
CREATE TABLE board(
  id INTEGER AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description VARCHAR(255) NOT NULL
)
```
이런식으로 테이블을 생성해야 하는데 하지만 TypeORM을 **사용할떄는 데이터 베이스 테이블로 변환되는 클래스**이기 떄문에 위처럼 사용하지 않고 클래스 생성후 그안에 컬럼을 정의하면 됩니다. 

### 엔티티 관련 데코레이터들
- @Entity()

  Entity 데코레이터 클래스는 Board 클래스가 엔티티임을 나타내는데 사용됩니다.
- @PrimaryGeneratedColumn()

  PrimaryGeneratedColumn() 데코레이터 클래스는 id열이 Board엔터티의 기본키열임을 나타내는데 사용됩니다.
- @Column()

  Column() 데코레이터 클래스는 Board 엔터티의 title 및 description과 같은 다른 열을 나타내는데 사용됩니다.

예제코드는 다음과 같습니다.

```ts
import { BaseEntity, Column, Entity, PrimaryGeneratedColumn } from "typeorm";
import { BoardStatus } from "./board-status.enum";


@Entity()
export class Board extends BaseEntity {
    @PrimaryGeneratedColumn()
    id:number

    @Column()
    title: string;
    
    @Column()
    description: string;
    
    @Column()
    status: BoardStatus;
}
```
id 를 Primary Key로 선언하고 column등을 여러개 선언하는 모습입니다. 

### TypeORM 3.0 Repository 사용하기
Repository는 엔터티 개체와 함께 작동하며 **엔티티 찾기, 삽입, 업데이트, 삭제** 등을 처리합니다.

우리는 엔티티를 생성했고, 찾거나 삽입하거나 업데이트, 삭제등을 처리하는 부분을 일컫습니다. 

Nest js의 흐름을 다시한번 생각해봅시다. client -> server로 가면 controller 로 가고 서비스로 가고 한번 더 Repository로 간다는 것입니다.

이렇게 하는 이유는 service는 데이터 정제를 담당하고 Repository는 DB와 데이터를 주고받는 용도로 사용하기 위해서 입니다. 

또한 이제  3.0 버전에서는 사용하는 방법이 달라졌는데 제가 사용한 예제를 들어드리겠습니다. 

```ts
import { DataSource, Repository } from "typeorm";
import { Board } from "./board.entity";
import { Injectable } from "@nestjs/common";
import { InjectRepository } from "@nestjs/typeorm";
import { CreateBoardDto } from "./dto/create-board.dto";
import { BoardStatus } from "./board-status.enum";

@Injectable()
export class BoardRepository extends Repository<Board>{
    constructor(private dataSource:DataSource){
        super(Board, dataSource.createEntityManager())
    }

    async createBoard(createBoardDto:CreateBoardDto):Promise<Board>{
        const {title, description} = createBoardDto;

        const board = this.create({
            title,
            description,
            status:BoardStatus.PUBLIC
        })
        await this.save(board);
        return board
    }
}
```
3.0 버전 이전에는 `@EntitiyRepository(Board)`로 선언하여 repository 를 사용하였으나 이제는 

@Injectable()로 선언하고 repository를 연결합니다.

이제 여기서 중요한것은 `dataSource` 부분 입니다. dataSource는 우리들이 database와 상호작용하기에 꼭 필요한 한가지 방법입니다. 

TypeORM's DataSource 는 데이터베이스와의 연결을 유지하고, 우리들이 데이터베이스를 사용할 수 있도록 해줍니다. 

원래는 커넥션을 연결하고 connection Pool을 유지하려면 반드시 `initialize method`를 호출해야 합니다.(DataSource instance에서)

연결을 끊기 위해서는 destroy 메서드가 호출되어야 끊깁니다. 

일반적으로는 `initialize` 메서드는 app이 bootstrap될때 실행됩니다. 그리고 `destroy`메서드는 데이터베이스 작업이 끝났을때 실행됩니다. 

실제로는, 우리들이 계속에서 백엔드 서버를 사용하고 있다면 destroy 메서드를 실행할 필요가 없습니다. 

`export class BoardRepository extends Repository<Board>` 다음으로 이 코드에서 Repository<Board>는

Board에 대한 entity만 다룬다. 즉 Board table을 다룬다! 라고 생각하면 됩니다.

우리는 entity에 대해서 이야기 할떄 그 table의 컬럼속성 들을 말한다고 했기 때문에 그 부분을 신경써서 처리하면 됩니다. 

이런식으로 데이터들을 연결하는 모습들을 보여주고 있습니다. 


## .env nest.js에서 사용하기 
nest.js에서는 .env를 사용하는 방식이 조금 특이합니다.

사실 이 부분은 좀 더 딥하게 들어가 봐야 하는데 현재 추측으로는 빌드되면서 환경변수를 읽어오지 못하는게 아닌가?

그렇기 떄문에 읽을 수 없어서 문제가 발생한다고 추측! 하고 있습니다. 혹시 아시는 분들은 댓글이나, 연락을 주신다면 

열심히 수정해보겠습니다. 

우선 NestJS에서는 기본적으로 .env 를 이용하고 있습니다.

따라서 NestJs에서 제시하고 있는 방법만 따라가면 되는데요. 또 여기서 그치는게 아니라 nestJs에서 맥북뿐만아니라 윈도우에서도 쓰일 수 있다는 사실을 예상하여 

짜봅시다. 그래서 우리는 cross-env를 설치하고