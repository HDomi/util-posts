
### [코드 하이라이트 테스트용 포스트]

```Java
yarn add -D @babel/core @babel/cli @babel/preset-env
```

- <code>@babel/core</code> : babel이 트랜스파일 할 수 있게 해준다.
- <code>@babel/cli</code> : CLI에서 babel을 실행할 수 있게 해준다.
- <code>@babel/preset-env</code> : <code>.babelrc</code>의 <code>targets</code> 옵션에 선언된 환경에 대응할 수 있는 plugin들을 모아놓은 preset 라이브러리. 가장 많이 쓰인다.

##### 📃 package.json

```json
{
  "devDependencies": {
    "@babel/cli": "^7.17.3",			// NEW
    "@babel/core": "^7.17.5",			// NEW
    "@babel/preset-env": "^7.16.11",	// NEW
    "jest": "^27.5.1"
  }
}
```
#### Babel 설정
프로젝트 루트에 <code>.babelrc</code> 파일 생성

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {        
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1"
        }
    ]
  ]
}
```
- <code>lib</code> : 해당 경로의 폴더 안에 있는 JS파일을 트랜스파일링
- <code>-w</code> : 파일들의 변경 감지 (--watch 축약형)
- <code>-d dist/js</code> : 트랜스파일링된 파일 저장 경로 (--out-dir 축약형)

#### Babel 실행
테스트 해보기 위해 graph.js에 <code>arrow function</code>과 <code>Set</code>을 사용 후 빌드해보았다.
그러나 아래와 같이 상단에 <code>"use strict";</code>만 추가되고 변한게 없다.
##### 📃 lib/graph.js

```javascript
function graph(num) {
  const setInstance = new Set([1, 2, 3, 4, 5]);
  const hasNum = () => setInstance.has(num)
  return hasNum
}
module.exports = graph
```
##### 📃 dist/js/graph.js

```javascript
"use strict";

function graph(num) {
  const setInstance = new Set([1, 2, 3, 4, 5]);

  const hasNum = () => setInstance.has(num);

  return hasNum;
}

module.exports = graph;
```
<code>"ie": "11"</code>을 <code>targets</code>에 추가하고 빌드해봤더니 <code>const</code>와 <code>arrow function</code>이 바뀐 것을 볼 수 있다.

##### 📃 dist/js/graph.js

```javascript
"use strict";

function graph(num) {
  var setInstance = new Set([1, 2, 3, 4, 5]);

  var hasNum = function hasNum() {
    return setInstance.has(num);
  };

  return hasNum;
}

module.exports = graph;
```

그런데 <code>Set</code>은 여전하다. <code>Set</code>같은 ES6의 객체들은 ES5로 구문 변환만 해서는 안되고 해당 객체들을 정의하는 polyfill을 추가해줘야 한다.
- - -
  ```javascript
  //before transpiling
  Promise.resolve().finally();

  //after transpiling
  require("core-js/modules/es.promise.finally");

  Promise.resolve().finally();
  ```
- <code>**entry**</code>
: 폴리필 모듈을 전역 스코프에 직접 삽입하면 타깃 환경에 필요한 폴리필만 전역 스코프에 추가된다.
- <code>**false**</code>
: default 값. 사용 안함.

실제 사용한 폴리필만 삽입되는 <code>usage</code>옵션이 있으면, 전역으로 타깃 환경에 필요한 폴리필을 미리 추가하는 <code>entry</code>를 사용할 일은 없지않나 싶었다.
<code>entry</code>의 사용해도 되고 사용하면 얻을 수 있는 이점이 무엇인지 궁금했다.
하지만 이쪽을 계속 파자니 한도 끝도 없어서 우선 나는 여기까지만 알아보고 <code>usage</code>를 사용하기로 결정!💃

#### corejs 옵션 설정
>corejs 옵션은 useBuiltIns 옵션과 함께 사용해야 합니다. 해당 옵션은 삽입되는 core-js 모듈의 버전을 설정합니다. default 값은 2이고, version 2는 업데이트가 중단되었기 때문에 현재는 version 3를 사용해야 합니다.
>
> <cite>[kakao Tech](https://tech.kakao.com/2020/12/01/frontend-growth-02/)</cite>

꼼꼼하게 정리된 kakao Tech 글을 보면서 차근차근 따라했다.
테스트를 위해 넣었던 ie는 타겟옵션에서 제거했다.

##### 📃 .babelrc

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1"
        },
        "useBuiltIns": "usage",
        "corejs": "3"  // NEW
      }
    ]
  ]
}
```

```javascript
"use strict";

require("core-js/modules/web.dom-collections.iterator.js");

function graph(num) {
  const setInstance = new Set([1, 2, 3, 4, 5]);

  const hasNum = () => setInstance.has(num);

  return hasNum;
}

module.exports = graph;
```