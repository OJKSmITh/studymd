# Nest Js Swagger
이번에 처음 회사 업무를 진행하면서 시간이 많이 걸렸던 업무중의 하나는 바로 API 문서 작업이었다.

API 문서 작업의 경우 swagger로 작성하거나, 혹은 PostMan으로 공유하긴 한다.

하지만 업무 공유상 이라는 목적 하에는 swagger로 작성하는게 프론트 개발자 분들이 보기에 편한다고 판단했고

작업을 진행했으나... `yml`파일을 작성하는것이 생각보다 지켜야할 문법도 많고, 작성하기도 굉장히 까다로웠다.(다른 분이 만든걸 참고한걸 했음에도 불구)

때문에 yml파일 대신에 자동화 하는 방법이 없을까? 라는 고민을 하던중 nest.js 의 swagger를 발견했다. (node.js에서는 가능하지만, 타입 지정이 안되기 때문에 더 복잡할 것으로 추론)

먼저 깔아줘야 할 라이브러리들은 다음과 같다.

```sh
npm install @nestjs/swagger swagger-ui-express
```
를 설치해 주면 되겠다. 

이렇게 설치를 하게 된후에는 main.ts로 이동한다. 원래 main.ts의 코드는 다음과 같았다.

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  await app.listen(3000);
}
bootstrap();
```
이 코드에 app.listen 하기전에 swagger 관련 모듈을 설치해준다고 생각하면 된다. 이제 설치한 모듈을 다음과 같이 불러오자.

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
  .setTitle('NestLearnProject API')
  .setDescription('Nest Learn을 위한 API 문서입니다.')
  .setVersion('1.0')
  .build()
  const document = SwaggerModule.createDocument(app, config)
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}
bootstrap();
```
이렇게 불러오고 나면 자동으로 이미 설정되어 있는 api들을 형성해 준다. 이때 주의할 점은 자동으로 불러와 주긴 하지만 이빨들이 듬성듬성 빠져있다는 점이다. 

따라서 body와 설명들을 적어주기 위해서는 다음과 같이 코드를 바꿔준다. 

```ts
// boards.controller.ts
import { Body, Controller, Delete, Get, Param, ParseIntPipe, Patch, Post, UsePipes, ValidationPipe } from '@nestjs/common';
import { BoardsService } from './boards.service';
import { BoardStatus, UpdateBoardStatusDto } from './board-status.enum';
import { CreateBoardDto } from './dto/create-board.dto';
import { BoardStatusValidationPipe } from './pipes/board-status-validation.pipe';
import { Board } from './board.entity';
import { ApiBody, ApiCreatedResponse, ApiOperation, ApiProperty } from '@nestjs/swagger';

@Controller('boards')
export class BoardsController {
    constructor(private boardsService:BoardsService){}

    @Get()
    @ApiOperation({summary: '게시판 전체 조회'})
    @ApiCreatedResponse({description: '모든 게시물 조회', type:[Board]})
    getAllBoard(): Promise<Board[]> {
        return this.boardsService.getAllBoards();
    }

    @Post()
    @ApiOperation({summary: "게시물 작성하기"})
    @ApiCreatedResponse({description: '게시물 작성완료', type:Board})
    @UsePipes(ValidationPipe)
    createBoard(@Body() CreateBoardDto: CreateBoardDto):Promise<Board>{
        return this.boardsService.createBoard(CreateBoardDto)
    }


    @Get("/:id")
    @ApiOperation({summary:"ID를 통해 게시물 조회하기"})
    @ApiCreatedResponse({description:"조회 성공", type:Board} )
    getBoardById(@Param('id') id:number): Promise<Board>{
         
        return this.boardsService.getBoardById(id);
    }
    
    @Delete('/:id')
    @ApiOperation({summary:"ID를 통해 게시물 삭제하기"})
    deleteBoard(@Param('id', ParseIntPipe) id):Promise<void>{
        return this.boardsService.deleteBoard(id)
    }

    @Patch("/:id/status")
    @ApiOperation({summary:"ID를 통해 게시물 업데이트 하기"})
    @ApiCreatedResponse({description:"업데이트 완료",type:Board })
    @ApiBody({
        description: '업데이트 할 상태 정보',
        type: UpdateBoardStatusDto
    })
    updateBoardStatus(
        @Param('id', ParseIntPipe) id: number,
        @Body('status', BoardStatusValidationPipe) status:BoardStatus
    ){
        return this.boardsService.updateBoardStatus(id,status)
    }
}

```
먼저 `@ApiOperation`의 경우는 해당하는 Api에 대한 상세 설명이라고 생각하면 된다. 

다음으로 `@ApiCreatedResponse`에서 description은 해당 게시물의 response값들에 대한 설명을 적어준다고 생각하면 되고, type이 바로, 어떤 데이터들이 실제로 돌아오는지 명시하는 부분이라고 생각하면된다.

마지막으로 `@ApiBody`는 실제 바디들이 어떻게 작성되어야 하는지 적어줘야하며, description은 설명, 그리고 마지막으로 type은 실제 들어가야 하는 값들을 적어주는 부분인데 이떄 주의해야 할점은 

dto를 다음과 같이 만들어서 해줘야 한다는 점이다. 

```ts
import { ApiProperty } from "@nestjs/swagger";

export enum BoardStatus {
    PUBLIC = "PUBLIC",
    PRIVATE = "PRIVATE"
}

export class UpdateBoardStatusDto {
    @ApiProperty({ enum: BoardStatus, example: BoardStatus.PUBLIC })
    status: BoardStatus
}
```
이런식으로 만들어주게 되면 정확히 어떤 타입을 원하는지 찍어주고 실행할 수 있도록 해준다는 것이다.

이렇게 코드를 통해 문서를 동시에 작업하게 되면 작업을 한번에 처리할 수 있다는 것이 또 장점일 것이다.

이때 접속은 localhost:3000/api 로 접속하게 되면 보이게 되고 swagger 문서로 만들고 싶으면 localhost:3000/api-json으로 가서 json 값을 추출하여 

swaggerhub에서 붙여넣으면 된다. 