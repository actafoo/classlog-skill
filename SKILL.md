---
name: classlog-app
description: ClassLog(교사용 형성평가 플랫폼)에 올릴 수 있는 인터랙티브 HTML 학습앱을 만들거나, 기존 HTML 앱에 ClassLog 연동을 추가할 때 사용하는 규격. 학생별 문항 정오답이 교사 대시보드에 자동 집계된다.
---

# ClassLog 앱 만들기 규격

이 문서는 AI 코딩 도구(Claude, ChatGPT, Cursor, Gemini 등)가 읽고 따르는 규격입니다.
사용자가 "이 규격에 맞춰 학습앱을 만들어줘" 또는 "내 앱에 이 규격을 반영해줘"라고 하면 아래를 그대로 지키세요.

## ClassLog란

교사가 HTML 학습앱(퀴즈·연습문제·학습활동)을 업로드하면, 학생들이 입장코드로 접속해 활동하고,
**누가 제출했는지 / 어떤 문항을 맞고 틀렸는지 / 반 전체가 많이 틀린 문제**가 교사 대시보드에 자동으로 쌓이는 서비스입니다.

앱은 ClassLog 안에서 iframe으로 실행되며, ClassLog가 전역 객체 `window.ClassLog`를 **자동으로 주입**합니다.
앱이 할 일은 아래 API를 호출하는 것뿐입니다. 별도의 서버 통신, 라이브러리, script 태그가 전혀 필요 없습니다.

## 필수 규칙

1. **단일 HTML 파일**로 만드세요. CSS/JS 모두 인라인. (외부 CDN은 가능하지만 최소화)
2. `window.ClassLog`를 직접 정의하거나 덮어쓰지 마세요. ClassLog가 주입합니다.
3. `window.parent.postMessage`를 직접 호출하지 마세요.
4. 앱이 ClassLog 밖(로컬 파일로 열기)에서도 동작하도록 **안전 래퍼**를 쓰세요:

```js
// 파일 상단에 이 래퍼를 넣고, 이후 CL.item / CL.done / CL.setScore 로 호출
const CL = window.ClassLog || { item(){}, setScore(){}, done(){}, progress(){} };
```

## API 계약

### 1) 문항이 채점되는 시점마다 — `CL.item(...)`

학생이 문항 하나에 답해서 정답/오답이 결정되는 순간마다 호출하세요. **이게 가장 중요합니다.**

```js
CL.item({
  id: "q1",              // 문항 고유 id (q1, q2, ... 순서대로)
  text: "3/4 + 1/4 = ?", // 문항 내용 그대로 (교사 대시보드에 표시됨)
  correct: false,         // 정답 여부
  response: "2/4"         // 학생이 낸 답 (선택형이면 고른 보기의 내용)
});
```

- 같은 문항을 다시 풀면 **같은 id로 다시 호출**하세요. 마지막 시도가 기록됩니다.
- `text`는 300자, 문항은 최대 100개까지 기록됩니다.
- 채점이 없는 문항(자유 서술 등)은 `correct`를 생략하고 `response`만 보내도 됩니다.

### 2) 활동이 끝나는 지점에서 — `CL.done()`

```js
CL.done();
```

- `item()`을 호출했다면 **점수는 정답률로 자동 계산**됩니다. 따로 넘기지 마세요.
- 문항 채점이 없는 활동이면 `CL.done({ score: 최종점수 })` 또는 인자 없이 `CL.done()`.
- 명시적인 "끝" 화면(결과 화면)이 있는 앱이라면 그 화면이 뜨는 시점에 호출하세요.

### 3) (선택) 진행 중 점수 표시 — `CL.setScore(점수)`

진행 중 점수가 계속 변하는 앱(연습 드릴 등)이면 점수가 바뀔 때마다 호출하세요.
ClassLog 하단 바에 현재 점수가 표시됩니다.

## 최소 예제

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>분수 덧셈 퀴즈</title></head>
<body>
<div id="quiz"></div>
<script>
const CL = window.ClassLog || { item(){}, setScore(){}, done(){}, progress(){} };

const QUESTIONS = [
  { id: "q1", text: "1/4 + 2/4 = ?", choices: ["2/4", "3/4", "3/8"], answer: 1 },
  { id: "q2", text: "1/2 + 1/2 = ?", choices: ["1", "2/4", "2/2만 정답"], answer: 0 },
];
let i = 0;

function show() {
  if (i >= QUESTIONS.length) { document.getElementById('quiz').textContent = '끝!'; CL.done(); return; }
  const q = QUESTIONS[i];
  document.getElementById('quiz').innerHTML =
    `<p>${q.text}</p>` + q.choices.map((c, k) => `<button onclick="pick(${k})">${c}</button>`).join('');
}
function pick(k) {
  const q = QUESTIONS[i];
  CL.item({ id: q.id, text: q.text, correct: k === q.answer, response: q.choices[k] });
  i++; show();
}
show();
</script>
</body>
</html>
```

## 체크리스트 (완성 전 확인)

- [ ] 단일 HTML 파일인가?
- [ ] 안전 래퍼(`const CL = window.ClassLog || …`)가 있는가?
- [ ] 모든 채점 지점에서 `CL.item({ id, text, correct, response })`를 호출하는가?
- [ ] 활동이 끝나는 지점에서 `CL.done()`을 호출하는가?
- [ ] `window.ClassLog`를 재정의하거나 `postMessage`를 직접 쓰지 않았는가?
- [ ] ClassLog 밖에서 파일을 그냥 열어도 에러 없이 동작하는가?
