---
layout: post
title:  "[Web] 의존성 없는 Node 정적 사이트로 생활 계산기 만들기"
author: Sung Kyungmo
catalog: true
tags: [web, javascript, cloudflare]
comments: true
---
# 오늘계산

연봉 실수령액, 퇴직금, 실업급여, 주휴수당처럼 자주 찾는 생활 계산을 모아 [오늘계산](https://todaycalc.pages.dev/)이라는 사이트를 만들었습니다. 계산기는 35개이고, 회원가입 없이 쓰며 입력한 값은 브라우저 밖으로 보내지 않습니다.

이번 글에서는 사이트를 어떤 구조로 만들었는지 정리해보겠습니다.

# 왜 정적 사이트인가

계산은 전부 브라우저에서 할 수 있어서 서버가 필요 없습니다. 그래서 빌드할 때 HTML을 미리 만들어 두고 [Cloudflare Pages](https://pages.cloudflare.com/)에 올렸습니다. GitHub 저장소에 push하면 Cloudflare가 빌드해서 배포까지 해 주고, 비용은 들지 않습니다.

프레임워크는 쓰지 않았습니다. `package.json`에 의존성이 하나도 없고, Node 20 이상의 ESM만으로 빌드 스크립트를 작성했습니다.

```json
"scripts": {
  "build": "node scripts/build.js",
  "test": "node --import ./test/setup.js --test 'test/**/*.test.js'"
}
```

`scripts/build.js`는 `src/tools`에 있는 계산기 정의를 읽어 페이지마다 HTML을 만들고, `sitemap.xml`과 RSS도 같이 만듭니다. 계산 함수는 순수 함수로 두어서 빌드할 때(Node)와 화면에서 다시 계산할 때(브라우저) 같은 코드를 씁니다. 그래서 정적 HTML에 찍힌 숫자와 사용자가 값을 바꿨을 때 나오는 숫자가 어긋나지 않습니다.

테스트는 Node 내장 테스트 러너(`node --test`)로 작성했습니다. 4대보험·소득세 계산, 페이지 생성 결과, 링크 같은 것을 확인하는 테스트가 750개 정도 있습니다.

# 해가 바뀌면 요율도 바뀐다

연봉 계산기는 국민연금, 건강보험 같은 요율이 매년 바뀝니다. 요율은 연도별로 객체에 넣어 두고, 지금 연도를 한국 시간 기준으로 골라 씁니다.

```javascript
// 한국 시간(UTC+9) 기준 연도. Cloudflare 빌드 서버(UTC)에서도 1월 1일 0시에 바뀐다.
const kstYear = (d) => new Date(d.getTime() + 9 * 3_600_000).getUTCFullYear();

export function ratesYear(date) {
  const y = date ? kstYear(date) : (globalThis.CALC_YEAR ?? kstYear(new Date()));
  return YEARS.filter((x) => x <= y).at(-1) ?? YEARS[0];
}
```

빌드 서버는 UTC로 돌기 때문에 `new Date().getFullYear()`를 그대로 쓰면 1월 1일 오전 9시가 되어서야 연도가 바뀝니다. 그래서 9시간을 더한 뒤 UTC 연도를 읽습니다.

브라우저는 이 함수로 새해가 되면 바로 새 요율로 계산하지만, 정적 HTML에 찍힌 제목과 표는 다시 빌드해야 바뀝니다. 그래서 GitHub Actions에 1월 1일 0시 5분(KST)에 Cloudflare Pages 배포 훅을 부르는 작업을 걸어 두었습니다.

```yaml
on:
  schedule:
    - cron: '5 15 31 12 *' # 1월 1일 00:05 KST
```

# 근로소득 간이세액표 옮기기

월급에서 떼는 소득세는 국세청의 근로소득 간이세액표를 따릅니다. 표가 PDF로만 나와서 `pdftotext -layout`으로 텍스트를 뽑은 다음, 스크립트로 구간과 세액을 읽어 JS 모듈로 저장했습니다.

```
pdftotext -layout 새파일.pdf data/income-tax-table-YYYY.txt
node scripts/extract-tax-table.js data/income-tax-table-YYYY.txt src/lib/taxTable.js
```

표가 개정되면 새 PDF로 같은 명령을 다시 실행하면 됩니다.

# 연봉별 페이지

"연봉 4000 실수령액"처럼 금액을 넣어 검색하는 경우가 많아서, 금액마다 결과를 미리 계산한 페이지도 만들었습니다. [연봉별 실수령액 표](https://todaycalc.pages.dev/salary/table/)와 [월급별 실수령액 표](https://todaycalc.pages.dev/monthly/table/)에서 금액별 페이지로 들어갈 수 있습니다.

# 입력값을 주소에 남기지 않기

계산기끼리 값을 넘길 때는 주소에 값을 붙입니다. 예를 들어 연봉별 페이지에서 계산기로 넘어갈 때 `/salary/#salary=4000`처럼 넘깁니다. 그런데 이 주소가 그대로 남아 있으면 광고나 통계 스크립트가 페이지 주소를 수집할 때 연봉도 같이 넘어갈 수 있습니다.

그래서 `<head>`에 넣은 인라인 스크립트가 광고 스크립트보다 먼저 값을 읽어 `window.__prefill`에 담고, `history.replaceState`로 주소창에서 값을 지웁니다. 계산기 화면은 주소 대신 `window.__prefill`에서 값을 읽습니다.

# 만든 계산기

자주 쓰는 계산기 몇 개를 적어 둡니다.

- [연봉 실수령액 계산기](https://todaycalc.pages.dev/salary/)
- [퇴직금 계산기](https://todaycalc.pages.dev/severance/)
- [실업급여 계산기](https://todaycalc.pages.dev/unemployment/)
- [연말정산 계산기](https://todaycalc.pages.dev/year-end-tax/)
- [주휴수당 계산기](https://todaycalc.pages.dev/weekly-holiday-pay/)
- [연차 계산기](https://todaycalc.pages.dev/annual-leave/)
- [전체 계산기 목록](https://todaycalc.pages.dev/all/)

계산 근거가 되는 법령과 고시는 계산기마다 아래쪽에 적어 두었습니다. 틀린 곳이 있으면 알려 주세요.
