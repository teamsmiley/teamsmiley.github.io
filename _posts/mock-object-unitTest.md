--- 
layout: post 
title: "Mock Object Unit Test" 
date: 2017-02-17 01:00  
author: teamsmiley 
tags: [Test]
image: /files/covers/blog.jpg
category: {Program}
---

# Mock Object Unit Test

인터페이스를 만들자. 
```cs
public interface IDiscountHelper {
        decimal ApplyDiscount(decimal totalParam);
}
```

테스트할 클레스가 구현해야할 인터페이스이다.

동작해야할 코드의 조건은 다음과 같다 

* 100불 이상이면 10% 할인 
* 10-100불 사이면 5불 할인 
* 0-10불 사이면 0불 할인 
* -면 에러


구현한 코드는 다음과 같다. 

```cs
namespace EssentialTools.Models {
        public class MinimumDiscountHelper : IDiscountHelper {
                public decimal ApplyDiscount(decimal totalParam) {
                        throw new NotImplementedException();
                }
        }
}
```

실제 내부  구현은 안한다. 무조건 에러를 던진다.

일단 테스트  코드를 만들자 
```cs
namespace EssentialTools.Tests {
        [TestClass]
        public class UnitTest1 {
                private IDiscountHelper getTestObject() {
                        return new MinimumDiscountHelper();
                }
        
                [TestMethod]
                public void Discount_Above_100() {
                        // arrange
                        IDiscountHelper target = getTestObject();
                        decimal total = 200;
                        // act
                        var discountedTotal = target.ApplyDiscount(total);
                        // assert
                        Assert.AreEqual(total * 0.9M, discountedTotal);
                }

                [TestMethod]
                public void Discount_Between_10_And_100() {
                        //arrange
                        IDiscountHelper target = getTestObject();
                        // act
                        decimal TenDollarDiscount = target.ApplyDiscount(10);
                        decimal HundredDollarDiscount = target.ApplyDiscount(100);
                        decimal FiftyDollarDiscount = target.ApplyDiscount(50);
                        // assert
                        Assert.AreEqual(5, TenDollarDiscount, "$10 discount is wrong");
                        Assert.AreEqual(95, HundredDollarDiscount, "$100 discount is wrong");

                        Assert.AreEqual(45, FiftyDollarDiscount, "$50 discount is wrong");
                }

                [TestMethod]
                public void Discount_Less_Than_10() {
                        //arrange
                        IDiscountHelper target = getTestObject();
                        // act
                        decimal discount5 = target.ApplyDiscount(5);
                        decimal discount0 = target.ApplyDiscount(0);
                        // assert
                        Assert.AreEqual(5, discount5);
                        Assert.AreEqual(0, discount0);
                }

                [TestMethod]
                [ExpectedException(typeof(ArgumentOutOfRangeException))]
                public void Discount_Negative_Total() {
                        //arrange
                        IDiscountHelper target = getTestObject();
                        // act
                        target.ApplyDiscount(-1);
                }

        }
}
```

MinimumDiscountHelper 를 만들어서 가져온 다음에  ApplyDiscount를 해서 결과값이  예상값과 같은지 확인한다. 

실제 코드를 구현을 안햇기 때문에 위 조건을 만족시키는 코드를 구현한다. 
```cs
public class MinimumDiscountHelper : IDiscountHelper {
        public decimal ApplyDiscount(decimal totalParam) {
                if (totalParam < 0) {
                        throw new ArgumentOutOfRangeException();
                } else if (totalParam > 100) {
                        return totalParam * 0.9M;
                } else if (totalParam > 10 && totalParam <= 100) {
                        return totalParam -5;
                } else {
                        return totalParam;
                }
        }
}
```
테스트를 돌려본다 통과한다. 

완료..

조금더 복잡한 테스트 구현

