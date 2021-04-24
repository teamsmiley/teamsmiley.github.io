---
layout: post
title: 'angular file download with browser'
author: teamsmiley
date: 2021-04-01
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# angular file download with browser (not xhr)

일반적인 다운로드는 크롬 다운로드창에서 보이면서 경과가 보여진다.

![]({{ site_baseurl }}/assets/2021-04-01-07-38-56.png)

그런데 angular에서 다운로드를 하면 xhr을 사용하여 스트림을 download하기때문에 파일이 완전히 다운로드가 된 후에 download창을 띄운다.

이 경우에는 file download progress를 화면에 보여주면서 고객이 대기하게 만든다.

작은 파일은 괞찮으나 큰 파일은 고객이 페이지를 바꿔버리면 다운로드가 멈추기 때문에 좋아보이지 않았다.

그래서 크롬 다운로드를 이용하기 위한 삽질을 좀 해봤다.

## xhr을 사용한 다운로드

### API

```cs
[HttpGet("download")]
public async Task<IActionResult> Get(string path, string fileName)
{
  if (string.IsNullOrEmpty(fileName))
  {
    return BadRequest("please provide valid file or valid path name");
  }

  var data = await _service.DownloadFileAsync(path, fileName);

  if (data.FileStream == null)
  {
    return NotFound();
  }

  return File(data.FileStream, data.ContentType);
}
```

### Frontend

1. filesaver 사용

```ts

downloadFile(fileName: string): any {
  return this.http.get(this.url + '/download?path=' + fileName, { responseType: 'blob' }).pipe(catchError(this.handleError));
}

download(element: FileResponse) {
  this.fileService.downloadFile(this._path + '&filename=' + element.name).subscribe((response: any) => {
    var newBlob: any = new Blob([response], { type: response.type });
    const data = window.URL.createObjectURL(newBlob);
    fileSaver.saveAs(newBlob, element.name);
  });
}
```

1. streamSaver 사용

```ts
download(element: FileResponse) {
  this.fileService.downloadFile(this._path + '&filename=' + element.name).subscribe((response: any) => {
    try {
      const fileStream = streamSaver.createWriteStream(element.name, {
        size: response.size,
      });
      // One quick alternetive way if you don't want the hole blob.js thing:
      // const readableStream = new Response(
      //   Blob || String || ArrayBuffer || ArrayBufferView
      // ).body
      const readableStream = response.stream();
      // more optimized pipe version
      // (Safari may have pipeTo but it's useless without the WritableStream)
      if (window.WritableStream && readableStream.pipeTo) {
        return readableStream.pipeTo(fileStream).then(() => console.log('done writing'));
      }
      // Write (pipe) manually
      const writer = fileStream.getWriter();
      const reader = readableStream.getReader();
      const pump = () => reader.read().then((res) => (res.done ? writer.close() : writer.write(res.value).then(pump)));
      pump();
    } catch (e) {
      console.error(e);
    }
  });
```

이렇게 하면 파일은 다운로드가 된다. 그런데 크롬 다운로드창에 넣어주고 싶었다. xhr을 subscribe 하는 구조때문에 이 방식으로는 동작하지 않았다.

## 브라우저 download에서 처리하게 하기

먼저 중요한건 response header에 특정 헤더값을 넣어줘야한다.

### API

```cs
[HttpGet("download")]
public async Task<IActionResult> Get(string path, string fileName)
{
  if (string.IsNullOrEmpty(fileName))
  {
    return BadRequest("please provide valid file or valid path name");
  }

  var data = await _service.DownloadFileAsync(path, fileName);

  if (data.FileStream == null)
  {
    return NotFound();
  }

  //여기
  var header = new ContentDisposition("attachment")
  {
    FileName = fileName,
    Inline = false,
  };

  Response.Headers.Add("Content-Disposition", header.ToString()); //여기
  Response.Headers.Add("Content-Length", data.ContentLength.ToString()); //여기

  return File(data.FileStream, data.ContentType);
}
```

`Content-Length` 를 넣어줘야 브라우저 download에서 진행 % 가 보인다.

### Frontend

간단하게 http 링크를 하나만들어서 테스트해보자.

