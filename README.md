# 웹 최적화 성능 보고

## Core Web Vitals

웹 성능을 최적화하여 사용자 경험을 향상시키는 것은 중요합니다. 지금 소개드리는 `Web Vitals`는 구글에서 제공하는 사용자에게 우수한 환경을 제공하는데 필수적인 품질 체크 요소 가이드라고 할 수 있습니다.

## 대표적인 지표 항목

  ![스크린샷 2024-08-14 오후 7 23 50 복사본](https://github.com/user-attachments/assets/cba35714-cd33-4c31-abac-b59e1f7d2a4c)


### 최대 콘텐츠 페인트 (LCP): 페이지가 로드될 때부터 가장 큰 텍스트 블록 또는 이미지 요소가 화면에 렌더링되는 시점까지의 시간을 측정

### 다음 페인트에 대한 상호작용 (INP): 페이지에서 이루어지는 모든 탭, 클릭 또는 키보드 상호작용의 지연 시간을 측정

### 레이아웃 변경 횟수 (CLS): 페이지가 로드되는 시점에 예상치 못한 레이아웃 변경의 누적 점수를 측정

<br/>

## 현재 프로젝트를 https://pagespeed.web.dev/ 에서 분석한 결과

![스크린샷 2024-08-14 오후 7 51 11](https://github.com/user-attachments/assets/618a99fd-2b0a-4ded-95c5-e7307258ca6f)



<br/>

## 개선하기

### FCP, LCP 개선

#### 이미지 형식(webp, avif)

<img width="951" alt="스크린샷 2024-08-12 오후 9 32 55" src="https://github.com/user-attachments/assets/1dbc55bc-e7da-4997-891e-9abe3a332ab6">
<img width="948" alt="스크린샷 2024-08-12 오후 9 33 43" src="https://github.com/user-attachments/assets/a7662173-fb15-4426-9bdf-fd49de997025">

<br/>

![스크린샷 2024-08-14 오후 8 49 52](https://github.com/user-attachments/assets/6bc4e41b-3e22-4268-bfba-79d94e8a2c9e)

> 2524Kb(jpg) -> 1406Kb(webp) -> 1037Kb(webp)  jpg에서 avif로 약 58%감축
>
> 실제 빠른 4G환경에서의 네트워크 탭
> ![스크린샷 2024-08-14 오후 9 22 20](https://github.com/user-attachments/assets/181b5769-204d-429b-ad2d-24f96761f6bb)


<br/>
<br/>


#### lazy loading
<img width="946" alt="스크린샷 2024-08-12 오후 9 40 36" src="https://github.com/user-attachments/assets/185ba1f0-a87e-4115-b575-da2a8b765437">

```html
<img loading="lazy" src="image.jpg" alt="..." />
<iframe loading="lazy" src="video-player.html" title="..."></iframe>
```
`loading속성으로 lazy`를 부여하여 지연 로딩 처리

<br/>

#### 렌더링 차단 리소스 제거

<img width="952" alt="스크린샷 2024-08-12 오후 9 41 06" src="https://github.com/user-attachments/assets/6f36db4c-425d-4e4e-836c-5d843f512599">

- font 프로젝트 내부에서 호스팅
![스크린샷 2024-08-14 오후 10 32 17](https://github.com/user-attachments/assets/b3f30c4b-cdc4-4033-899f-42f6f0b7740b)


- script 비동기 처리
```html
    <script
      defer
      type="text/javascript"
      src="//www.freeprivacypolicy.com/public/cookie-consent/4.1.0/cookie-consent.js"
      charset="UTF-8"
    ></script>
    <script defer type="text/javascript" charset="UTF-8">
      // Dom이 로드 된 이후에 실행해주기
      document.addEventListener("DOMContentLoaded", function () {
        cookieconsent.run({
          notice_banner_type: "simple",
          consent_type: "express",
          palette: "light",
          language: "en",
          page_load_consent_levels: ["strictly-necessary"],
          notice_banner_reject_button_hide: false,
          preferences_center_close_button_hide: false,
          page_refresh_confirmation_buttons: false,
          website_name: "Performance Course",
        });
      });
    </script>
```

<br/>

### CLS 개선

#### img태그 width/height 속성 & 반응형 이미지 제공
<img width="948" alt="스크린샷 2024-08-12 오후 9 39 31" src="https://github.com/user-attachments/assets/b708548b-e619-4cf4-8158-dff69c02a882">

```html
<picture>
  <source
    media="(max-width: 575px)"
    width="575"
    height="575"
    srcset="images/Hero_Mobile.avif"
  />
  <source
    media="(min-width: 576px) and (max-width: 960px)"
    width="960"
    height="770"
    srcset="images/Hero_Tablet.avif"
  />
  <img
    src="images/Hero_Desktop.avif"
    width="960"
    height="560"
    alt="Hero"
  />
</picture>
```
`img태그에 width & height속성 부여 -> 미리 설정하여 layout shifting방지`
`반응형으로 이미지 제공 -> 뷰 포트에 맞는 이미지만 로드`

<br/>


#### fallback UI 제공
- AS-IS
![layout shifting before](https://github.com/user-attachments/assets/f65e3dc9-1a10-43c8-bc13-1184bdd1a463)
- TO-BE
![layout shifting after](https://github.com/user-attachments/assets/14f064a4-658f-447c-afdb-fd100724339a)
`fallback UI 제공 -> layout shifting 방지`

<br/>


#### 실제 상호작용이 일어날 때 스크립트 실행처리

프로젝트 시나리오 : 뷰포트에서 스크롤을 내려서 product section이 보일 때 loadProducts실행 및 무거운연산 처리하기
```javascript
window.onload = () => {
  let productSection = document.querySelector("#all-products");
  let status = "idle";

  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting && status === "idle") {
        status = "loading";
        loadProducts().then(() => {
          status = "idle";
        });
        // Simulate heavy operation. It could be a complex price calculation.
        for (let i = 0; i < 10000000; i++) {
          const temp = Math.sqrt(i) * Math.sqrt(i);
        }
      }
    });
  });

  observer.observe(productSection);
};
```
`scroll이벤트 방식은 scroll이 발생할 때마다 이벤트가 발생하는 문제가 있음 -> Intersection Observer API를 활용하여 특정 지점을 교차할 때 이벤트 발생시키기`


### 접근성 및 검색엔진 최적화 하기

#### 이미지에 alt속성 부여
alt속성을 부여한 장점
- 이미지 로드 실패 시 대체 텍스트를 제공.
- 검색 엔진에 최적화 할 수 있음.
- 스크린 리더기에 의미를 제공.

#### meta tag description속성 부여
```javascript
<meta name="description" content="hanghae-plus" />
```

#### 제목요소를 내림차순으로 표시하기
```html
<h1>Article Title</h1>
  <section>
    <h2>Introduction</h2>
    <p>...</p>
  </section>
  <section>
    <h2>Overview</h2>
    <section>
      <h3>Benefits</h3>
      <p>...</p>
      <section>
        <h4>Case Study</h4>
        <p>...</p>
      </section>
      <section>
        <h4>Statistics</h4>
        <p>...</p>
      </section>
    </section>
    <section>
      <h3>Conclusion</h3>
      <p>...</p>
    </section>
  </section>
```


## TO-BE

![최종](https://github.com/user-attachments/assets/a2d46820-c8cd-4738-8ab4-9ad7bd702c07)
