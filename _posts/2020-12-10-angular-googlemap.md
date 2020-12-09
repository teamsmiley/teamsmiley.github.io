---
layout: post
title: "Angular(9/10/11)에서 구글맵 사용하기"
author: teamsmiley
date: 2020-12-10
tags: [coding]
image: /files/covers/blog.jpg
category: { program }
---

# Angular에서 구글맵 사용하기

앵귤러에 구글 맵을 사용해야할 일이 잇어서 작업해보았다.

구글 검색을 하면 Angular Google Maps (AGM) 이라는게 제일 먼저 검색되서 그걸로 작업을 다 하다가 마지막에 angular 9에서 앵귤러가 googlemap 컴포넌트를 냈다는것을 알수 있엇다.
기존걸 다 지우고 이제 다시 해보았다.

https://github.com/angular/components/tree/master/src/google-maps#readme

youtube와 clipboard관련 컴포넌트도 추가되있다고하니 필요하신분은 확인해보면 좋을듯 싶다.

## Install Angular Google Maps component

```bash
npm install @angular/google-maps
```

## change index.html

index.html에 다음 코드를 추가한다.

```html
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY"></script>
```

## 사용하는 모듈에 설정

나는 user.module.ts파일을 사용하는데 이걸 수정하자.

```ts
@NgModule({
  declarations: [
  ],
  imports: [..., GoogleMapsModule],
  exports: [],
  providers: [],
})
export class UserModule {}
```

## html에 랜더링

원하는 컴포넌트에서 다음 태그를 추가

```html
<google-map></google-map>
```

구글 지도가 보이면 성공이다.

지도 위치와 줌을 설정해보자.

```html
<google-map
  [center]="center"
  [zoom]="zoom"
  [options]="options"
  height="500px"
  width="100%"
></google-map>
```

```ts
zoom = 12;
center: google.maps.LatLngLiteral;
options: google.maps.MapOptions = {
  mapTypeId: "terrain",
  zoomControl: false,
  scrollwheel: false,
  disableDoubleClickZoom: true,
  maxZoom: 15,
  minZoom: 8,
};

ngOnInit() {
  // 서버에서 식당 정보를 가져와서 lat/lng를 적용해준다.
  this.service.getRestaurant(this.id).subscribe((result) => {
    this.restaurantResponse = result as RestaurantResponse;
    console.log('lat,lng', this.restaurantResponse?.lat, this.restaurantResponse?.lng);
    this.center = {
      lat: this.restaurantResponse?.lat,
      lng: this.restaurantResponse?.lng,
    };
  });
}
```

이제 center와 zoom이 된 지도가 보이면 된다.

## 지도에 식당을 mark 해보자.

```ts
markers = []; // 배열을 생성해둔다. 여러개가 될수도 있어서

ngOnInit() {
//지도 보여주는 로직 다음에
this.markers.push({
        position: {
          lat: this.restaurantResponse?.lat,
          lng: this.restaurantResponse?.lng,
        },
        label: {
          color: 'red',
          text: this.restaurantResponse?.name,
        },
        title: this.restaurantResponse?.name,
        options: { animation: google.maps.Animation.BOUNCE },
      });
      // markers배열에 1개를 추가한다.
}
```

이제 이걸 기준으로 화면에 뿌려주면 된다.

```html
<google-map
  height="500px"
  width="100%"
  [zoom]="zoom"
  [center]="center"
  [options]="options"
>
  <map-marker
    *ngFor="let marker of markers"
    [position]="marker.position"
    [label]="marker.label"
    [title]="marker.title"
    [options]="marker.options"
  >
  </map-marker>
</google-map>
```

google-map 안에 marker가 추가되야 한다.

이렇게 하면 지도위에 마커가 찍혀서 이름과 함게 보인다.

이정도만 하면 될듯 일단 여기서 멈춤

![]({{ site_baseurl }}/assets/2020-12-08-17-09-21.png)
