# 토스뱅크 X 메리츠증권 채권

채권 매매 퍼널 퍼블리싱 소스코드 저장소입니다.

## 개발 가이드

- 각 화면의 UI를 렌더링하는 코드는 모두 JS로 작성 후 난독화되어있기 떄문에 직접적으로 수정할 수 없습니다. UI 수정은 전혀 신경쓰실 필요가 없지만, 혹시라도 이 부분과 관련해 도움이 필요하다면 슬랙에서 `@최근원`을 찾아주세요.
- 각 화면에 해당하는 HTML 파일을 열어보면 `<script></script>`에서 `window.toss` 객체에 프로퍼티를 정의한 부분을 확인할 수 있습니다. `window.toss`에 정의한 프로퍼티는 화면에 필요한 정보를 표시하는 데 사용되거나 버튼 클릭 시 실행할 콜백 함수 등으로 사용됩니다.
- `<script></script>` 내부의 각 로직에 대한 설명을 주석으로 남겨놓았습니다.
- 실제로 직접 수정해주셔야 할 부분은 `TODO:` 마크를 추가해두었습니다.
- `Promise` Polyfill 스크립트를 추가해두었습니다. `async ~ await` 문법은 사용자 브라우저 환경에 따라 지원되지 않을 수 있으니, 비동기 로직이 필요한 경우 ES6의 `Promise` API를 이용해주세요.

## 디렉토리 구조 (예시)

```
shared/
  js/
    toss-app-bridge.js            # 토스 앱과 통신하기 위한 브릿지 함수가 정의되어있습니다. 자세한 내용은 'tossAppBridge' 섹션을 확인해주세요.
    toss-utils.js                 # 금액 포맷팅(`commaizeNumber`) 등 유틸리티 함수가 정의되어있습니다. 자세한 내용은 'tossUtils' 섹션을 확인해주세요.
  css/
    style.css                     # 공통 CSS 변수를 정의합니다.
    tds.css                       # TDS(Toss Design System) 관련 스타일 코드를 정의합니다.
구매하기/                           # 구매하기 퍼널 HTML & JS 소스를 모아두었습니다.
  js/                             # 각 화면의 UI를 렌더링하는 JS 코드가 들어있습니다.
  1-ProductInfo.html
  2-Terms.html
  3-Amount.html
  4-BuyingInfo.html
  5-UnsuitableInvestment.html
  6-ProprietyAnalysisReport.html
  7-Password.html
  8-Complete.html
  9-Failed.html
판매하기/                           # 판매하기 퍼널 HTML & JS 소스를 모아두었습니다.
  js/                             # 각 화면의 UI를 렌더링하는 JS 코드가 들어있습니다.
  1-Detail.html
  2-Password.html
  3-Complete.html
  4-Failed.html
  5-Interest.html
```

## `tossAppBridge`

웹뷰 환경에서 App과 Web이 통신하기 위한 중간자 역할을 하는 도구를 **앱 브릿지**라고 합니다. 대표적으로 웹뷰 닫기, 토스트 띄우기, 앱의 뒤로 가기 버튼 제어하기 등이 있습니다.

`window` 객체에 정의된 `tossAppBridge` 프로퍼티를 통해 앱 브릿지 함수를 호출할 수 있도록 정의해두었습니다.

### `closeWebView`

현재 띄워진 웹뷰를 닫습니다.

```js
// 현재 띄워진 웹뷰를 닫습니다.
tossAppBridge.closeWebView();

// `nextUrl`을 통해 웹뷰가 닫힌 후 이동할 supertoss 스킴을 지정할 수 있습니다.
// 현재 띄워진 웹뷰를 닫고 토스뱅크 홈으로 이동합니다.
tossAppBridge.closeWebView({
  nextUrl: 'supertoss://lab?url=https%3A%2F%2Fservice.tossbank.com%2Fhome',
  pendingDismiss: true
});
```

### `sendPDF`

base64 인코딩된 PDF 파일을 받아 PDF 리더를 실행합니다.

- data: base64 인코딩된 PDF 파일
- filename: 파일 이름
- ctaTitle: 하단 CTA 버튼 타이틀

```js
tossAppBridge.sendPDF({
  data: base64EncodedPDF,
  filename: '메리츠증권',
  ctaTitle: '확인'
});
```

### `addAccessoryButton`

네이티브 웹뷰의 우상단 부분에 아이콘 또는 텍스트 버튼을 추가합니다.

```js
tossAppBridge.addAccessoryButton({
  type: 'text',
  title: '버튼 제목',
  callback: () => {
    alert('버튼을 클릭했어요');
  }
});
```

### `removeAccessoryButton`

네이티브 웹뷰의 우상단 부분에 `addAccessoryButton`으로 추가된 아이콘 또는 텍스트 버튼을 삭제합니다.

```js
removeAccessoryButton();
```

### `setBackPressHandler`

웹뷰 좌측 상단의 뒤로 가기 키가 눌릴 경우, `callback`으로 전달된 함수를 실행합니다. `callback` 의 반환값이 `true`이면 뒤로 가기가 방지되고, `false` 또는 `undefined`이면 콜백 실행 후 뒤로 가기가 수행됩니다.

```js
tossAppBridge.setBackPressHandler({
  callback: () => {
    alert('뒤로 가기 클릭');

    return true;
  },
});
```

### `unsetBackPressHandler`

`setBackPressHandler`로 등록된 `callback` 핸들러를 제거합니다.

```js
tossAppBridge.unsetBackPressHandler();
```

### `showToast`

토스트를 띄웁니다.

```js
tossAppBridge.showToast('나는 토스트입니다.');
```

## `tossUtils`

유틸리티 함수를 `window` 객체에 정의된 `tossUtils` 프로퍼티를 통해 접근할 수 있습니다.

### `commaizeNumber`

입력받은 숫자에 콤마를 적절히 삽입합니다.

```js
tossUtils.commaizeNumber(1000000); // 1,000,000
```
