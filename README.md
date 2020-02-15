# Cache_System
# cache

web이 생성된 후 인터넷엔 많은 변화가 있었다. -> 남이 생성한 문서를 내 컴퓨터에 저장할 수 있는 엄청난 효과가 발생했다.

하지만, Web이 폭발적으로 성장하게 되면서 인류는 속도에 목마르게 된다. 그 노력중 하나가 **cache**이다.

**" 이미 다운로드 받은 파일을 왜 또 다운받아야하지 ?"** 

-> 파일을 cache 했다가, 다시 접속했을 때 cache를 활용해 속도를 단축할 수 있다.

-> 문제점이 발생한다. ( cache를 최신상태로 유지해야한다. )



## Apache Web Server

### chrome Network tab

(크롬 네트워크의 preserve log를 체크하면 기록들이 초기화 되는 것을 막을 수 있다.)

(reload에 hard reload를 통해서 캐시를 지워버리고 다시 다운로드 받을 수도있다.)

(network tab의 200이라는 값은 읽어오는 데 성공했다는 뜻이다. )

(network의 tab에 size에서 캐시로부터 data를 읽어오는지 확인할 수 있다.)

(hrome의 network tab에서 online tab을 설정하면 속도를 강제로 느리게도 바꿀 수 있다.)

### Apache

* 먼저 Apache의 confiture를 통해서 conf file을 열 수 있다. (restart 해주어야지 반영된다.)

  

  **SetEnv no-gzip 1** : gzip을 하는 압축 방식을 하지 않는다.
  **Header set <span style = "color:Red">Cache-control</span> "no-store"** 

  : Header에 Cache를 하지 말 것에 대한 Header를 추가해준다. 이는 HTTP 1.1 표준에 정의되어 있는 지시자이다.

 접속할때마다 개편된 데이터를 가져올 수 있는 장점이 있기도 하다.

###Cache-contorl를 통해서 최신의 값인지 값을 비교할 수 있다.

"no-store" 저장하지 않는 것

"no-cache" 캐쉬가 유효한지를 항상 확인하는 옵션 : 새로운건지 항상 확인하는 것이다.

max-age = 3153600 수명을 설정 ( 1년 ) => <span style = "color:red">치명적인 문제가 있다. 1년동안은 사용자가 신선한 새로운 데이터를 읽어오지 못한다.</span> data를 읽어올때 캐시에 이미 저장이 되어ㅅ있는데 서버에서 변경된것을 인지 못하고, 신선하지 않은 정보를 갖게되는 문제를 야기할수도 있다.

: max-age = 초단위 숫자(sec)



####수정된 파일을 읽어올 때

**request** 

If-Modified-Since : Sat, 15 Feb 2020 05:53:42 GMT

**response**

Last-Modified: Sat, 15 Feb 2020 05:56:28 GMT



최근 수정된 시간이 불일치 하므로, 수정이 필요한 상황이다 따라서 response는 새로 수정된 data를 보내주게 된다. 이때는 새로 수정된 것을 보내주는 것이 정상적으로 잘 처리됐다는 의미로, Status Code: 200을 전송하게 된다.

####수정되지 않은 파일을 읽어올 때

**request** 

If-Modified-Since: Sat, 15 Feb 2020 05:53:42 GMT

**response**

Last-Modified: Sat, 15 Feb 2020 05:53:42 GMT

시간이 일치하므로 따로 수정을 할 필요가 없다. 수정되지 않았다는 의미로 Status Code: 304 을 전송하게 된다. 하지만 이때도 max-age를 설정한 숫자를 초과한다면 header를 통해서 확인 후 header를 전송해주게 된다. 따라서 아주 소량의 데이터는 주고받는다. 하지만 실질적인 data는 주고 받지 않으므로 속도가 오래걸리지는 않는다.

 

우리는 웹사이트에 처음 접속했을 때 파일을 다운받게 된다. 그 때 서버는 우리에게 last-modifed라는 매우 유용한 정보를 제공한다. 그리고 이것을 기준으로 해서 추후에도 최신인지 아닌지를 감지할 수 있다.

### Etag : Etag를 통해서도 비교를 통해 최신 파일인지 아닌지를 검사할 수 있다.

**request** 

ETag: "45d-59e883666c940"

**response**

If-None-Match: "45d-59e883666c940"



<span style = "color:red">결론적으로 얘기하면 맨처음 파일을 받을때는 순수하게 파일을 요청에 따라 내려받게 된다. 하지만 다음에 파일을 요청하게 될땐 If-None-Match의 Etag와 If-Modified-Since인 최종 수정시간 두가지를 통해서 서버에 파일이 변경됐는지 확인을 요청하게 되고 둘 중에 하나라도 변경된게 있을 경우 파일을 새로 보내주게 되는 것이다.</span>

캐시 정책에 대한 유용한 링크:

https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching

![img](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/images/http-cache-decision-tree.png)

* 파일을 받았는데 또 써야하나? (reusable)
* validate - 매번 유효성을 검사해야되나? ( no-cache를 하면 계속 물어본다.)
* intermidiate : 중계자, 중간 / 캐시를 하다보면 캐싱서버라는 것을 동작하게 된다. 
* ( 예를 들어 소비자가 미국에 있으면 속도가 매우 느리다. 그러면 사용자와 가까운 곳에 캐싱서버라는 것을 둘 수 있다. 웹서버의 내용을 캐싱서버에 캐싱해두면 사용자는 캐싱서버에 접속하는 것을 통해서 데이터를 고속으로 가져갈 수 있게 된다.  이렇게 되면 내 컴퓨터에 있는 캐시가 있고, 캐싱서버에 있는 캐시가 있다. 보안이 필요하면 private를 하면 캐싱서버가 이것을 저장하지 않는다, public은 캐싱서버가 이것을 저장한다. )
* maximum age를 통해서 수명을 정할수도있다.
* **ETag header는 항상 넣어라^^**
