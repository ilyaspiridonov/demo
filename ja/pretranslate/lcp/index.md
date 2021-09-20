---
layout: post
title: 最大のコンテンツフルペイント（LCP）
authors:
  - philipwalton
date: '2019-08-08'
updated: '2020-06-17'
description: |2

  この投稿では、最大のコンテンツフルペイント（LCP）メトリックを紹介し、説明します

  それを測定する方法
tags:
  - performance
  - metrics
---

{% Aside %} Largest Contentful Paint（LCP）は、ページのメインコンテンツが読み込まれる可能性が高いページ読み込みタイムラインのポイントをマークするため、[知覚される読み込み速度](/user-centric-performance-metrics/#types-of-metrics)を測定するための重要なユーザー中心の指標です。高速LCPは、ユーザーにページは[便利です](/user-centric-performance-metrics/#questions)。 {% endAside %}

これまで、Web開発者にとって、Webページのメインコンテンツが読み込まれ、ユーザーに表示される速度を測定することは困難でした。

[load](https://developer.mozilla.org/en-US/docs/Web/Events/load)やDOMContentLoadedなどの古いメトリックは、ユーザーが画面に表示するものに必ずしも対応していないため、 [適切ではありません。](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded)[また、First Contentful Paint（FCP）の](/fcp/)ような新しい、ユーザー中心のパフォーマンスメトリックは、読み込みエクスペリエンスの最初の部分のみをキャプチャします。ページにスプラッシュ画面が表示されたり、読み込みインジケーターが表示されたりする場合、この瞬間はユーザーにはあまり関係ありません。

過去には、 [First Meaningful Paint（FMP）](/first-meaningful-paint/)[やSpeed Index（SI）](/speed-index/) （どちらもLighthouseで利用可能）などのパフォーマンスメトリックを推奨して、最初のペイント後の読み込みエクスペリエンスをより多くキャプチャできるようにしましたが、これらのメトリックは複雑で説明が困難です、そしてしばしば間違っています—ページのメインコンテンツがいつロードされたかをまだ識別しないことを意味します。

単純な方が良い場合もあります。 [W3C Webパフォーマンスワーキンググループ](https://www.w3.org/webperf/)での議論とGoogleで行われた調査に基づいて、ページのメインコンテンツがいつ読み込まれるかを測定するより正確な方法は、最大の要素がいつレンダリングされたかを調べることであることがわかりました。

## LCPとは何ですか？

Largest Contentful Paint（LCP）メトリックは[、ページが最初にロードを開始](https://w3c.github.io/hr-time/#timeorigin-attribute)したときと比較して、ビューポート内に表示される[最大の画像またはテキストブロック](#what-elements-are-considered)のレンダリング時間を報告します。

<picture>
  <source srcset="{{ " image imgix media="(min-width: 640px)" width="400" height="100">{% Img src="image/eqprBhZUGfb8WYnumQ9ljAxRrA72/8ZW8LQsagLih1ZZoOmMR.svg", alt="Good LCP values are 2.5 seconds, poor values are greater than 4.0 seconds and anything in between needs improvement", width="400", height="300", class="w-screenshot w-screenshot--filled width-full" %}</source></picture>

### 良いLCPスコアとは何ですか？

**優れたユーザーエクスペリエンスを提供するために、サイトは2.5秒**以下の最大コンテンツフルペイントを使用するように努める必要があります。ほとんどのユーザーでこの目標を確実に達成するために、測定するのに適したしきい値は、モバイルデバイスとデスクトップデバイス間でセグメント化されたページ読み込みの**75パーセンタイルです。**

{% Aside %}この推奨事項の背後にある調査と方法論の詳細については、「コアWebVitals[メトリックしきい値の定義{% endAside %}」を参照してください。](/defining-core-web-vitals-thresholds/)

### どの要素が考慮されますか？

[Largest Contentful Paint API](https://wicg.github.io/largest-contentful-paint/)で現在指定されているように、Largest ContentfulPaintで考慮される要素のタイプは次のとおりです。

- `<img>`要素
- `<svg>` &gt;要素内の`<image>`
- `<video>`要素（ポスター画像が使用されます）
- [（CSSグラデーションで](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Images/Using_CSS_gradients)はなく[`url()`](https://developer.mozilla.org/en-US/docs/Web/CSS/url())関数を介してロードされた背景画像を持つ要素
- テキストノードまたは他のインラインレベルのテキスト要素の子を含む[ブロックレベルの要素。](https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements)

要素をこの限定されたセットに制限することは、最初は物事を単純にするために意図的なものであることに注意してください。将来、さらに調査が行われるにつれて、追加の要素（ `<svg>` 、 `<video>` ）が追加される可能性があります。

### 要素のサイズはどのように決定されますか？

最大コンテンツフルペイントについて報告される要素のサイズは、通常、ビューポート内でユーザーに表示されるサイズです。要素がビューポートの外側に伸びている場合、または要素のいずれかがクリップされているか[、目に見えないオーバーフローがある](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow)場合、それらの部分は要素のサイズにカウントされません。

[固有のサイズ](https://developer.mozilla.org/en-US/docs/Glossary/Intrinsic_Size)からサイズ変更された画像要素の場合、報告されるサイズは、表示サイズまたは固有サイズのいずれか小さい方です。たとえば、本来のサイズよりもはるかに小さいサイズに縮小された画像は、表示されているサイズのみを報告しますが、拡大または拡大された画像は、本来のサイズのみを報告します。

テキスト要素の場合、テキストノードのサイズのみが考慮されます（すべてのテキストノードを含む最小の長方形）。

すべての要素について、CSSを介して適用されるマージン、パディング、または境界線は考慮されません。

{% Aside %}どのテキストノードがどの要素に属しているかを判断するのは難しい場合があります。特に、子にインライン要素とプレーンテキストノードだけでなくブロックレベルの要素も含まれている要素の場合はなおさらです。重要な点は、すべてのテキストノードが最も近いブロックレベルの祖先要素に属している（そして属しているだけである）ということです。[仕様では、](https://wicg.github.io/element-timing/#set-of-owned-text-nodes)各テキストノードは、それ[を含むブロック](https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block)を生成する要素に属します。 {% endAside %}

### 最大の満足のいくペイントはいつ報告されますか？

多くの場合、Webページは段階的に読み込まれるため、ページの最大の要素が変更される可能性があります。

この変更の可能性を処理するために、ブラウザーは、ブラウザーが最初のフレームをペイントするとすぐに、最大のコンテンツフル要素を識別するタイプ`largest-contentful-paint` [`PerformanceEntry`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry)ただし、後続のフレームをレンダリングした後、最大のコンテンツ要素が変更されるたびに、 [`PerformanceEntry`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry)

たとえば、テキストとヒーロー画像を含むページでは、ブラウザは最初にテキストをレンダリングするだけです。その時点で、ブラウザは`element` `<p>`または`<h1>`参照する可能性が高い`largest-contentful-paint`エントリをディスパッチします。その後、ヒーロー画像の読み込みが`largest-contentful-paint`エントリがディスパッチされ、その`element`プロパティが`<img>`参照します。

要素は、レンダリングされてユーザーに表示された後でのみ、最大のコンテンツコンテンツ要素と見なすことができることに注意することが重要です。まだロードされていない画像は「レンダリング済み」とは見なされません。 [フォントブロック期間](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display#The_font_display_timeline)中にWebフォントを使用するテキストノードもありません。このような場合、小さい要素が最大のコンテンツコンテンツ要素として報告されることがありますが、大きい要素がレンダリングを終了するとすぐに、別の`PerformanceEntry`オブジェクトを介して報告されます。

画像やフォントの読み込みが遅れるだけでなく、新しいコンテンツが利用可能になると、ページによってDOMに新しい要素が追加される場合があります。これらの新しい要素のいずれかが以前の最大のコンテンツコンテンツ要素よりも大きい場合、新しい`PerformanceEntry`も報告されます。

現在最大のコンテンツコンテンツ要素である要素がビューポートから削除された場合（またはDOMから削除された場合）、より大きな要素がレンダリングされない限り、その要素は最大のコンテンツコンテンツ要素のままになります。

{% Aside %} Chrome 88より前は、削除された要素は最大のコンテンツコンテンツ要素とは見なされず、現在の候補を削除すると、新しい`largest-contentful-paint`コンテンツフルペイントエントリがディスパッチされていました。ただし、DOM要素を削除することが多い画像カルーセルなどの一般的なUIパターンのため、ユーザーのエクスペリエンスをより正確に反映するようにメトリックが更新されました。詳細については、 [CHANGELOG](https://chromium.googlesource.com/chromium/src/+/master/docs/speed/metrics_changelog/2020_11_lcp.md)を参照してください。 {% endAside %}

ユーザーがページを操作すると（タップ、スクロール、またはキーを押すと）、ブラウザーは新しいエントリの報告を停止します。これは、ユーザーの操作によってユーザーに表示される内容が変わることが多いためです（これは特にスクロールの場合に当てはまります）。

分析の目的で、最後にディスパッチされた`PerformanceEntry`のみを分析サービスに報告する必要があります。

{% Aside 'caution' %}ユーザーはバックグラウンドタブでページを開くことができるため、ユーザーがタブにフォーカスするまで、最大のコンテンツペイントが発生しない可能性があります。これは、最初にタブをロードしたときよりもはるかに遅くなる可能性があります。 {% endAside %}

#### ロード時間とレンダリング時間

[`Timing-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin)ヘッダーがないクロスオリジン画像の画像のレンダリングタイムスタンプは公開されません。代わりに、ロード時間のみが公開されます（これは、他の多くのWeb APIを介してすでに公開されているため）。

以下の[使用例](#measure-lcp-in-javascript)は、レンダリング時間が利用できない要素を処理する方法を示しています。 [`Timing-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin)ヘッダーを設定することを常にお勧めします。これにより、メトリックがより正確になります。

### 要素のレイアウトとサイズの変更はどのように処理されますか？

新しいパフォーマンスエントリの計算とディスパッチのパフォーマンスオーバーヘッドを低く抑えるために、要素のサイズまたは位置を変更しても、新しいLCP候補は生成されません。ビューポート内の要素の初期サイズと位置のみが考慮されます。

これは、最初に画面外にレンダリングされ、次に画面上で遷移する画像が報告されない場合があることを意味します。また、ビューポートで最初にレンダリングされた要素が押し下げられ、ビュー外に表示されても、ビューポート内の初期サイズが報告されることも意味します。

### 例

最大のコンテンツフルペイントがいくつかの人気のあるWebサイトで発生する場合の例を次に示します。

{% Img src="image/admin/bsBm8poY1uQbq7mNvVJm.png", alt="Largest Contentful Paint timeline from cnn.com", width="800", height="311" %}

{% Img src="image/admin/xAvLL1u2KFRaqoZZiI71.png", alt="Largest Contentful Paint timeline from techcrunch.com", width="800", height="311" %}

上記の両方のタイムラインで、最大の要素はコンテンツの読み込みに応じて変化します。最初の例では、新しいコンテンツがDOMに追加され、最大の要素が変更されます。 2番目の例では、レイアウトが変更され、以前は最大だったコンテンツがビューポートから削除されます。

読み込みが遅いコンテンツがすでにページにあるコンテンツよりも大きい場合がよくありますが、必ずしもそうとは限りません。次の2つの例は、ページが完全に読み込まれる前に発生する最大のコンテンツフルペイントを示しています。

{% Img src="image/admin/uJAGswhXK3bE6Vs4I5bP.png", alt="Largest Contentful Paint timeline from instagram.com", width="800", height="311" %}

{% Img src="image/admin/e0O2woQjZJ92aYlPOJzT.png", alt="Largest Contentful Paint timeline from google.com", width="800", height="311" %}

最初の例では、Instagramのロゴは比較的早く読み込まれ、他のコンテンツが徐々に表示されても最大の要素のままです。 Google検索結果ページの例では、最大の要素は、画像またはロゴの読み込みが完了する前に表示されるテキストの段落です。個々の画像はすべてこの段落よりも小さいため、ロードプロセス全体で最大の要素のままです。

{% Aside %} Instagramタイムラインの最初のフレームで、カメラのロゴの周りに緑色のボックスがないことに気付くかもしれません。これは、これが`<svg>`要素であり、 `<svg>`要素は現在LCP候補とは見なされていないためです。最初のLCP候補は、2番目のフレームのテキストです。 {% endAside %}

## LCPの測定方法

LCPは[、ラボ](/user-centric-performance-metrics/#in-the-lab)または[フィールド](/user-centric-performance-metrics/#in-the-field)で測定でき、次のツールで利用できます。

### フィールドツール

- [Chromeユーザーエクスペリエンスレポート](https://developers.google.com/web/tools/chrome-user-experience-report)
- [PageSpeedインサイト](https://developers.google.com/speed/pagespeed/insights/)
- [検索コンソール（コアWeb Vitalsレポート）](https://support.google.com/webmasters/answer/9205520)
- [`web-vitals`ライブラリ](https://github.com/GoogleChrome/web-vitals)

### ラボツール

- [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/)
- [灯台](https://developers.google.com/web/tools/lighthouse/)
- [WebPageTest](https://webpagetest.org/)

### JavaScriptでLCPを測定する

JavaScriptでLCPを測定するには、 [Largest Contentful PaintAPIを](https://wicg.github.io/largest-contentful-paint/)使用できます。 `largest-contentful-paint`エントリをリッスンしてコンソールに記録する[`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver)を作成する方法を示しています。

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('LCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'largest-contentful-paint', buffered: true});
```

{% Aside 'warning' %}

`largest-contentful-paint`エントリをコンソールに記録する方法を示していますが、JavaScriptでのLCPの測定はより複雑です。詳細については、以下を参照してください。

{% endAside %}

上記の例では、ログに記録された`largest-contentful-paint`エントリは、現在のLCP候補を表しています。一般`startTime`されたエントリのstartTime値はLCP値ですが、常にそうであるとは限りません。 `largest-contentful-paint`エントリがLCPの測定に有効であるとは限りません。

次のセクションでは、APIが報告する内容とメトリックの計算方法の違いを示します。

#### メトリックとAPIの違い

- APIは`largest-contentful-paint`エントリをディスパッチしますが、LCPを計算するときはそれらのページを無視する必要があります。
- `largest-contentful-paint`エントリをディスパッチし続けますが、LCPを計算するときは、これらのエントリを無視する必要があります（要素は、ページが常にフォアグラウンドにある場合にのみ考慮されます）。
- [APIは、ページがバック/フォワードキャッシュ](/bfcache/#impact-on-core-web-vitals)から復元されたときに、 `largest-contentful-paint`エントリを報告しませんが、ユーザーは個別のページアクセスとしてLCPを経験するため、これらの場合はLCPを測定する必要があります。
- APIはiframe内の要素を考慮しませんが、LCPを適切に測定するには、それらを考慮する必要があります。サブフレームはAPIを使用して、 `largest-contentful-paint`エントリを親フレームに報告して集計できます。

開発者は、これらの微妙な違いをすべて記憶するのではなく、 [`web-vitals` JavaScriptライブラリ](https://github.com/GoogleChrome/web-vitals)を使用してLCPを測定できます。これにより、（可能な場合）これらの違いが処理されます。

```js
import {getLCP} from 'web-vitals';

// Measure and log LCP as soon as it's available.
getLCP(console.log);
```

JavaScriptでLCPを測定する方法の完全な例については、 [`getLCP()`ソースコードを](https://github.com/GoogleChrome/web-vitals/blob/master/src/getLCP.ts)参照してください。

{% Aside %}場合によっては（クロスオリジンiframeなど）、JavaScriptでLCPを測定できないことがあります。詳細については、 `web-vitals`[ライブラリの制限](https://github.com/GoogleChrome/web-vitals#limitations)セクションを参照してください。 {% endAside %}

### 最大の要素が最も重要でない場合はどうなりますか？

場合によっては、ページ上の最も重要な要素（または複数の要素）が最大の要素と同じではなく、開発者は代わりにこれらの他の要素のレンダリング時間を測定することに関心があるかもしれません。[これは、カスタムメトリック](/custom-metrics/#element-timing-api)に関する記事で説明されているように、 [Element TimingAPI](https://wicg.github.io/element-timing/)を使用して可能です。

## LCPを改善する方法

LCPは、主に次の4つの要因の影響を受けます。

- サーバーの応答時間が遅い
- レンダリングをブロックするJavaScriptとCSS
- リソースのロード時間
- クライアント側のレンダリング

LCPを改善する方法の詳細については、「 [LCPの最適化](/optimize-lcp/)」を参照してください。 LCPを改善することもできる個々のパフォーマンス手法に関する追加のガイダンスについては、以下を参照してください。

- [PRPLパターンでインスタントロードを適用する](/apply-instant-loading-with-prpl)
- [重要なレンダリングパスの最適化](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/)
- [CSSを最適化する](/fast#optimize-your-css)
- [画像を最適化する](/fast#optimize-your-images)
- [Webフォントを最適化する](/fast#optimize-web-fonts)
- [JavaScriptを最適化する](/fast#optimize-your-javascript)（クライアントがレンダリングするサイト用）

## 追加のリソース

- [Chromeで監視し、パフォーマンスから学んだ教訓](https://youtu.be/ctavZT87syI)によって[アニー・サリバン](https://anniesullie.com/)で[performance.now（）](https://perfnow.nl/) （2019）

{% include 'content/metrics/metrics-changelog.njk' %}