인터페이스를 만들자.
```cs
using System.Collections.Generic;
namespace EssentialTools.Models {
        public interface IValueCalculator {
                decimal ValueProducts(IEnumerable<Product> products);
        }
}
```
이 아이를 구현하는 클래스는 다음과 같다.
```cs
using System.Collections.Generic;
using System.Linq;

namespace EssentialTools.Models {

    public class LinqValueCalculator : IValueCalculator {
        private IDiscountHelper discounter;

        public LinqValueCalculator(IDiscountHelper discounterParam) {
            discounter = discounterParam;
        }

        public decimal ValueProducts(IEnumerable<Product> products) {
            return  discounter.ApplyDiscount(products.Sum(p => p.Price));
        }
    }
}
```
테스트를 생성하자. 
목적을 잊지말자 .
목적은 ValueProducts를 테스트하기 위함이다. ...
```
namespace EssentialTools.Tests {
        [TestClass]
        public class UnitTest2 {
                private Product[] products = {
                        new Product {Name = "Kayak", Category = "Watersports", Price = 275M},
                        new Product {Name = "Lifejacket", Category = "Watersports", Price = 48.95M},
                        new Product {Name = "Soccer ball", Category = "Soccer", Price = 19.50M},
                        new Product {Name = "Corner flag", Category = "Soccer", Price = 34.95M}
                };

                [TestMethod]
                public void Sum_Products_Correctly() {
                        // arrange
                        var discounter = new MinimumDiscountHelper();
                        var target = new LinqValueCalculator(discounter);
                        var goalTotal = products.Sum(e => e.Price);
                        // act
                        var result = target.ValueProducts(products);
                        // assert
                        Assert.AreEqual(goalTotal, result);
                }
        }
}
```
AAA(arrange , act , assert) 패턴을 잘 지키며 다 잘했다.

대충 이상없는거같아보인다.

그러나 우리는 앞 예제에서 볼수없었던  문제가 보인다. 

테스트 실패의 경우 우리는 어떤 클래스의 문제인지 알수가 없다.
LinqValueCalculator 인지  MinimumDiscountHelper 인지 확인할 방법이 없다. 

일단 실패를 하고 나면 우리는 IDiscountHelper 의 구현 클래스인 MinimumDiscountHelper 의 클래스 내용까지 확인해야 한다.  이 MinimumDiscountHelper 의 로직이 바뀔경우 이 테스트는 실패하게 된다.  

(MinimumDiscountHelper 의 로직 변경후 테스트는 minimumDiscountHelperTest에서 해야하며 
LinqValueCalculator의 테스트가 실패하면 안되는것이다. )

테스트는 꼭 하나의 테스트만을 해야하는에 이경우는 여러개의 클래스가 합쳐져서 테스트가 가능하게 된 것이다.
문제 다..

이걸 해결하기 위해 mock object가 나왓다.
Mock 오프젝트는 공부따로 하고 여기서는 바로 적용

누겟 moq를 설치한다.

테스트 코드를 수정하자 mock  오브젝트로 진행하자.

```cs
[TestMethod]
public void Sum_Products_Correctly() {
        // arrange
        Mock<IDiscountHelper> mock = new Mock<IDiscountHelper>();
        mock.Setup(m => m.ApplyDiscount(It.IsAny<decimal>())).Returns<decimal>(total => total); // 여기 주목

        var target = new LinqValueCalculator(mock.Object);
        
        // act
        var result = target.ValueProducts(products);
        
        // assert
        Assert.AreEqual(products.Sum(e => e.Price), result);
}
```

간단하게 설명하면 목오브젝트를 생성하는데 여기서 리턴할 값을 미리 지정한다. 
같은 의미로 실제 코드에서는 discounterParam에 따라서 값이 달라지지만 
이 테스트에서는 그건 무시하고 정해진 값이 리턴해지게 만드는 것이다. 
그러고 나면 우리는 ValueProducts 함수 테스트에 집중할수 있게 된다. ==>이게 목오브젝트를 사용하는 목적 

일단 목 오프젝트를 생성한다.

```cs
Mock<IDiscountHelper> mock = new Mock<IDiscountHelper>();
```

<> 안에 제너릭 타입으로 클래스를 넣어주면 된다. 

그리고 메소드를 선택한다. 
```cs
mock.Setup(m => m.ApplyDiscount(It.IsAny<decimal>())).Returns<decimal>(total => total);
```
ApplyDiscount 함수를 가짜로 만들어서 진짜처럼 행동해야하므로 m.ApplyDiscount가 되고 

It.IsAny<decimal>() 는 

It 클래스의 함수이다. 다음 4가지 함수가 있다. 

* Is<T>(predicate) 
* IsAny<T>() 
* IsInRange<T>(min, max,kind)
* IsRegex(expr) 

 IsAny<decimal>  은 값이 decimal 이 들어오는거에 반응한다. 

 이제 결과를 정의한다. 
```cs
.Returns<decimal>(total => total); 
```

total을 넘겨준다.