```html
<a href="https://localhost:5001/files/download?path=/&fileName=test.blend">download</a>
```

![]({{ site_baseurl }}/assets/2021-04-01-07-59-21.png)

예상시간과 진행과정이 보인다.

## 인증을 한후 파일을 다운로드 해야하는 경우

위 a tag 방식으로는 인증 정보를 전달할수 없다. 그래서 앞에 두 방식을 합쳐서 구현해야했다.

![]({{ site_baseurl }}/assets/2021-04-01-08-03-17.png)

1. 앵귤러가 hash endpoint(인증이 필요한) api에 접속을 해서 원하는 파일을 요청을 한다.
1. API 가 인증을 처리한후 Guid를 새로 발급하고 요청받은 파일 정보를 함께 저장해 둔다. (db, redis, memory cache, file) API가 Guid를 리턴해준다.
1. 앵귤러는 받자마자 download를 guid로 요청을 한다. 이 end point는 인증이 없다.
1. donwload endpoint는 저장해둔 정보를 찾아서 실제 경로를 가져와서 파일을 스트림해서 보내준다. 이때 헤더값을 잘 줘서 크롬 다운로드창이 뜨게 해줘야한다.

이해가 되었나? 중요한점은 download endpoint에서 header값을 잘 줘야한다는것. 파일정보를 저장을 가능한한 짧은 시간을 해야한다는것.

나는 db / redis등이 귀찮아서 memory cache를 사용했다. 정보는 3초만 보관되고 없어지게 하였다.

쿠버네티스를 사용하다보니 컨테이너가 여러개 뜰경우 memory cache가 다른 컨테이너로 가면 안되서 쿠버네티스에서 sticky session을 사용하여 같은 유저는 같은컨테이너로 가게 해두었다.

코드를 보면 다음과 같다.

### API

```cs
[HttpGet("hash")]
public async Task<IActionResult> Get(string path, string fileName)
{
  if (string.IsNullOrEmpty(fileName))
  {
    return BadRequest("please provide valid file or valid path name");
  }

  // Key not in cache, so get data.
  var fileId = Guid.NewGuid();

  // Set cache options.
  var cacheEntryOptions = new MemoryCacheEntryOptions()
      .SetAbsoluteExpiration(TimeSpan.FromSeconds(3))
      ;

  // Save data in cache.
  _cache.Set(fileId, new DownloadFileInfo { Path = path, FileName = fileName }, cacheEntryOptions);

  return Ok(fileId);
}

[HttpGet("download")]
[AllowAnonymous] //인증없이 요청이 가능하게
public async Task<IActionResult> Get(Guid fileId)
{
  DownloadFileInfo cacheEntry;
  _cache.TryGetValue(fileId, out cacheEntry);

  System.Console.WriteLine($"asdf {cacheEntry.FileName}");

  var path = cacheEntry.Path;
  var fileName = cacheEntry.FileName;

  var data = await _service.DownloadFileAsync(path, fileName);

  if (data.FileStream == null)
  {
    return NotFound();
  }

  // 이걸 해줘야 다운로드창이 뜬다.
  var header = new ContentDisposition("attachment")
  {
    FileName = fileName,
    Inline = false,
  };

  Response.Headers.Add("Content-Disposition", header.ToString());
  Response.Headers.Add("Content-Length", data.ContentLength.ToString());

  return File(data.FileStream, data.ContentType);
}

public class DownloadFileInfo
{
  public string Path { get; set; } = string.Empty;
  public string FileName { get; set; } = string.Empty;
}
```

### Frontend

```ts
downloadHash(queryString?: any) {
  return this.http
    .get(this.url + '/hash' + this.toQueryString(queryString), {
      headers: { Accept: 'application/json' },
    })
    .pipe(catchError(this.handleError));
}

//----

getFileHash(queryString?: any) {
  return super.downloadHash(queryString);
}

//----
async download(path,fileName) {
  const query = {
    path: path,
    fileName: fileName,
  };
  this.fileService.getFileHash(query).subscribe((res) => {
    window.location.href = this.apiURL + '/files/download?fileId=' + res;
  });
}
```

downloadHash 에서 interceptor를 세팅해두면 token이 들어간다.

이러면 다운로드가 크롬 브라우저에서 잘된다.
