---
layout: post
title: 'XUnit Test' 
author: teamsmiley
date: 2020-03-11
tags: [test]
image: /files/covers/blog.jpg
category: {test}
---

# XUnit Test

테스트를 작성해보려고 했다. 그런데 기본 내용이 잘 기억나지 않아서 다시 정리해보려고 한다. 

사용하는 툴은 XUnit 이다.

## automatic test

## Test Type
* Unit Test
* Integration Test
* Subcutaneous Test
* UI Test
* End To End Test

## Testing behaviour
* dont test detail implementation (dont test private Method)
* 상세 구현을 테스트하려고 하지 말라. 프라이빗 함수도 테스트하지 마라.

## AAA

* Arange : 설정
* Act    : 실행
* Assert : 검사

## 프로젝트 생성 
* xunit project생성하면된다.
* xunit, xunit.runner.visualstudio nuget package설치해도 됨.
* `dotnet new xunit`

## 테스트 실행 
* Test expolor(visual studio)
* dotnet test (command line)

## how many assert per test
```cs
public async Task<Guid> Handle(CreateProductCommand request, CancellationToken cancellationToken)
{
  //1
  MyAuthorization au = new MyAuthorization(_context);
  if (!au.IsAdmin(_currentUserService.UserId))
    throw new UnauthorizedAccessException();

  //2 
  var validator = new CreateProductCommandValidator();
  var validatorResult = validator.Validate(request);
  if (!validatorResult.IsValid)
    throw new CustomValidationException(validatorResult.Errors);

  //3
  var productEntity = _mapper.Map<Product>(request);
  _context.Products.Add(productEntity);
  await _context.SaveChangesAsync(cancellationToken);
  return productEntity.Id;
}
```
위와 같으면 총 3개의 테스트가 필요한듯  대충 보기에 함수에서 빠져나가는 부분을 테스트하면되는듯. 제일 먼저 할건 happy case로 맨마지막까지 가는 코드를 테스트해보면 나머지는 조금 쉬워지는듯 보인다.

*궁금한것

다른 테스트 클래스가 CreateProductCommandValidator를 테스트한다. 그런데 2번이 꼭 필요한것인가? ==> 고민을 좀 해보니 다른 테스트는 CreateProductCommandValidator를 테스트하는것이고 //2 번은 튕겨나올때 exception을 발생시키는것을 테스트하는 것이니 하는게 맞는듯

## String Assert Code
```cs
Assert.True()
Assert.Equal() 
Assert.StartWith()
Assert.EndWith()
Assert.Contain()
Assert.Matches("regx","aaa")
Assert.Equal("aaa", sut.name, ignoreCase: true) //ignore case
```

## Number Assert Code
```cs
Assert.Equal() 
Assert.NotEqual() 
Assert.True()
Assert.InRange()
Assert.True( sut.age >= 100 && sut.age <= 111)
Assert.InRange<int>(sut.age,100,111)
Assert.Equal(100.00, 100.00000001) // false
Assert.Equal(100.00, 100.00000001,2) // true 소수점 2자리
```
## Null Value
```cs
Assert.Null(sut.name)
Assert.NotNull()
```

## Array
```cs
Assert.Contains("aaa", sut.Names)
Assert.DoesNotContain("aaa",sut.Names)
Assert.Contains(sut.Names, name => name.Contains("brian"))
```

## Object
```cs
Assert.IsType<Product>(product)
Assert.IsNotType<Product>(product)
Assert.IsAssingableFrom<Product>(test)
Assert.Same(aaa,bbb)
Assert.NotSame(aaa,bbb)
```

## Exception
```cs
var ex = Assert.Throws<ArgumentNullException>(() => sut.Create(null));
var ex = Assert.Throws<AdAccountInvalidException>(() => (AdAccount)"AAA");
var ex = await Assert.ThrowsAsync<UnauthorizedAccessException>(async () => await sut.Handle(command, CancellationToken.None));
Assert.Equal("aaa",ex.message)
```

## Event raised
```cs
Assert.Raises<EventArgs>(
  handler => sut.SendEmail += handler, //이벤트 추가
  handler => sut.SendEmail -= handler, //이벤트 삭제
  () => sut.aaa()
);

Assert.PropertyChanged(sut,"AAA", () => sut.KKK(10)); //AAA property name
```

## categorize
* Trait : 테스트들을 구분해준다.
```cs
[Fact]
[Trait("Category"."product")]
public async Task Handle_ShouldCreateProduct()
{
  //Arrange
  var command = new CreateProductCommand
  {
    Name = "test product",
    Price = 100.00f,
    Description = "test description",
    IsAvailable = true,
  };

  //Act
  var sut = new CreateProductCommand.CreateProductCommandHandler(Context,Mapper,_currentUserServiceMock.Object);

  var result = await sut.Handle(command, CancellationToken.None);

  //Assert
  var entity = Context.Products.Find(result);

  entity.ShouldNotBeNull();
  entity.Name.ShouldBe(command.Name);
}
```

* command line : `dotnet test --filter "Category=product | Category=product2"`

## Skip Test
```cs
[Fact(Skip="reason")]
```

## Output
```cs
using Xunit;
using Xunit.Abstractions;

private readonly ITestOutputHelper _output;

public CreateProductCommandTests(ITestOutputHelper output)
{
  _output = output;
}

public async Task test(){
  _output.WriteLine(command.Name);
}
```

* command line : `dotnet test --logger:trx`

## constructor

xunit에서 테스트마다 인스턴스를 만들지 말고 생성자에서 한번 생성하면 된다 모든 테스트함수는 생성자를 실행후 실행된다. 참고로 Dispose 도 각각 테스트 끝나고 바로 실행된다.


