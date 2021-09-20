---
layout: post
title: 최대 함량 페인트(LCP)
authors:
  - philipwalton
date: '2019-08-08'
updated: '2020-06-17'
description: |2

  이 게시물은 LCP(Large Contentful Paint) 측정항목을 소개하고 설명합니다.

  그것을 측정하는 방법
tags:
  - performance
  - metrics
---

{% Aside %} LCP(Large Contentful Paint)는 페이지의 주요 콘텐츠가 로드되었을 가능성이 있는 페이지 로드 타임라인의 지점을 표시하기 때문에 [감지된 로드 속도를 측정하기 위한 중요한 사용자 중심 측정항목입니다. 빠른 LCP는 사용자가](/user-centric-performance-metrics/#types-of-metrics) 페이지가 [유용합니다](/user-centric-performance-metrics/#questions) . {% endAside %}

역사적으로 웹 개발자가 웹 페이지의 주요 콘텐츠가 얼마나 빨리 로드되고 사용자에게 표시되는지 측정하는 것은 어려운 일이었습니다.

[load](https://developer.mozilla.org/en-US/docs/Web/Events/load) 또는 [DOMContentLoaded](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded) 와 같은 오래된 메트릭은 사용자가 화면에서 보는 것과 반드시 일치하지 않기 때문에 좋지 않습니다. [그리고 FCP(First Contentful Paint)](/fcp/) 와 같은 새로운 사용자 중심 성능 메트릭은 로딩 경험의 시작 부분만을 포착합니다. 페이지에 시작 화면이 표시되거나 로딩 표시기가 표시되면 이 순간은 사용자와 별로 관련이 없습니다.

과거에 우리는 초기 페인트 후 더 많은 로딩 경험을 포착하는 데 도움이 되도록 [FMP(First meaningful Paint)](/first-meaningful-paint/) 및 [SI(Speed Index)](/speed-index/) (둘 다 Lighthouse에서 사용 가능)와 같은 성능 메트릭을 권장했지만 이러한 메트릭은 복잡하고 설명하기 어렵습니다. , 그리고 종종 잘못된 것입니다. 즉, 페이지의 주요 콘텐츠가 로드된 시점을 여전히 식별하지 못합니다.

때로는 단순한 것이 더 좋습니다. [W3C Web Performance Working Group](https://www.w3.org/webperf/) 의 토론과 Google에서 수행된 연구에 따르면 페이지의 주요 콘텐츠가 로드되는 시기를 측정하는 보다 정확한 방법은 가장 큰 요소가 렌더링된 시기를 확인하는 것입니다.

## LCP란?

LCP(Large Contentful Paint) 측정항목 [은 페이지가 처음 로드되기 시작한](https://w3c.github.io/hr-time/#timeorigin-attribute) 시점을 기준으로 표시 영역 내에서 볼 수 [있는 가장 큰 이미지 또는 텍스트 블록](#what-elements-are-considered) 의 렌더링 시간을 보고합니다.

<picture>
  <source srcset="{{ " image imgix media="(min-width: 640px)" width="400" height="100">{% Img src="image/eqprBhZUGfb8WYnumQ9ljAxRrA72/8ZW8LQsagLih1ZZoOmMR.svg", alt="Good LCP values are 2.5 seconds, poor values are greater than 4.0 seconds and anything in between needs improvement", width="400", height="300", class="w-screenshot w-screenshot--filled width-full" %}</source></picture>

### 좋은 LCP 점수는 무엇입니까?

**좋은 사용자 경험을 제공하기 위해 사이트는 2.5초** 이하의 콘텐츠가 포함된 최대 페인트를 갖도록 노력해야 합니다. 대부분의 사용자가 이 목표를 달성할 수 있도록 모바일 및 데스크톱 장치에 걸쳐 분류된 페이지 로드 **의 75번째 백분위수를 측정하는 것이 좋습니다.**

{% Aside %} 이 권장사항의 이면에 있는 연구 및 방법론에 대해 자세히 알아보려면 [핵심 성능 향상 측정항목 임계값 정의](/defining-core-web-vitals-thresholds/) {% endAside %}를 참조하세요.

### 어떤 요소가 고려됩니까?

[현재 Largest Contentful Paint API 에](https://wicg.github.io/largest-contentful-paint/) 지정된 대로 Largest Contentful Paint에 대해 고려되는 요소 유형은 다음과 같습니다.

- `<img>` 요소
- `<svg>` 요소 내부의 `<image>`
- `<video>` 요소(포스터 이미지 사용)
- [`url()`](https://developer.mozilla.org/en-US/docs/Web/CSS/url()) 함수를 통해 로드된 배경 이미지가 있는 요소 [(CSS 그래디언트](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Images/Using_CSS_gradients) 와 반대)
- 텍스트 노드 또는 기타 인라인 수준 텍스트 요소 자식을 포함하는 [블록 수준 요소입니다.](https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements)

이 제한된 집합으로 요소를 제한하는 것은 처음부터 일을 단순하게 유지하기 위한 의도였습니다. 추가 요소(예: `<svg>` , `<video>` )는 향후 더 많은 연구가 수행됨에 따라 추가될 수 있습니다.

### 요소의 크기는 어떻게 결정됩니까?

가장 큰 콘텐츠가 포함된 페인트에 대해 보고된 요소의 크기는 일반적으로 뷰포트 내에서 사용자가 볼 수 있는 크기입니다. 요소가 뷰포트 외부로 확장되거나 요소가 잘리거나 보이지 않는 [오버플로가 있는](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow) 경우 해당 부분은 요소 크기에 포함되지 않습니다.

[고유 크기](https://developer.mozilla.org/en-US/docs/Glossary/Intrinsic_Size) 에서 크기가 조정된 이미지 요소의 경우 보고되는 크기는 가시적 크기 또는 고유 크기 중 더 작은 것입니다. 예를 들어, 기본 크기보다 훨씬 작게 축소된 이미지는 표시되는 크기만 보고하는 반면, 더 큰 크기로 늘이거나 확장된 이미지는 고유 크기만 보고합니다.

텍스트 요소의 경우 텍스트 노드의 크기만 고려됩니다(모든 텍스트 노드를 포함하는 가장 작은 직사각형).

모든 요소에 대해 CSS를 통해 적용된 여백, 패딩 또는 테두리는 고려되지 않습니다.

{% Aside %} 어떤 텍스트 노드가 어떤 요소에 속하는지 결정하는 것은 때때로 까다로울 수 있습니다. 특히 자식이 인라인 요소와 일반 텍스트 노드를 포함하지만 블록 수준 요소도 포함하는 요소의 경우에는 더욱 그렇습니다. 요점은 모든 텍스트 노드가 가장 가까운 블록 수준 조상 요소에 속한다는 것입니다. [사양 측면에서](https://wicg.github.io/element-timing/#set-of-owned-text-nodes) [: 각 텍스트 노드는 포함하는 블록](https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block) 을 생성하는 요소에 속합니다. {% endAside %}

### 가장 많이 함유된 페인트는 언제 보고됩니까?

웹 페이지는 종종 단계적으로 로드되며 결과적으로 페이지에서 가장 큰 요소가 변경될 수 있습니다.

이러한 변경 가능성을 처리하기 위해 브라우저는 브라우저가 첫 번째 프레임을 그리는 즉시 가장 큰 콘텐츠 요소를 식별하는 `largest-contentful-paint` 유형 [`PerformanceEntry`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry) 그러나 후속 프레임을 렌더링한 후 가장 큰 콘텐츠 요소가 변경될 때마다 [`PerformanceEntry`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry)

예를 들어, 텍스트와 영웅 이미지와 페이지에 브라우저는 처음에 단지 텍스트에서 브라우저가 파견 것이라고 지적하는 렌더링 할 수 있습니다 `largest-contentful-paint` 그 항목 `element` 특성 가능성이 참조 할 것 `<p>` 또는 `<h1>` . 나중에 영웅 이미지 로드가 완료되면 두 번째로 `largest-contentful-paint` 항목이 전달되고 해당 `element` 속성이 `<img>` 참조합니다.

요소가 렌더링되고 사용자에게 표시되면 가장 큰 콘텐츠 요소로 간주될 수 있다는 점에 유의하는 것이 중요합니다. 아직 로드되지 않은 이미지는 "렌더링"된 것으로 간주되지 않습니다. [글꼴 차단 기간](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display#The_font_display_timeline) 동안 웹 글꼴을 사용하는 텍스트 노드도 마찬가지입니다. 이러한 경우 더 작은 요소가 가장 큰 콘텐츠가 있는 요소로 보고될 수 있지만 더 큰 요소가 렌더링을 완료하는 즉시 다른 `PerformanceEntry` 개체를 통해 보고됩니다.

이미지와 글꼴을 늦게 로드하는 것 외에도 페이지는 새 콘텐츠를 사용할 수 있게 되면 DOM에 새 요소를 추가할 수 있습니다. 이러한 새 요소 중 하나가 이전의 가장 큰 콘텐츠 요소보다 큰 경우 새 `PerformanceEntry` 도 보고됩니다.

현재 콘텐츠가 포함된 가장 큰 요소인 요소가 뷰포트에서 제거(또는 DOM에서 제거)되더라도 더 큰 요소가 렌더링되지 않는 한 콘텐츠가 포함된 가장 큰 요소로 유지됩니다.

{% Aside %} Chrome 88 이전에는 제거된 요소가 가장 큰 콘텐츠가 있는 요소로 간주되지 않았으며 현재 후보를 제거하면 새로운 `largest-contentful-paint` 항목이 전달되도록 트리거됩니다. 그러나 DOM 요소를 자주 제거하는 이미지 캐러셀과 같은 인기 있는 UI 패턴으로 인해 사용자 경험을 보다 정확하게 반영하도록 측정항목이 업데이트되었습니다. 자세한 내용은 [CHANGELOG](https://chromium.googlesource.com/chromium/src/+/master/docs/speed/metrics_changelog/2020_11_lcp.md) 를 참조하십시오. {% endAside %}

브라우저는 사용자가 페이지와 상호작용(탭, 스크롤 또는 키 누르기를 통해)하는 즉시 새 항목 보고를 중지합니다. 사용자 상호작용은 종종 사용자에게 표시되는 내용을 변경하기 때문입니다(특히 스크롤링의 경우).

분석을 위해 가장 최근에 발송된 `PerformanceEntry` 만 분석 서비스에 보고해야 합니다.

{% Aside 'caution' %} 사용자는 배경 탭에서 페이지를 열 수 있으므로 사용자가 탭을 처음 로드했을 때보다 훨씬 늦게 탭에 초점을 맞출 때까지 가장 큰 콘텐츠가 포함된 페인트가 발생하지 않을 수 있습니다. {% endAside %}

#### 로드 시간 대 렌더링 시간

[`Timing-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin) 헤더가 없는 교차 출처 이미지의 경우 이미지의 렌더링 타임스탬프가 노출되지 않습니다. 대신 로드 시간만 노출됩니다(다른 많은 웹 API를 통해 이미 노출되어 있기 때문에).

[아래 사용 예](#measure-lcp-in-javascript) 는 렌더링 시간을 사용할 수 없는 요소를 처리하는 방법을 보여줍니다. [`Timing-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin) 헤더를 설정하는 것이 좋습니다. 그러면 메트릭이 더 정확해집니다.

### 요소 레이아웃 및 크기 변경은 어떻게 처리됩니까?

새 성능 항목을 계산하고 디스패치하는 성능 오버헤드를 낮게 유지하기 위해 요소의 크기나 위치를 변경해도 새 LCP 후보가 생성되지 않습니다. 뷰포트에서 요소의 초기 크기와 위치만 고려됩니다.

즉, 처음에 화면 밖에서 렌더링된 다음 화면에서 전환되는 이미지는 보고되지 않을 수 있습니다. 또한 처음에 뷰포트에서 렌더링된 요소가 아래로 밀려 나와 뷰에서 벗어나면 여전히 초기 뷰포트 내 크기를 보고함을 의미합니다.

### 예

다음은 몇 가지 인기 있는 웹사이트에서 가장 큰 콘텐츠가 포함된 페인트가 발생하는 몇 가지 예입니다.

{% Img src="image/admin/bsBm8poY1uQbq7mNvVJm.png", alt="Largest Contentful Paint timeline from cnn.com", width="800", height="311" %}

{% Img src="image/admin/xAvLL1u2KFRaqoZZiI71.png", alt="Largest Contentful Paint timeline from techcrunch.com", width="800", height="311" %}

위의 두 타임라인에서 콘텐츠가 로드되면 가장 큰 요소가 변경됩니다. 첫 번째 예에서 새 콘텐츠가 DOM에 추가되고 가장 큰 요소가 변경됩니다. 두 번째 예에서는 레이아웃이 변경되고 이전에 가장 컸던 콘텐츠가 뷰포트에서 제거됩니다.

늦게 로드하는 콘텐츠가 페이지에 이미 있는 콘텐츠보다 큰 경우가 많지만 반드시 그런 것은 아닙니다. 다음 두 예는 페이지가 완전히 로드되기 전에 발생하는 가장 큰 콘텐츠가 포함된 페인트를 보여줍니다.

{% Img src="image/admin/uJAGswhXK3bE6Vs4I5bP.png", alt="Largest Contentful Paint timeline from instagram.com", width="800", height="311" %}

{% Img src="image/admin/e0O2woQjZJ92aYlPOJzT.png", alt="Largest Contentful Paint timeline from google.com", width="800", height="311" %}

첫 번째 예에서는 Instagram 로고가 비교적 일찍 로드되어 다른 콘텐츠가 점진적으로 표시되더라도 가장 큰 요소로 남아 있습니다. Google 검색 결과 페이지 예에서 가장 큰 요소는 이미지 또는 로고 로드가 완료되기 전에 표시되는 텍스트 단락입니다. 모든 개별 이미지가 이 단락보다 작기 때문에 로드 프로세스 전체에서 가장 큰 요소로 유지됩니다.

{% Aside %} Instagram 타임라인의 첫 번째 프레임에서 카메라 로고 주위에 녹색 상자가 없음을 알 수 있습니다. 이는 `<svg>` 요소이고 `<svg>` 요소는 현재 LCP 후보로 간주되지 않기 때문입니다. 첫 번째 LCP 후보는 두 번째 프레임의 텍스트입니다. {% endAside %}

## LCP 측정 방법

[LCP는 실험실](/user-centric-performance-metrics/#in-the-lab) 이나 [현장에서](/user-centric-performance-metrics/#in-the-field) 측정할 수 있으며 다음 도구에서 사용할 수 있습니다.

### 현장 도구

- [Chrome 사용자 경험 보고서](https://developers.google.com/web/tools/chrome-user-experience-report)
- [PageSpeed 인사이트](https://developers.google.com/speed/pagespeed/insights/)
- [Search Console(핵심 Web Vitals 보고서)](https://support.google.com/webmasters/answer/9205520)
- [`web-vitals` JavaScript 라이브러리](https://github.com/GoogleChrome/web-vitals)

### 실험 도구

- [Chrome 개발자 도구](https://developers.google.com/web/tools/chrome-devtools/)
- [등대](https://developers.google.com/web/tools/lighthouse/)
- [웹페이지 테스트](https://webpagetest.org/)

### JavaScript에서 LCP 측정

JavaScript에서 LCP를 측정하려면 [Largest Contentful Paint API를](https://wicg.github.io/largest-contentful-paint/) 사용할 수 있습니다. `largest-contentful-paint` 항목을 수신 대기하고 콘솔에 기록 [`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) 를 만드는 방법을 보여줍니다.

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('LCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'largest-contentful-paint', buffered: true});
```

{% Aside 'warning' %}

`largest-contentful-paint` 항목을 콘솔에 기록하는 방법을 보여주지만 JavaScript에서 LCP를 측정하는 것은 더 복잡합니다. 자세한 내용은 아래를 참조하세요.

{% endAside %}

위의 예에서 기록된 각각의 `largest-contentful-paint` 항목은 현재 LCP 후보를 나타냅니다. 일반적으로 내보낸 `startTime` 값은 LCP 값이지만 항상 그런 것은 아닙니다. `largest-contentful-paint` 항목이 LCP 측정에 유효한 것은 아닙니다.

다음 섹션에는 API가 보고하는 내용과 측정항목이 계산되는 방법 간의 차이점이 나열되어 있습니다.

#### 측정항목과 API의 차이점

- API는 `largest-contentful-paint` 항목을 발송하지만 LCP를 계산할 때 해당 페이지를 무시해야 합니다.
- API는 `largest-contentful-paint` 항목을 계속 발송하지만 LCP를 계산할 때 이러한 항목을 무시해야 합니다(요소는 페이지가 전체 시간 포그라운드에 있었던 경우에만 고려될 수 있음).
- [API는 페이지가 뒤로/앞으로 캐시](/bfcache/#impact-on-core-web-vitals) 에서 복원될 때 `largest-contentful-paint` 가 포함된 페인트 항목을 보고하지 않지만 사용자가 개별 페이지 방문으로 경험하기 때문에 이러한 경우 LCP를 측정해야 합니다.
- API는 iframe 내의 요소를 고려하지 않지만 LCP를 적절하게 측정하려면 이를 고려해야 합니다. 하위 프레임은 API를 사용하여 `largest-contentful-paint` 항목을 집계를 위해 상위 프레임에 보고할 수 있습니다.

이러한 미묘한 차이점을 모두 기억하는 대신 개발자는 [`web-vitals` JavaScript 라이브러리](https://github.com/GoogleChrome/web-vitals) 를 사용하여 가능한 경우 이러한 차이점을 처리하는 LCP를 측정할 수 있습니다.

```js
import {getLCP} from 'web-vitals';

// Measure and log LCP as soon as it's available.
getLCP(console.log);
```

JavaScript에서 LCP를 측정하는 방법에 대한 전체 예제는 [`getLCP()` 대한 소스 코드를](https://github.com/GoogleChrome/web-vitals/blob/master/src/getLCP.ts) 참조할 수 있습니다.

{% Aside %} 일부 경우(예: 교차 출처 iframe) 자바스크립트에서 LCP를 측정할 수 없습니다. 자세한 내용은 `web-vitals` [라이브러리의 제한 사항](https://github.com/GoogleChrome/web-vitals#limitations) 섹션을 참조하십시오. {% endAside %}

### 가장 큰 요소가 가장 중요하지 않다면 어떻게 될까요?

어떤 경우에는 페이지에서 가장 중요한 요소(또는 요소)가 가장 큰 요소와 같지 않으며 개발자는 대신 이러한 다른 요소의 렌더링 시간을 측정하는 데 더 관심이 있을 수 있습니다. [이것은 사용자 정의 측정항목](/custom-metrics/#element-timing-api) 에 대한 문서에 설명된 대로 [Element Timing API를](https://wicg.github.io/element-timing/) 사용하여 가능합니다.

## LCP를 개선하는 방법

LCP는 주로 4가지 요인에 의해 영향을 받습니다.

- 느린 서버 응답 시간
- 렌더링 차단 JavaScript 및 CSS
- 리소스 로드 시간
- 클라이언트 측 렌더링

LCP를 개선하는 방법에 대한 자세한 내용은 [LCP 최적화 를](/optimize-lcp/) 참조하십시오. LCP도 개선할 수 있는 개별 성능 기술에 대한 추가 지침은 다음을 참조하십시오.

- [PRPL 패턴으로 즉시 로딩 적용](/apply-instant-loading-with-prpl)
- [중요한 렌더링 경로 최적화](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/)
- [CSS 최적화](/fast#optimize-your-css)
- [이미지 최적화](/fast#optimize-your-images)
- [웹 글꼴 최적화](/fast#optimize-web-fonts)
- [JavaScript 최적화](/fast#optimize-your-javascript) (클라이언트 렌더링 사이트용)

## 추가 리소스

- [performance.now()](https://perfnow.nl/) (2019)에서 [Annie Sullivan이](https://anniesullie.com/) [Chrome의 성능 모니터링에서 얻은 교훈](https://youtu.be/ctavZT87syI)

{% include 'content/metrics/metrics-changelog.njk' %}