합치면 
```cs
mock.Setup(m => m.ApplyDiscount(It.IsAny<decimal>())).Returns<decimal>(total => total);
```
이렇게 된다.

이제 mock obj를 이용한다. 
```cs
var target = new LinqValueCalculator(mock.Object);
```

assert한다. 

```cs
Assert.AreEqual(products.Sum(e => e.Price), result);
```

실제 프로덕트 리스트를 가지고 썸을 한결과와 
LinqValueCalculator 의  ValueProducts 해서 나온 결과가 같은지 비교한다. 


이렇게 하는것의 장점은 이 테스트 클래스는 LinqValueCalculator의 ValueProducts 테스트에 
집중하게 만들어 준다. 


이제 나머지 테스트 코드들도 추가 한다. 
```cs
private Product[] createProduct(decimal value) {
        return new[] { new Product { Price = value } };
}
[TestMethod]
[ExpectedException(typeof(System.ArgumentOutOfRangeException))]
public void Pass_Through_Variable_Discounts() {
        // arrange
        //처음에 디스카운트헬퍼 클래스에서 지정한 조건 
        //100불 이상이면 10% 할인 
        //10-100불 사이면 5불 할인 
        //0-10불 사이면 0불 할인 
        //-면 에러

        Mock<IDiscountHelper> mock = new Mock<IDiscountHelper>();
        mock.Setup(m => m.ApplyDiscount(It.IsAny<decimal>())).Returns<decimal>(total => total);
        mock.Setup(m => m.ApplyDiscount(It.Is<decimal>(v => v == 0))).Throws<System.ArgumentOutOfRangeException>();
        mock.Setup(m => m.ApplyDiscount(It.Is<decimal>(v => v > 100))).Returns<decimal>(total => (total * 0.9M));
        mock.Setup(m => m.ApplyDiscount(It.IsInRange<decimal>(10, 100,Range.Inclusive))).Returns<decimal>(total => total - 5);

        var target = new LinqValueCalculator(mock.Object);
        
        // act
        decimal FiveDollarDiscount = target.ValueProducts(createProduct(5));
        decimal TenDollarDiscount = target.ValueProducts(createProduct(10));
        decimal FiftyDollarDiscount = target.ValueProducts(createProduct(50));
        decimal HundredDollarDiscount = target.ValueProducts(createProduct(100));
        decimal FiveHundredDollarDiscount = target.ValueProducts(createProduct(500));
        
        // assert
        // 테스트 코드 결과 확인...
        Assert.AreEqual(5, FiveDollarDiscount, "$5 Fail");
        Assert.AreEqual(5, TenDollarDiscount, "$10 Fail");
        Assert.AreEqual(45, FiftyDollarDiscount, "$50 Fail");
        Assert.AreEqual(95, HundredDollarDiscount, "$100 Fail");
        Assert.AreEqual(450, FiveHundredDollarDiscount, "$500 Fail");
        target.ValueProducts(createProduct(0));
}
```

처음에 디스카운트헬퍼 클래스에서 지정한 조건 
* 100불 이상이면 10% 할인 
* 10-100불 사이면 5불 할인 
* 0-10불 사이면 0불 할인 
* -면 에러

을 가짜로 만들어서 리턴하게 만들어 준 목 오프젝트 셋업이다. 

```cs
//It.Is     Is 함수 v 조건이 만족을 하면  뒤에꺼를 한다.

mock.Setup(m => m.ApplyDiscount(It.Is<decimal>(v => v == 0))).Throws<System.ArgumentOutOfRangeException>();

//v가 0이명 에러를 던진다. 

mock.Setup(m => m.ApplyDiscount(It.Is<decimal>(v => v > 100))).Returns<decimal>(total => (total * 0.9M));
//100보다 작으면 10% 할인가를 제공한다. 

//IsInRange 조건안에 들어가면 뒤에껄 리턴한다. 

mock.Setup(m => m.ApplyDiscount(It.IsInRange<decimal>(10, 100, Range.Inclusive)))
.Returns<decimal>(total => total - 5);

//It.is로 하면 다음 코드와 같다. 
mock.Setup(m => m.ApplyDiscount(It.Is<decimal>(v => v >= 10 && v <= 100)))
.Returns<decimal>(total => total - 5);
```

추가 관련 내용은 다음을 확인 
https://github.com/Moq/moq4/wiki/Quickstart