## sharing context betwen multiple Test Method

그냥 테스트하나에서 두개 생성해도 되고 이게 여의치 않는 경우는 

AAAFixture 클래스를 생성한다. 거기에 관련 내용을 넣는다.

테스트 클래스에서 IClassFixture를 상속받는다.
```cs
public class MappingTests : IClassFixture<MappingTestsFixture>
{
  private readonly MappingTestsFixture _fixture;

  public MappingTests(MappingTestsFixture fixture)
  {
    _fixture = fixture;
  }

  [Fact]
  public void ShouldSupportMappingFromSourceToDestination(Type source, Type destination)
  {
    _fixture.Map(instance, source, destination); //위에서 인젝션 받은 내용을 사용한다.
  }
}

```

이렇게 해서 테스트를 작성하면 한번 클래스를 만들고 끝나면 클래스를 지운다.

## shared context between multiple Test Class

fixture를 받는 collection을 만들어서 사용

```cs
[CollectionDefinition("QueryTests")]
public class QueryCollection : ICollectionFixture<QueryTestFixture> { }
```

테스트클래스에서는 다음처럼 
```cs
[Collection("QueryTests")]
public class GetProductQueryTests
{
```

## Data-driven Test : Theory 
```cs
[Theory]
[InlineData(typeof(TodoList), typeof(TodoListDto))]
[InlineData(typeof(TodoItem), typeof(TodoItemDto))]
public void ShouldSupportMappingFromSourceToDestination(Type source, Type destination)
{
  var instance = Activator.CreateInstance(source);

  _mapper.Map(instance, source, destination);
}
```

하나의 테스트함수에 아규먼트를 여러개로 바꿔가면서 줄수 있다. 기존에는 함수를 계속 복사해서 만들어야해서 코드 중복이 많앗는데 theory를 사용하면 코드 중복이 사라진다.

InlineData 를 두번줘서 이 테스트함수는 2번 실행이 된다.

## sharing test data accoss Tests

테스트 데이터를 여러군데에서 사용해야할경우

SharedTestData.cs 클래스를 만들어서 inline data에 잇는 내용을 넣는다.

```cs
public class SharedTestData
{
  public static IEnumerable<object[]> TestData
  {
   
    //private static readonly List<object[]> Data = new List<object[]>
    //     {
    //            new object[] {0, 100},
    //            new object[] {1, 99},
    //            new object[] {50, 50},
    //            new object[] {101, 1}
    //     };

    //public static IEnumerable<object[]> TestData => Data;

    get
    {
        yield return new object[] { 0, 100 };
        yield return new object[] { 1, 99 };
        yield return new object[] { 50, 50 };
        yield return new object[] { 75, 25 };
        yield return new object[] { 101, 1 };
    }
  }
}
```

주석한 내용이 더 좋으면 그렇게 해도 된다.

이제 테스트하고 싶은 함수에서 가져다 쓴다.

```cs
[Theory]
[MemberData("TestData", MemberType = typeof(SharedTestData))]
public void AAA(int aaa, int bbb)
{
  ...
}

```

MemberData 가 중요

"TestData"가 스트링이라 좋은 코드는 아니다. 다음처럼 바꿀수 있다.
```cs
[MemberData(nameof(SharedTestData.TestData),MemberType = typeof(SharedTestData))]
```

리네임을 하거나 할때 같이 바뀌고 컴파일시 에러가 나서 오류를 쉽게 찾을수 있다.

## 외부에서 데이터를 가져와야하는경우 (예 csv)
```cs
public class ExternalTestData
{
    public static IEnumerable<object[]> TestData
    {
        get
        {
            string[] csvLines = File.ReadAllLines("TestData.csv");

            var testCases = new List<object[]>();

            foreach (var csvLine in csvLines)
            {
                IEnumerable<int> values = csvLine.Split(',').Select(int.Parse);

                object[] testCase = values.Cast<object>().ToArray();

                testCases.Add(testCase);
            }

            return testCases;
        }
    }
}
```

이제 사용하는 곳에서 
```cs
[Theory]
[MemberData(nameof(ExternalTestData.TestData),MemberType = typeof(ExternalTestData))]
```

## custom data attribute 

`[MemberData(nameof(ExternalTestData.TestData),MemberType = typeof(ExternalTestData))]` 이부분이 계속 반복된다. 이걸 한줄로 줄여보자.

클래스를 하나를 만든다. 
```cs
public class CustomDataAttribute : DataAttribute
{
  public override IEnumerable<object[]> GetData(MethodInfo testMethod)
  {
    yield return new object[] { 0, 100 };
    yield return new object[] { 1, 99 };
    yield return new object[] { 50, 50 };
    yield return new object[] { 75, 25 };
    yield return new object[] { 101, 1 };
  }
}
```

이제 테스트클래스에서 
```cs
[Theory]
//[MemberData(nameof(ExternalHealthDamageTestData.TestData), 
//    MemberType = typeof(ExternalHealthDamageTestData))]
[CustomData]
```
라고 사용하면 된다.

물론 yield 부분을 외부에서 데이터를 가져오는 부분으로 수정해도 된다.

```cs
public override IEnumerable<object[]> GetData(MethodInfo testMethod)
{
    string[] csvLines = File.ReadAllLines("TestData.csv");

    var testCases = new List<object[]>();

    foreach (var csvLine in csvLines)
    {
        IEnumerable<int> values = csvLine.Split(',').Select(int.Parse);

        object[] testCase = values.Cast<object>().ToArray();

        testCases.Add(testCase);
    }

    return testCases;
}
```


