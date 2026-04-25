
## nanoclow code분석 src/index.ts

### 1. `import './config.js'` 의미
- src dir 아래에 config.js가 있음

src/index.ts에서 import로 config.js에 선언된 ASSISTANT_NAME을 가져옴

``` typescript
import {
  ASSISTANT_NAME,
    ...
} from './config.js';
```

src/config.ts에는 ASSISTANT_NAME이 있음

``` typescript
export const ASSISTANT_NAME =
  process.env.ASSISTANT_NAME || envConfig.ASSISTANT_NAME || 'Andy';
```

### 2. `import './channels/index.js';` 의미
- "채널 관련 기능들이 정의된 index.js를 이 위치에서 곧바로 실행해 줘!"라는 명령으로 이해
- 안에 있는 모든 기능이 실행
-  index.js은 모듈의 진입점


### 3. import 괄호 사이의 의미

``` typescript
import {
  ContainerOutput,              // export interface
  runContainerAgent,
  writeGroupsSnapshot,
  writeTasksSnapshot,
} from './container-runner.js';
```
- src/containr-runner.ts에서
- 1번째는 interface의 노출
- 2~4번째는 함수의 노출

### 4. function 객체를 받아서 연결하기

```
  setProcessMessagesFn(fn: (groupJid: string) => Promise<boolean>): void {
    this.processMessagesFn = fn;
  }
```
- class가 가지는 setProcessMessagesFn메소드는 리턴이 없고, fn을 인자로 받는다.
  - groupJid를 받고 Promoise를 리턴하는 fn


### 5. Record > sessions 변수 선언

``` typescript
let sessions: Record<string, string> = {};
```
- Record는 내장 유틸리티 타입(Built-in Utility Types)임

``` typescript
sessions = {
    "user_01": "session_id_aaaa",
    "auth_token": "xyz123",
    "device_id": "iphone_15"
};
```

### 6. Record > nav 객체 변수 선언 - 3가지 key 제한

``` typescript
type Page = 'home' | 'about' | 'contact';

// 키는 반드시 'home', 'about', 'contact' 중 하나여야 함
const nav: Record<Page, string> = {
home: '/index',
      about: '/intro',
      contact: '/call '
};
```

### 7. 객체 리터럴 구현

- 인터페이스가 정의되어 있다면, 함수를 호출할 때, 그 구조에 맞는 객체를 즉석에서 만들어 넘길 수 있음.h:w!


### 8. `import { OneCLI } from '@onecli-sh/sdk';`

- 외부 라이브러리(패키지)에서 OneCLI라는 특정 클래스나 도구를 가져오겠다는 의미
- 전체를 다 가져오는게 아니라 OneCLI만 가져옴

``` typescript
import * as OneCLISDK from '@onecli-sh/sdk';
...
const cli = new OneCLISDK.OneCLI();
```

### 9. `import { Client, GatewayIntentBits, TextChannel } from 'discord.js';`
- discord.js 라이브러리에서 bot을 만드는데 핵심이 되는 3가지 도구를 가져옴

  - Client : 봇의 몸체
  - GatewayIntentBits : 권한설정
  - TextChannel : 채팅방 타입

