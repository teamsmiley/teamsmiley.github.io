---
layout: post
title: 'Code Review - 2 ' 
author: teamsmiley 
date: 2018-05-14
tags: [code review]
image: /files/covers/blog.jpg
category: {program}
---

# code review - 2

시간 날 때마다 코드 리뷰를 해보고 문서를 작성하자. 

```ts
uploadPostImg(resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url, resource)
        .pipe(
            catchError(this.handleError)
        );

}

uploadPostThumbnail(resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url + '/thumbnail', resource)
        .pipe(
            catchError(this.handleError)
        );
}
```

## 함수 이름이 마음에 안든다. 
이 클래스는 파일을 업로드하는 기능인데 post라는 변수명이 갑자기 등장. 잘못된듯 싶고 img라는 이름은 꼭 이미지만 업로드하는 기능같아 보이는데 코드에는 그런 부분이 없다. 그냥 일반적인 파일업로드만으로 봐야해서 이름을 바꿔야겟다. 

```ts
uploadFile(resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url, resource)
        .pipe(
            catchError(this.handleError)
        );

}

uploadThumbnailFile(resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url + '/thumbnail', resource)
        .pipe(
            catchError(this.handleError)
        );
}
```

## 코드에 중복을 제거 

코드를 보니까 중복이 많다.  매개변수로 빼면 금방 처리될듯 보인다.

```ts
uploadFile(addtionUrl: string, resource: any): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;

    return this.http.post(url + '/' + addtionUrl, resource)
        .pipe(
            catchError(this.handleError)
        );
}

// uploadThumbnailFile(addtionUrl: string, resource: any): Observable<any> {
//     let url = `${this.apiUrl}boardfiles`;

//     return this.http.post(url + '/' + addtionUrl, resource)
//         .pipe(
//             catchError(this.handleError)
//         );
// }

//사용법
 this.fileDataService.uploadFile('', formData)
                .subscribe(result => {
                    console.log('uploaded');
                });
                
 this.fileDataService.uploadFile('thumnail', formData)
                .subscribe(result => {
                    console.log('uploaded');
                });
```

함수 하나로 바꿔져서 해결됬다.

### 사용하는곳에서 저장위치를 마음대로 바꿔버릴 염려가 있다. 

```ts
uploadFile(resource: any, isThumnail:boolean): Observable<any> {
    let url = `${this.apiUrl}boardfiles`;
    if (isThumnail) url = url + '/thumnail';

    return this.http.post(url , resource)
        .pipe(
            catchError(this.handleError)
        );
}

//사용할때
this.fileDataService.uploadFile(formData,true)
                .subscribe(result => {
                    console.log('uploaded');
                });
```

이게 thumnail과 상위폴더 두개로 강제할수 있다. 


api를 보자  post가 3개 있다. 그런데 구분을 url로 한다. 이러면 안됨 미디어 타입으로 구분해줘야한다. 

고치기 전에 구지 3개나 있어야하는지 확인해보자. 

