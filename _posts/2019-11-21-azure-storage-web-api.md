---
layout: post
title: 'webapi file up/download with Azure Storage' 
author: teamsmiley
date: 2019-11-21
tags: [azure]
image: /files/covers/blog.jpg
category: {dev}
---

# Web API File up/download with Azure Storage 

webapi로 파일을 업로드 다운로드 해볼 일이 있어서 정리해본다.

업로드는 다음 두가지 시나리오를 생각해 봤다. 

1. api에서 받아서 로컬 또는 pod에 파일을 업로드/다운로드 하는 경우 
1. api에서 받아서 azure blob storage나 s3로 업로드하는 경우

다운로드의 경우 1번은 실제 파일을 다운로드 시켜주는건 문제가 없어보이지만 2번의 경우는 다른 시나리오가 생긴다. 

1. api에서 azure blob storage나 s3에서 받아서 메모리 스트림으로 다시 요청자에게 보내는 방법 
1. api에서 azure를 이용해서 임시 토큰을 발급해서 그 주소를 리다이렉트로 요청자에게 보내면 요청자는 자동으로 s3나 azure blob storage로 요청을 해서 거기서 직접 받는경우 

이렇게 두가지 시나리오가 생긴다. 

이걸 정리하면 server가 파일을 remote storage에서 받아서 스트림으로 보내냐 요청자에게 리다이렉트를 줘서 요청자가 remote storage에서 직접 받는냐. 이런 차이가 있다.

![](file-download-remote-server.png)

다운로드의 경우 1번만 해봣는데 2번은 누가 해봣으면 알려주시기 바랍니다.

Azure의 경우 Azure Storage Explorer 를 제공해준다. 일단 이걸 설치하자.

<https://azure.microsoft.com/en-us/features/storage-explorer/>

설치후 실행해보면 다음이 보인다. 

![]({{ site.baseurl }}/assets/2019-11-21-07-14-44.png)

emulator가 보이는가? 그렇다 윈도우의 경우 에뮬레이터를 이용할수 있다.(macos와 linux는 오픈소스가 있다고함 확인안해봣음.)

emulator를 설치하여 로컬에서 해보자. 

<https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator>

standalone installer 라는 부분을 클릭하여 다운로드 받은후 실행한다.

화면에 오른쪽 하단에 아이콘이 하나생기고 아이콘 위에서 오른쪽 버튼 클릭하면 옵션이 나오는것을 알수 있다.

![]({{ site.baseurl }}/assets/2019-11-21-07-18-30.png)

show storage emulator ui를 클릭해보자.

![]({{ site.baseurl }}/assets/2019-11-21-07-18-59.png)

이제 초기화를 하고 실행해보자. 

```
AzureStorageEmulator.exe init
AzureStorageEmulator.exe start
AzureStorageEmulator.exe status
```
에뮬레이터가 실행됨을 알수 있다. 

Azure Storage Explorer를 실행하여 내용을 확인해보자.

![]({{ site.baseurl }}/assets/2019-11-21-07-21-14.png)

create blob을 해서 document라고 이름을 준다. 이것이 컨테이너 이름이다. 나중에 사용할것임

이제 컨테이너를 만들엇으니 코딩을 해서 업로드를 해보자.

* appsettings.json 을 수정한다.
```json
  "StorageConnectionString": "UseDevelopmentStorage=true",
  "ContainerName": "document"
```

document 는 조금전에 만든 컨테이너 이름

* nuget package 설치 

```
dotnet add package WindowsAzure.Storage --version 9.3.3
```

* factory와 service를 만들자. 

