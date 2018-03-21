# Angular Custom Header 

Angular 프로젝트를 하던중 삽질을 한것이 있어서 적어둔다.

api 는 다음과 같다. 

```cs
[HttpGet("{id}", Name = "GetBoardDocument")]
public IActionResult GetBoardDocument(int id, [FromQuery] string fields)
{
    var apps = _unitOfWork.BoardDocuments.Get(id);

    if (apps == null)
    {
        return NotFound();
    }
    var result = Mapper.Map<BoardDocumentVM>(apps);

    Response.Headers.Add("X-SubjectId", "test");
    return Ok(result.ShapeData(fields));
}
```

앵귤러 기존코드 

```javascript
getAllQueryString(queryString : string) {
let options = this.setHeader('application/json');

return this.http.get(this.url + queryString,options)
    .map(response => response.json())
    .catch(this.handleError);
  }
```

헤더를 가져와 보자. 
```javascript
getAllQueryString(queryString : string) {
let options = this.setHeader('application/json');

return this.http.get(this.url + queryString,options)
    .map(res => {
        var payload = res.json();
        var headers = res.headers;
        console.log("header: " , headers)
        return { payload,headers}
      })
    .catch(this.handleError);
  }
```

이제 헤더를 찍어볼수 있게 되었다. 

그런데 막상 해보면 헤어가 안찍힌다. 왜냐면 cors때문.

다음처럼 헤더에 뭐를 쓴다고 같이 알려줘야 한단다. 
```
Access-Control-Expose-Headers: Content-Length, X-My-Custom-Header, X-Another-Custom-Header
```

그러므로 api코드를 수정하자. 
```cs
[HttpGet("{id}", Name = "GetBoardDocument")]
public IActionResult GetBoardDocument(int id, [FromQuery] string fields)
{
    var apps = _unitOfWork.BoardDocuments.Get(id);

    if (apps == null)
    {
        return NotFound();
    }
    var result = Mapper.Map<BoardDocumentVM>(apps);

    Response.Headers.Add("X-SubjectId", "test");

    Response.Headers.Add("Access-Control-Expose-Headers", "X-Pagination,X-CustomHeader");
    return Ok(result.ShapeData(fields));
}
```

이제 앵귤러에서 찍어보면 잘 찍힌다.

