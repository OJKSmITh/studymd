# pm2의 fork와 cluster
pm2를 생각하면 주로 우리는 무중단 서비스를 위한 라이브러리라고 생각하는 사람이 많을것 같다.

이번에 프로젝트를 맡으면서 pm2의 클러스터를 보게되면서 클러스터를 왜 사용할까? 저건 뭘까? 하다가 작업중

local에서는 소켓 통신이 잘되는데 실 서버에서 소켓 통신이 작업하지 않는 이슈를 발견 하게 되었다.(**도대체 나한테 왜!!!! 라고 처음에 소리 질렀던것 같습니다.**)

그래서 pm2 log를 살펴봤을때 다음과 같은 특이점을 발견하게 되었습니다. 

<img src="./img폴더/pm2 log.png"/>

이렇듯 0번 프로세스에서 요청은 받아놓고 실제 처리는 1번 프로세스에서 하는 <span style="color:red">pm2의 클러스터 모드를 사용하고 있었기 때문입니다.</span>

처음에는 이 문제를 보고 뭐가 문젤까... 같은 프로세스 아냐? 라고 생각했지만 뒤에서 다른 직장 책임님과 팀장님께서 바로 답변을 주셨습니다(최고!!)

이유는 클러스터를 쓸 경우 스레드를 같이 쓰는게 아니라 **각각의 프로세스 이므로 <span style="color:red">메모리 공유가 불가능</span> 하다는 것이었습니다**!

따라서 이를 해결하기 위해서 redis가 등장하게 되는데... 이렇게 해서 문제를 해결하게 됐던건 나중에 다시 정리를 하겠습니다.

근데 이 문제를 해결하면서 들었던 궁금증은 pm2로 어떻게 저렇게 만들까? 그리고 클러스터는 무엇일까? 라는 궁금점이였던것 같습니다. 

그러면서 두가지 pm2에는 두개의 기능이 있는것을 알게 되었습니다.

## cluster
pm2에는 cluster가 있는데 이와 같은 기능이 나오게 된 배경은 다음과 같습니다. 

Node는 싱글스레드 언어이고 Non blocking 을 사용합니다. 즉 하나의 Cpu이고 하나의 프로세스에서 동작한다는 의미인데요

즉 우리가 막 8개 20개 100개의 코어가 있다고 하더라도 node로 쓸 수 있는 코어는 하나밖에 없다는 의미입니다. 

즉 이러한 문제를 해결하기 위해서 등장하게 된것이 pm2의 클러스터 기능입니다.

cluster 의 경우 네트워크 애플리케이션을 위한 기능인데,  하나의 포트에 독립된 N개의 프로세스가 실행되도록 합니다. 근데 하나의 포트에 어떻게 각각 독립된 프로세스들이 동작하게 될까요?

이떄 pm2 의 `Master Process`가 그것을 가능하게 합니다. 

pm2 는 scheduler 라는 것이 존재합니다. 이 스케쥴러가 요청을 모두 수신하고 워커 프로세스로 작업을 전달하고 외부에서 봤을때 하나의 시스템처럼 보이도록 한다고 하였는데

이 마스터 프로세서가 스케줄러의 역활을 수행하며, 주어진 포트에서 수신 대기하고 수신된 내용을 워커 프로세스에게 전달 처리하게 하여 마스터와 워커 프로세스 사이의 작업의 전달은 메세지를 이용하여 통신하게 한다는것입니다. 

하지만 이 **프로세스를 사용한다는 것** 때문에 스레드가 아니여서 메모리 공유가 불가능하다는 단점이 나오지만 node의 가장 큰 단점인 cpu를 단 하나만 사용한다는 것을 상쇄시켜 주기 때문에

저는 좋다고 생각합니다. 

## 클러스터 모드의 준비시간 문제
우리가 app을 시작할때 dbconnection, redis server connection등 다양한 서버 접속이 필요하다고 생각해봅시다. 

근데 워커 프로세스가 `spawn`이 되면 `ready` 이벤트를 마스터 프로세스로 보내게 되고, 이를 수신한 마스터 프로세스는 `old` 영역에 있는 프로세스에 `sigint` 이벤트를 보낼것입니다.

sigint 이벤트를 수신한 `old` 영역의 프로세스는 프로세스를 종료하거나  1600ms가 지나서 `sigkill` 이벤트를 수신하게 될것입니다. 하지만 새로운 워커 프로세스는 아직 구동이 완료되지 않았으므로 서비스에 장애가 발생하게 됩니다.

따라서 이와 같은 문제 해결을 위해 워커 프로레스에 자신이 정상 작동이 가능한 상태가 되었을떄 `ready`이벤트를 보내도록 설정합니다. 

자 그러면 이 이벤트를 수신할 pm2에도 워커 프로세스의 ready 신호를 받도록 해야 하기 때문에 설정을 추가해야 되는데 간단한 소스코드를 통해 확인해 봅시다.

```js
app.js
const express = require("express");
const app = express()
const port = process.env.port || 8000
app.get('/',(req,res)=>{res.send('Hello world')})

const server = require("http").createServer(app)
server.listen(port, async ()=>{
    console.log('Express server listening on port' + server.address().port)
    process.send('ready')
})
```

```js
ecosystem.config.js
module.exports = {
    apps: [{
        name: 'app',
        script: './app.js',
        instances: 0,
        exec_mode: 'cluster',
        wait_ready: true,
        listen_timeout:10000
    }]
}
```
여기서 `app.js` 에서 확인해야 할 부분은 `process.send('ready')`입니다. 해당 코드는 PM2로 ready 메시지를 직접 보내겠다는 의미입니다. 

이 코드는 정상적으로 작동이 가능하다고 판단되는 부분에 작성하면 되겠습니다.

다음으로 `ecosystem.config.js`를 보게 되면 `wait_ready`, `listen_timeout`이라는 두 부분이 보일겁니다.

`wait_ready`는 pm2가 워커 프로세스에서 보내는 ready 메시지를 기다리겠다는 뜻이고 `listen_timeout`은 메시지를 수신 대기하는 최대 시간을 가리킵니다.

설정에서는 10000ms 지정하고 있는데 기본 3000보다 많이 늘린 것입니다. 