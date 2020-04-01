---
layout: post
title: 'Fluent Validation' 
author: teamsmiley
date: 2020-04-01
tags: [Validation]
image: /files/covers/blog.jpg
category: {c#}
---

# Fluent Validation

<https://github.com/FluentValidation/FluentValidation>

C#에서 validation할때 주로 사용됨.

## ScalePrecision 함수 테스트

validation 할 클래스 

```cs
public class Product {
  public float Discount { get; set; }
}
```

validator 작성 

```cs
public class TestValidator : InlineValidator<Product>
{
  public TestValidator()
  {
     RuleFor(x => x.Discount)
            .ScalePrecision(2, 2)
            .NotEmpty()
            ;

  }
```

이 코드를 테스트하다보니 에러가 나야하는데 아무리 해도 에러가 안난다. 그래서 파보기 시작했다.

<https://docs.fluentvalidation.net/en/latest/built-in-validators.html#scaleprecision-validator> 문서를 보자.

문서에는 다음처럼 되잇다.

```
Checks whether a decimal value has the specified scale and precision.

RuleFor(x => x.Amount).ScalePrecision(2, 4);
Example error: ‘Amount’ must not be more than 4 digits in total, with allowance for 2 decimals. 5 digits and 3 decimals were found.
```

`ScalePrecision(int scale, int precision)` 이렇게 되있다. 

scale은 무엇인고 precision은 무엇일가?

precision : 정밀도 : 항상 양수 
scale : 소수점 오른쪽의 소수점 자릿수

precision이 항상 scale보다 같거나 커야한다.

23.5141 => precision : 6 scale: 4

그럼 내코드는 문제가 나야한다. 입력값으로 100.333333f를 넣어줫으므로 4,2로 테스트하면 에러가 나와야하는데 에러가 안나온다. 

왜지?


## Test Validator

```cs
  public TestValidator(params Action<TestValidator>[] actions)
  {
    foreach (var action in actions)
    {
      action(this);
    }
  }
}
```

테스트 코드 작성 
```cs

[Fact]
public void Scale_and_precision_should_work()
{
  var validator = new TestValidator(v => v.RuleFor(x => x.Discount).SetValidator(new ScalePrecisionValidator(2, 4)));

  var result = validator.Validate(new Product { Discount = 123.456778m });
  Assert.False(result.IsValid);

  result = validator.Validate(new Product { Discount = 12.34M });
  Assert.True(result.IsValid);

  result = validator.Validate(new Product { Discount = 12.3414M });
  result.IsValid.ShouldBeFalse();

  result = validator.Validate(new Product { Discount = 1.344M });
  result.IsValid.ShouldBeFalse();

  result = validator.Validate(new Product { Discount = 156.3M });
  result.IsValid.ShouldBeTrue();

```

테스트를 실행해보면 모두 통과한다는것을 알수 있다. 그럼 내가 넣은 값은 float로 바꿔서 테스트해보자.

```cs
[Fact]
public void Scale_and_precision_should_work()
{
  var validator = new TestValidator(v => v.RuleFor(x => x.Discount).SetValidator(new ScalePrecisionValidator(2, 4)));

  var result = validator.Validate(new Product { Discount = 123.456778f });
  Assert.False(result.IsValid);
}
``` 

false가 나와서 테스트를 통과해야하는데 실패 . 실제로는 true가 나와버림.  왜?

소스코드를 봐보자. 

<https://github.com/FluentValidation/FluentValidation/blob/master/src/FluentValidation/Validators/ScalePrecisionValidator.cs>

소스코드가 전부 Decimal로 되있다.

```cs
private static uint GetUnsignedScale(decimal Decimal) {
			var bits = GetBits(Decimal);
			uint scale = (bits[3] >> 16) & 31;
			return scale;
		}
```

결론 : ScalePrecision을 쓸때는 decimal값을 넣어주자.

참고로 다음 차이를 보자.

```
float : 7 digits (32 bit)
double : 15-16 digits (64 bit)
decimal : 28-29 significant digits (128 bit)
```

Decimals and Floats/Doubles cannot be compared without a cast whereas Floats and Doubles can. Decimals also allow the encoding or trailing zeros.
```cs
float flt = 1F/3;
double dbl = 1D/3;
decimal dcm = 1M/3;
Console.WriteLine("float: {0} double: {1} decimal: {2}", flt, dbl, dcm);
```

Result :
```
float: 0.3333333  
double: 0.333333333333333  
decimal: 0.3333333333333333333333333333
```