```cs
[HttpPost("{id}")]
public async Task<ActionResult> Post(Guid id)
{
    try
    {
        var files = Request.Form.Files;

        if (files != null && files.Count > 0)
        {
            foreach (var file in files)
            {
                var ext = file.FileName.Split('.').LastOrDefault();
                var filename = id + "_" + DateTime.Now.ToString("yyyyMMdd_HHmmss") + "." + ext;
                //경로는 추후 변경
                var savePath = Path.Combine(@"C:\temp\" +  filename).Replace("/", "\\");

                using (var fileStream = new FileStream(savePath, FileMode.Create))
                {
                    await file.CopyToAsync(fileStream);
                    _unitOfWork.BoardFiles.SaveBoardFile(id, file.FileName, savePath);
                    _unitOfWork.Complete();
                }

                return Created(savePath, file);
            }
            
        }

        return BadRequest();
    }
    catch (Exception ex)
    {
        return StatusCode(500, ex.Message);
    }
}
[HttpPost("thumbnail")]
public async Task<ActionResult> PostThumbnail()
{
    try
    {
        var files = Request.Form.Files;

        if (files != null && files.Count > 0)
        {
            foreach (var file in files)
            {
                var ext = file.FileName.Split('.').LastOrDefault();
                var filename =  DateTime.Now.ToString("yyyyMMdd_HHmmss") + "." + ext;

                string contentRootPath = _hostingEnvironment.ContentRootPath;

                string imgDir =  @"\Hosting\s4.www.zzz.com\wwwroot\thumbnail\";
                var date = DateTime.Now.ToString("yyyyMM");
                var saveDir = Path.Combine(imgDir + date);

                DirectoryInfo thumbnailDirectoryInfo = new DirectoryInfo(saveDir);

                if (!thumbnailDirectoryInfo.Exists)
                {
                    thumbnailDirectoryInfo.Create();
                }

                var saveRootPath = Path.Combine(saveDir + "\\" + filename).Replace("/", "\\");
                var saveDbPath = UPLOAD_URL + "thumbnail/" + date + "\\" + filename;

                using (var fileStream = new FileStream(saveRootPath, FileMode.Create))
                {
                    await file.CopyToAsync(fileStream);
                    //_unitOfWork.BoardDocuments.SaveThumbnail(id, file.FileName, saveDbPath);
                    _unitOfWork.Complete();
                }

                return Ok(saveDbPath);
            }

        }

        return BadRequest();
    }
    catch (Exception ex)
    {
        return StatusCode(500, ex.Message);
    }
}

[HttpPost("img")]
public async Task<ActionResult> PostImg()
{
    try
    {
        var files = Request.Form.Files;

        if (files != null && files.Count > 0)
        {
            foreach (var file in files)
            {
                var ext = file.FileName.Split('.').LastOrDefault();
                var filename = DateTime.Now.ToString("yyyyMMdd_HHmmss") + "." + ext;

                string contentRootPath = _hostingEnvironment.ContentRootPath;

                string imgDir = @"\Hosting\s4.www.zzz.com\wwwroot\img\";
                var date = DateTime.Now.ToString("yyyyMM");
                var saveDir = Path.Combine(imgDir + date);

                DirectoryInfo thumbnailDirectoryInfo = new DirectoryInfo(saveDir);

                if (!thumbnailDirectoryInfo.Exists)
                {
                    thumbnailDirectoryInfo.Create();
                }

                var saveRootPath = Path.Combine(saveDir + "\\" + filename).Replace("/", "\\");
                var saveDbPath = UPLOAD_URL + "img/" + date + "\\" + filename;

                using (var fileStream = new FileStream(saveRootPath, FileMode.Create))
                {
                    await file.CopyToAsync(fileStream);
                }

                return Ok(saveDbPath);
            }

        }

        return BadRequest();
    }
    catch (Exception ex)
    {
        return StatusCode(500, ex.Message);
    }
}
```

첫번째 포스트는 id를 받아서 뭔가를 만드는거고 두번째 3번째는 파일을 저장하는 것이다. 

첫번째 포스트는 필요가 없어 보인다. 클래스 자체가 파일을 저장하는게 목적이기 때문이다. 

첫번재 포스트는 삭제하자. 두번째와 세번째는 코드가 비슷해 보인다. 저장 경로가 달라지는것 같다.

```cs
string imgDir =  @"\Hosting\s4.www.zzz.com\wwwroot\thumbnail\";
```

일단 두번째도 삭제하고 3번째만 가지고 진행을 해보자. 

```cs
private const string UPLOAD_URL = "https://s4.www.zzz.com/";
```

이부분이 하드코딩. 그러면 안되는데 로컬에서 개발시는 로컬환경 서버에 적용시는 서버환경이 되야하기 때문이다. 설정은 꼭 설정파일에 넣는 버릇을 들여야한다. 

appsettings.Development.json에 추가하자. 

```json
  "FileServer": {
    "ServerUrl": "http://localhost:4200",
    "SaveDir": "c:\\temp\\img\\"
  }
```

컨트롤러에서 인젝트 받고 가져다 쓰면 된다.
```cs
private readonly IConfiguration _configuration;

public BoardFilesController(...
            IConfiguration configuration )
        {
           ...
            _configuration = configuration;
        }

var serverUrl = _configuration["FileServer:ServerUrl"];
```

