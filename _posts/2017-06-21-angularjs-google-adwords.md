--- 
layout: post 
title: "Google Adwords Conversion(전환)설정 Angularjs" 
date: 2017-06-21 00:00  
author: teamsmiley 
tags: [entity]
image: /files/covers/blog.jpg
category: {entity}
---

# Google Adwords의 전환 측정 AngularJs 로 사용해보기 

## Google Analytic에 가입한다. 
https://adwords.google.com

## 메뉴 확인 

* 도구 >> 전환액션 클릭 >> 전환 + 버튼을 눌러서 전환을 추가한다. >> 웹사이트 >> 이름을 적어넣고 나머지 옵션은 기본값으로 일단 저장한다.

* 만들어진 전환액션을 클릭해서 들어가보면  태그 설정 부분이 있다 이 코드를 가져오자.  전환을 하나 더 만들면 코드가 하나가 더 생긴다. 

```html
<!-- Google Code for LogIn Conversion Page -->
<script type="text/javascript">
/* <![CDATA[ */
var google_conversion_id = 1071463456; //<===== 
var google_conversion_language = "en_US";
var google_conversion_format = "1";
var google_conversion_color = "ffffff";
var google_conversion_label = "cX4RCOGESxCR-PT-lw"; //<=====
var google_remarketing_only = false;
/* ]]> */
</script>
<script type="text/javascript" src="//www.googleadservices.com/pagead/conversion.js">
</script>
<noscript>
<div style="display:inline;">
<img height="1" width="1" style="border-style:none;" alt="" src="//www.googleadservices.com/pagead/conversion/1071463441/?label=cX4RCOGESxCR-PT-Aw&amp;guid=ON&amp;script=0"/>
</div>
</noscript>
```

## 앵귤러에 적용 
### index.html수정 
다음 코드를 추가한다. 
```html
<!-- Google Adwords Conversion Script -->
<script type="text/javascript" src="//www.googleadservices.com/pagead/conversion_async.js"></script>
```


### GoogleAdWordsService  서비스를 만들자. 

```js
angular.module('adwordsApp') //<==3
    .factory('GoogleAdWordsService', function ($window) {
        // Conversion labels 
        var google_conversion_label = {
            'register-customer': "12abCDef3gH5Klm6789", //<==2
            'book-order': "9876mlK5Hg3feDCba21"
        };
       // Basic settings for AdWords Conversion
        var googleTrackConversion = function (conversion_label) {
            $window.google_trackConversion({
                google_conversion_id: 0987612345, //<==1
                google_conversion_language: "en",
                google_conversion_format: "3",
                google_conversion_color: "ffffff",
                google_conversion_label:                  google_conversion_label[conversion_label],
                google_conversion_value: 0,
                google_remarketing_only: false
            });
        };
 return {
            sendRegisterCustomerConversion: function () {
                // Trigger register-customer conversion 
                googleTrackConversion('register-customer'); //<==4
            },
            sendBookOrderConversion: function () {
                  // Trigger book-order conversion 
                googleTrackConversion('book-order');
            }
        };
    });
```
이 코드를 적당히 수정하면된다. ..

//<==3 : 일단 앱이름을 사용하는거랑 맞게 한다. 
//<==2 : 웹사이트에서 받은 코드를 잘 보면 저 비슷한 숫자가 보인다. 여러개의 전환을 만들면 코드가 다 다르다  이걸 복사해다 넣으면 된다.  여러개를 만들면 코드가 여러개가 된다. 샘플은 코드가 2개인 걸 가정햇다. 전환이 여러개로  추가되면  //<==2번에 추가해주고 //<==4번에도 맞는 이름을 넣어줘야한다. 

### 필요한 컨트롤러에 코드를 추가하자. 
나는 로그인 한 유저를 확인하고 싶엇다.  login controller에 logIn 함수를 수정하자. 

```js
app.controller('loginController', [..,'GoogleAdWordsService', function (..., GoogleAdWordsService) {

$scope.login = function () {

        GoogleAdWordsService.logInConversion(); // 이 코드 추가 
        ....
}
```

두군데를 추가해주면 된다. 

컨트롤러가 눌릴때마다 정보를 구글로 보낸다.

참고 <https://medium.com/@zainzafar/google-adwords-conversion-tracking-using-angularjs-f2047b3331f4>