AzureBlobConnectionFactory.cs
```cs
using Microsoft.Extensions.Configuration;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using System;
using System.Threading.Tasks;

namespace SP.WebAPI.Services
{
	public interface IAzureBlobConnectionFactory
	{
		Task<CloudBlobContainer> GetBlobContainer();
	}

	public class AzureBlobConnectionFactory : IAzureBlobConnectionFactory
	{
		private readonly IConfiguration _configuration;
		private CloudBlobClient _blobClient;
		private CloudBlobContainer _blobContainer;

		public AzureBlobConnectionFactory(IConfiguration configuration)
		{
			_configuration = configuration;
		}

		public async Task<CloudBlobContainer> GetBlobContainer()
		{
			if (_blobContainer != null)
				return _blobContainer;

			var containerName = _configuration.GetValue<string>("ContainerName");
			if (string.IsNullOrWhiteSpace(containerName))
			{
				throw new ArgumentException("Configuration must contain ContainerName");
			}

			var blobClient = GetClient();

			_blobContainer = blobClient.GetContainerReference(containerName);
			if(await _blobContainer.CreateIfNotExistsAsync())
			{
				await _blobContainer.SetPermissionsAsync(new BlobContainerPermissions { PublicAccess = BlobContainerPublicAccessType.Blob });
			}
			return _blobContainer;
		}

		private CloudBlobClient GetClient()
		{
			if (_blobClient != null)
				return _blobClient;

			var storageConnectionString = _configuration.GetValue<string>("StorageConnectionString");
			if(string.IsNullOrWhiteSpace(storageConnectionString))
			{
				throw new ArgumentException("Configuration must contain StorageConnectionString");
			}

			if(!CloudStorageAccount.TryParse(storageConnectionString, out CloudStorageAccount storageAccount))
			{
				throw new Exception("Could not create storage account with StorageConnectionString configuration");
			}

			_blobClient = storageAccount.CreateCloudBlobClient();
			return _blobClient;
		}
	}
}
```

AzureBlobService.cs
```cs
using Microsoft.AspNetCore.Http;
using Microsoft.WindowsAzure.Storage.Blob;
using SP.Application.Exceptions;
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading.Tasks;

namespace SP.WebAPI.Services
{
  public interface IAzureBlobService
  {
    Task<IEnumerable<Uri>> ListAsync();
    Task<string> UploadAsync(IFormFile file,string userId);
    Task DeleteAsync(string fileUri);
    Task DeleteAllAsync();
    Task<Stream> DownloadAsync(string filePath);
  }

  public class AzureBlobService : IAzureBlobService
  {
    private readonly IAzureBlobConnectionFactory _azureBlobConnectionFactory;

    public AzureBlobService(IAzureBlobConnectionFactory azureBlobConnectionFactory)
    {
      _azureBlobConnectionFactory = azureBlobConnectionFactory;
    }

    public async Task DeleteAllAsync()
    {
      var blobContainer = await _azureBlobConnectionFactory.GetBlobContainer();

      BlobContinuationToken blobContinuationToken = null;
      do
      {
        var response = await blobContainer.ListBlobsSegmentedAsync(blobContinuationToken);
        foreach (IListBlobItem blob in response.Results)
        {
          if (blob.GetType() == typeof(CloudBlockBlob))
            await ((CloudBlockBlob)blob).DeleteIfExistsAsync();
        }
        blobContinuationToken = response.ContinuationToken;
      } while (blobContinuationToken != null);
    }

    public async Task DeleteAsync(string fileUri)
    {
      var blobContainer = await _azureBlobConnectionFactory.GetBlobContainer();

      Uri uri = new Uri(fileUri);
      string filename = Path.GetFileName(uri.LocalPath);

      var blob = blobContainer.GetBlockBlobReference(filename);
      await blob.DeleteIfExistsAsync();
    }

    public async Task<IEnumerable<Uri>> ListAsync()
    {
      var blobContainer = await _azureBlobConnectionFactory.GetBlobContainer();
      var allBlobs = new List<Uri>();
      BlobContinuationToken blobContinuationToken = null;
      do
      {
        var response = await blobContainer.ListBlobsSegmentedAsync(blobContinuationToken);
        foreach (IListBlobItem blob in response.Results)
        {
          if (blob.GetType() == typeof(CloudBlockBlob))
            allBlobs.Add(blob.Uri);
        }
        blobContinuationToken = response.ContinuationToken;
      } while (blobContinuationToken != null);
      return allBlobs;
    }

    public async Task<string> UploadAsync(IFormFile file,string userId)
    {
      var blobContainer = await _azureBlobConnectionFactory.GetBlobContainer();

      var newFileFullPath = userId + "/" + GetRandomBlobName(file.FileName);

      var blob = blobContainer.GetBlockBlobReference(newFileFullPath);
      using (var stream = file.OpenReadStream())
      {
        await blob.UploadFromStreamAsync(stream);
      }

      return newFileFullPath;
    }

    private string GetRandomBlobName(string filename)
    {
      return string.Format("{0:10}_{1}_{2}", DateTime.Now.Ticks, Guid.NewGuid(), filename);
    }

    public async Task<Stream> DownloadAsync(string filePath)
    {
      MemoryStream ms = new MemoryStream();
      
      var blobContainer = await _azureBlobConnectionFactory.GetBlobContainer();

      var blob = blobContainer.GetBlockBlobReference(filePath);

      if (await blob.ExistsAsync())
      {
        await blob.DownloadToStreamAsync(ms);
        return ms;
      }
      else
      {
        throw new NotFoundException(nameof(AzureBlobService),"file is not exist" );
      }
    }
  }
}

```