foreach안에서 초기화를 시킬필요가 없어서 상단으로 뺐다. 
```cs
  var serverUrl = _configuration["FileServer:ServerUrl"];
  var imgDir = _configuration["FileServer:SaveDir"];
  string contentRootPath = _hostingEnvironment.ContentRootPath;
  var saveDir = Path.Combine(imgDir + DateTime.Now.ToString("yyyyMM"));
```
combine을 쓰면서 + 로 붙이는건 뭐지? 

foreach 안에서 한번 루프를 돌고 빠져나오네 그럼 파일을 1개만 업로드한다는 이야기같은데. 그럼 files를 받을 이유도 없고. 일단 업로드 완료시 Url을 돌려주는 기능을 없애보자 나중에 넣자. 

변수명이 잘 이해가 안되서 변경했다. 

파일명을 초까지 해서 만들었음 그런데 이건 2개 파일이 거의 동시에 오면 에러가 난다. guid생성해서 붙여서 해결했다. 

```cs
[HttpPost]
public async Task<ActionResult> SaveFiles()
{
    var localPath = _configuration["FileServer:SaveDir"]; //c:\temp\upload_images
    var localPathWithDate = Path.Combine(localPath, DateTime.Now.ToString("yyyyMM"));

    DirectoryInfo localPathWithDateDirectoryInfo = new DirectoryInfo(localPathWithDate);

    if (!localPathWithDateDirectoryInfo.Exists)
    {
        localPathWithDateDirectoryInfo.Create();
    }
    try
    {
        var files = Request.Form.Files;
        if (files != null && files.Count > 0)
        {
            foreach (var file in files)
            {
                var ext = file.FileName.Split('.').LastOrDefault();
                var newFileName = DateTime.Now.ToString("yyyyMMdd_HHmmss") + "-" +  Guid.NewGuid() + "." + ext;
                using (var fileStream = new FileStream(localPathWithDate + "\\" + newFileName, FileMode.Create))
                {
                    await file.CopyToAsync(fileStream);
                }
            }
            return Ok();
        }
        else
        {
            return BadRequest();
        }
    }
    catch (Exception ex)
    {
        return StatusCode(500, ex.Message);
    }
}
```

이제 조금전에 지웟던 2번째 포스트를 다시 구현하자. 내용인즉은 thumbnail이면 imges/thunbnail/아래에 저장을 해야한다는건데 .

```cs
 [HttpPost]
        public async Task<ActionResult> SaveFiles(bool isThumbnail)
        {
            var localPath = _configuration["FileServer:SaveDir"]; //c:\temp\upload_images
            if (isThumbnail) localPath = Path.Combine(localPath, "thumbnail");

            var localPathWithDate = Path.Combine(localPath, DateTime.Now.ToString("yyyyMM"));

            DirectoryInfo localPathWithDateDirectoryInfo = new DirectoryInfo(localPathWithDate);
            if (!localPathWithDateDirectoryInfo.Exists)
            {
                localPathWithDateDirectoryInfo.Create();
            }
```

이렇게 쿼리 스트링으로 받으면 된다. 

완성된 코드 
```cs
[HttpPost]
public async Task<ActionResult> SaveFiles(bool isThumbnail)
{
    var localPath = _configuration["FileServer:SaveDir"]; //c:\temp\upload_images
    if (isThumbnail) localPath = Path.Combine(localPath, "thumbnail");

    var localPathWithDate = Path.Combine(localPath, DateTime.Now.ToString("yyyyMM"));

    DirectoryInfo localPathWithDateDirectoryInfo = new DirectoryInfo(localPathWithDate);
    if (!localPathWithDateDirectoryInfo.Exists)
    {
        localPathWithDateDirectoryInfo.Create();
    }

    try
    {
        var files = Request.Form.Files;
        if (files != null && files.Count > 0)
        {
            foreach (var file in files)
            {
                var ext = file.FileName.Split('.').LastOrDefault();
                var newFileName = DateTime.Now.ToString("yyyyMMdd_HHmmss") + "-" +  Guid.NewGuid()  + "." + ext;
                using (var fileStream = new FileStream(localPathWithDate + "\\" + newFileName, FileMode.Create))
                {
                    await file.CopyToAsync(fileStream);
                }
            }
            return Ok();
        }
        else
        {
            return BadRequest();
        }
    }
    catch (Exception ex)
    {
        return StatusCode(500, ex.Message);
    }
}
```