소스코드 참고는 apress pro  asp.net mvc 4 


## 사용 케이스 

1. Session 및 Identify에 대한 내용을 같이 테스팅 하는 것은 불가능하다.
Mock을 이용해서 HttpContext를 구성하고, 구성된 HttpContext를 ControllerContext로 구성하여 Controller를 사용하게 되면 위와 같은 문제를 모두 해결 할 수 있다. 
<http://netframework.tistory.com/entry/ASP-NET-MVC와-Mock을-이용한-Testing>
2. 한 클래스 테스트에 데이터베이스를 활용해야 하는 경우가 있다고 가정을 합시다. 이 경우 직접 데이터베이스에 정보를 넣었다 뺐다 하면 상당히 큰 부담이 됩니다.
이런 부담을 주지 않고, 데이터베이스의 행동을 흉내냄으로써 테스트를 더욱 용이하게 합니다.
하지만 이런 용이함을 얻기 위해서는 인터페이스들에 의해서만 프로그램을 조작하는 설계를 해야 하기는 합니다.
간단히 이야기해서 Mock 라이브러리는 복잡하고 시간이 걸리는 테스트의 시간을 줄여주기 위해서 존재한다고 볼 수 있습니다.
3. 특정 객체를 구현하다가 그 객체가 종속성을 갖는 다른 객체들이 아직 완성되지 않아서 몰두하여 구현하던 객체를 잠시 버려두고 종속성을 갖는 다른 객체를 구현해야 하는 경우가 발생한다. 
만일 종속성을 갖는 객체가 다른 개발자나 팀에 의해 개발되어야 한다면 기다려야만 한다는 더 안 좋은 문제가 발생한다.
종속성을 갖는 객체를 Mock 객체로 대체하여 
개발자가 현재 자신이 몰두하고 있는 객체에만 전념하여 개발을 진행할 수 있는 방법
이러한 멈춤은 현 작업에서 주의를 분산시키기 때문이다
4. 실제 객체가 비-결정적인(즉, 그때 그때 주변 여건에 따라 다른) 행위를 보인다. 
5. 실제 객체에 대한 초기 설정이 까다롭다. (DB 나 네트웍 커넥션 등이 이에 해당하겠지.)
6. 실제 객체의 행위를 재현하기가 어렵다. (특정 네트웍 에러 등. ) -> 1번 이유와 비슷한 듯.
7. 실제 객체 행위가 느리다. (역시 DB 작업이나 네트웍 통신등이 해당할 듯.)
8. 실제 객체가 UI 를 가지고 있거나, UI 그 자체이다.
9. 테스트 과정에서 객체에 질의를 던져야 할 필요가 있는데, 실제 객체는 해당 질의를 처리하지 않는다. 
예) 콜백이 호출되었는 가? (이 경우 Mock Object는 실제 객체보다 "더 많은" 기능을 가지게 된다. 이는 Stub가 일반적으로 실제 객체보다 "훨씬 적은" 기능을 갖는 것과 반대이다.)
10. 실제 객체는 대부분 "정상 동작"을 하지만, 아주 가끔 "예외 동작"을 한다. 단위 테스트를 통해서 특정 객체의 "정상 동작"/"예외 동작" 여부와 관계없이 시스템의 나머지 부분은 정상적인 동작을 하는 것을 확인하고자 한다. -> 5번 이유와 비슷한 듯.
11. 아직 실제 객체를 만들지 않았다. ->3번과 같음 

## 추가 내용 

* 정의 
 - Mock Object 는 검사하고자 하는 코드와 맞물려 동작하는 객체들을 대신하여 동작하기 위해 만들어진 객체이다. 검사하고자 하는 코드는 Mock Object 의 메서드를 부를 수 있고, 
이 때 Mock Object는 미리 정의된 결과 값을 전달한다. 
MockObject는 자신에게 전달된 인자를 검사할 수 있으며, 이를 테스트 코드로 전달할 수도 있다.


* Martin Fowler TDD 실천자

        * 전통주의자
                가능한 실제 객체를 사용하고, 실제 객체를 사용하기 어려운 경우에만 Test Double 을 사용한다.
                Data warehouse엔 실제 객체를, mail service에는 Test Double을 사용할 것이다

         * Mock 신봉자
                주목할 만한 행위를 가진 객체들에 대해 항상 mock 을 사용한다. 
                 Data warehouse 나 mail service 모두에 Test Double을 사용한다.