* controller를 만들어서 업로드해보자. 

FilesController.cs
```cs
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using SP.WebAPI.Helpers;
using SP.WebAPI.Services;

namespace SP.WebAPI.Controllers
{
  [Route("files")]
  [ApiController]
  public class FilesController : ControllerBase
  {
    private readonly IAzureBlobService _azureBlobService;

    public FilesController(IAzureBlobService azureBlobService)
    {
      _azureBlobService = azureBlobService;
    }

    [HttpOptions]
    [ProducesResponseType(typeof(void), StatusCodes.Status200OK)]
    public IActionResult Options()
    {
      Response.Headers.Add("Allow", "OPTIONS,POST");
      return Ok();
    }

    [HttpPost]
    public async Task<IActionResult> UploadFile([FromForm(Name = "file")] IFormFile file)
    {

      var userId = new Guid(User.Claims.FirstOrDefault(t => t.Type == "sub").Value).ToString(); //userid를 이용하여 폴더를 만든후 그 하위에 저장하기 위함.

      if (file == null)
      {
        return BadRequest("Could not upload files");
      }

      var newFileName = await _azureBlobService.UploadAsync(file, userId);
      
      var result = new
      {
        UploadFileName = newFileName
      };

      return new JsonResult(result); //생긴 파일명을 화면에 뿌려주기 위함.
    }

    [HttpGet]
    public async Task<IActionResult> DownloadFile(string fileName)
    {
      if (fileName == null)
      {
        return BadRequest("Missing fileName");
      }

      var stream = await _azureBlobService.DownloadAsync(fileName);
      
      if (stream == null)
        return NotFound();

      stream.Position = 0; //이거 안하면 500에러나거나 0k파일이 만들어짐

      return  File(stream, "application/octet-stream"); 

    }
  }
}
```

위 코드는 3일을 삽질한후 정리한것입니다. 여러군데의 인터넷을 참조햇을수 있습니다.

문제엿던 부분을 간단히 설명하면 

upload에서는 [FromForm(Name = "file")] 이부분을 안넣으면 file이 넘어오지 않던 문제가 있었음.

download에서는 stream.Position = 0; 이부분을 추가하지 않으면 500에러가 났습니다. 

둘다 메뉴얼에는 없어서 한참을 삽질을 했습니다. 

이렇게 하면 이제 업로드 다운로드가 될것입니다.

Postman으로 테스트해보겟습니다.(미리 Azure Storage Explorer를 통해 에뮬레이터에 document에 파일을 올려두었습니다. )

다운로드 

![]({{ site.baseurl }}/assets/2019-11-21-07-32-28.png)

https://localhost:7001/files?fileName=userid/aaa.png

이런 주소로 요청을 하면 파일이 잘 받아집니다. `fileName=` 이 부분이 중요한데 코드에 

![]({{ site.baseurl }}/assets/2019-11-21-07-33-38.png)

이부분과 일치해야 합니다.

업로드

![]({{ site.baseurl }}/assets/2019-11-21-07-39-52.png)

A 부분을 누르시면 file을 첨부할수 있습니다. 

업로드도 잘 됨을 알수 있습니다. 여기에 file항목이름은 controller에서

![]({{ site.baseurl }}/assets/2019-11-21-07-41-33.png)

이부분과 같아야 합니다.


서비스에 올리시려면 azure 에 로그인한후 storage account를 생성합니다.

생성후 Access keys에 가면  key가 있습니다.(왜 키가 2개인지 아시는분) connection string을 복사하여 appsetting.json에 넣어주면 됩니다.

```json
//"StorageConnectionString": "UseDevelopmentStorage=true",
"StorageConnectionString": "DefaultEndpointsProtocol=https;AccountName=sample;AccountKey=CXfaH1gSJU6Zu1bV1/44r0QpLWNIHHbCVY17lCwg5HJKBwFd8PD87jA==;EndpointSuffix=core.windows.net"
```


이상 끝.







