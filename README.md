# React, Google Maps, D3 를 이용한 Hexagonal Binning

> 위치기반 데이터를 시각화 할때 기존 Scatterplot 보다 더 좋은 방법은 없을까?

<p align="center">
  <img src="https://cloud.githubusercontent.com/assets/16535279/17847211/c66f66fe-6887-11e6-88a8-3f80d4abad9b.png"/>
</p>

## 목차
1. [결과물] (#end-product)
2. [포스트의 목적] (#objective)
3. [Front-End Core Library] (#core-library)
4. Conceptual Walkthrough
5. Actual Code
6. Wrap-up


## <a name="end-product"></a>결과물
<p align="center">
  <img src="https://cloud.githubusercontent.com/assets/16535279/17847619/e4e94670-6889-11e6-9d73-b396d7d8cba7.png"/>
</p>

Hexagonal Binning 이 적용된 강남구 지역 위치 데이터 분포도입니다. 육각형 통( = hexagonal bin 이하 hexbin) 안에 숫자는 그 육각형 통이 차지하고있는 지역 안에 존재하는 데이터 포인트의 수 입니다. 밀집도를 한눈에 볼수 있도록 각 hexbin 의 색은 현제 zoom level 의 상대적 밀집도를 표현합니다.

샘플데이터를 이용한 Live Demo 보기

## <a name="objective"></a>포스트의 목적

Business Intelligence 란 서비스에 있어서 다음 액션을 취하는데 필요한 객관적인 정보입니다. 메쉬코리아 커머스랩에선 부탁해! 상점 영업상태를 조금더 직관적으로 보고싶었습니다. 상점의 분포도를 scatterplot 으로 표현하는것 이상으로 나아가서 

최대한 간소화된 소스 코드도 나누고 싶었습니다.

이 포스팅은 조금이나마 저희들의 경험을 나누고 싶었습니다. React 와 D3 를 얼마나 아느냐에 따라서 어려울수도 있고 쉬울수도 있다. implementation 최대한 complexity 를 낮추었다. 이 포스팅의 초점은 3가지 의 Gotcha 인데요. 개발하면서 straight-forward 하지 않았던 3가지 포인트입니다. 1. Lat / Lng 은 linear scale 이 아니였다. 2. Svg 가 안보여요! 3. D3-hexbin 사용방법!

Live Demo: 

이 포스트에 소개되는 코드는 본 프로젝트의 원본 소스코드를 블로그 포스트 공유 상황에 맞게 수정한것입니다. 물론 data 도 sample data 로 교체되었습니다.

## <a name="core-library"></a>Front-End Core Library

### React
요즘 React 가 핫합니다. React 란 Facebook 사가 직접만든 UI 개발용 Open-source JavaScript library 입니다.
메쉬코리아에서는 2016년 1월 React 를 높이 평가하여, 내부 web application 에 점진적으로 도입하고 있는데요, React 가 제공하는 디자인 파라다임에 굉장히 만족하고있습니다. 그리고 어떻게 하면 더욱 효과적으로 쓸 수 있나에 대해서 항상 고민하고 있습니다.

### Google Maps
Google Maps 는 다들 아시죠? 위치 기반 서비스를 개발할수 있도록 해주는 API 를 제공합니다. 대안으로는 Naver Map 또는 Daum Maps 가 있겠죠? 
[Google Maps JavaScript API] (https://developers.google.com/maps/documentation/javascript/) 를 React 컴포넌트화 시킨 [react-google-maps] (https://github.com/tomchentw/react-google-maps) library 를 사용했습니다.

### D3
데이터 시각화 library 인 D3 입니다. D3 는 컴퓨터 공학자이자, 데이터 시각화 전문가인 Mike Bostock 이 만든 Open-source libary 로서, raw data 를 다루기에 유용한 함수들부터, 가공된 data 를 효율적으로 DOM 에 렌더링해줄수 있는 유틸리티 함수들을 하나의 library 에 모아서 제공합니다. 또 React 개발자들에게 좋은 소식은, D3 가 4.0 버전으로 올라오면서 모듈화를 선언했다는 점입니다. 이 포스트에서는 구체적으로 D3 의 d3-hexbin, d3-scale, d3-interpolate 모듈들을 사용하여 완성하였습니다.

## Conceptual Walkthrough
!!!!!! needs lots of pics !!!!!!!

### Data
모든 데이터 시각화의 시작은 데이터 입니다. 상점의 분포도를 시각화 하려면 먼저 상점의 위치 데이터가 필요합니다.

	// 상점 하나의 위치 데이터
	const singleLocation = { "lat": 37.505434, "lng": 127.026385 };

위치정보가 하나하나 모여서 배열이 됩니다.
	
	// 상점 위치 데이터를 모아놓은 배열
	const multipleLocations = [ 
		{ "lat": 37.505434, "lng": 127.026385 },
		{ "lat": 37.516167, "lng": 127.040858 },
		...,
	];
	
위의 위치 데이터를 Google Maps 에서 제공 하는 Marker 로 찍을수가 있겠죠.


### 2D Projection

Hexbin 을 사용하기 위해선 상점위치 데이터를 먼저 2D Projection 을 해줘야 합니다. 왜냐하면 위도 경도는 linear scale 이 아니기때문입니다. // 그림으로 설명

Google Maps API 는 Projection 이라는 객채가 있는데, 이 객채는 위치데이터를 위도 경도 에서  x { 0, 256 }, y { 0, 256 } coordinate system 으로 project 해줍니다. 

위
d3-hexagon expects a boundary in a 2D linear plane.

### D3-hexbin
이 프로젝트의 꽃입니다. Point 기반 위치 데이터를 hexbin 에 넣어줍니다.

2D Projection 이 된 data point 들을 특정크기의 hexagon 에 넣어주는데, zoom level 에 따라서 그 hexagon 의 radius 를 조정합니다. 그 이유는 hexagon 의 크기를 일정한 pixel 크기로 fix 하기 위해서입니다.

### Drawing

d3-hexbin 이 제공하는 함수중에 반지름을 넣어주면 그 크기의 육각형 path 를 반환하는 함수가 있습니다. 이 path 를 svg 의 path element 로 넘겨줍니다. 여기서 d3-scale 과 d3-interpolate 를 사용하여 hexbin 의 상대적 밀집도를 계산하고, 색깔을 정해줍니다.

## Actual Code

### 최상위 컴포넌트
#### index.js


### Hexbin
#### D3HexbinComponent.js
```
import React, { Component, PropTypes } from 'react';
import { OverlayView } from 'react-google-maps';
import d3Hexbin from 'd3-hexbin';
import { scaleLinear } from 'd3-scale';
import { max } from 'd3-array';
import { interpolateLab } from 'd3-interpolate';
```	
	
## Conclusion