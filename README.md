
# ClassLog 학습앱 스킬

이 문서는 AI 코딩 도구(Claude, ChatGPT, Gemini, Cursor 등)가 읽고 따르는 작업 지침입니다.
사용자는 대부분 **코딩을 모르는 교사**입니다. 기술 용어 없이 대화하고, 아래 절차를 그대로 지키세요.

## ClassLog란

교사가 HTML 학습앱(퀴즈·연습문제·학습활동)을 올리면, 학생들이 참여 코드로 접속해 활동하고,
**누가 제출했는지 / 어떤 문항을 맞고 틀렸는지 / 반 전체가 많이 틀린 문제**가 교사 대시보드에 자동으로 쌓이는 서비스입니다.

앱은 ClassLog 안에서 iframe으로 실행되며, ClassLog가 전역 객체 `window.ClassLog`를 **자동으로 주입**합니다.
앱이 할 일은 아래 API를 호출하는 것뿐입니다. 별도의 서버 통신, 라이브러리, script 태그가 전혀 필요 없습니다.

---

## 먼저: 어떤 상황인지 판단하세요

- 사용자가 **HTML 파일/코드를 주면서** "ClassLog에 맞춰줘 / 이 규격을 반영해줘"라고 하면 → **워크플로우 B (기존 앱 연동)**
- 사용자가 **새 학습앱을 만들어달라**고 하면 → **워크플로우 A (새로 만들기)**
- 애매하면 물어보세요: "이미 만들어둔 HTML이 있나요, 아니면 새로 만들까요?"

---

## 워크플로우 A — 새로 만들기: 코드보다 질문이 먼저

요청이 구체적이지 않다면 **코드를 쓰기 전에** 아래를 한 번에 묶어 질문하세요.
이미 답이 나온 항목은 다시 묻지 않습니다.

1. **무엇을 배우는 활동인가요?** — 학년, 과목, 학습 주제 (예: 초3 수학, 분수의 덧셈)
2. **문항 재료가 있나요?** — 수업 자료, 교과서 단원, 예시 문제, 출제 범위가 있으면 붙여넣어 달라고
   요청하세요. **재료 없이 AI가 문항을 지어내면 교육과정과 어긋날 수 있습니다.** 재료가 없다면:
   만들 문항 목록(문제+정답)을 먼저 보여주고 교사의 확인을 받은 뒤 코드를 만드세요.
3. **몇 문항, 어떤 형식인가요?** — 문항 수, 선택형/단답형/섞기, 난이도
4. **활동이 어떻게 끝나나요?** — 셋 중 하나를 고르게 하세요 (ClassLog 업로드 시 '기록 방식'과 일치해야 합니다):
   - **퀴즈형**: 정해진 문항을 다 풀면 끝, 점수가 나옴
   - **연습형**: 끝없이 반복하는 드릴 (예: 무한 연산 연습)
   - **완료형**: 채점 없는 체험 활동, 끝냈다는 기록만

답을 받은 뒤 제작하세요. 완성 후에는 "ClassLog에 올리면 문항별 정오표가 자동으로 쌓여요"라고 알려주세요.

## 워크플로우 B — 기존 HTML에 연동: 수정보다 분석이 먼저

**1) 분석.** 파일을 읽고 다음을 찾으세요:
- 채점 지점: 정답을 비교하는 코드 (예: `if (userAnswer === correct)`)
- 문항 데이터: 문제 텍스트·정답이 어디에 있는지 (배열, 하드코딩, 동적 생성)
- 점수 변수와 종료 지점 (결과 화면, "끝" 버튼, 또는 끝이 없는 무한 반복)

**2) 판정 보고.** 수정하기 전에 사용자에게 어느 수준으로 연동되는지 알려주세요:

| 파일 상태 | 연동 수준 |
|---|---|
| 문항별 채점 코드가 있음 | ✅ 완전 연동 — 문항별 정오표까지 |
| 점수만 있고 문항 구분이 없음 | ⚠️ 점수 연동 — `setScore`/`done`만 (정오표 없음) |
| 채점이 아예 없음 (체험형) | ⚠️ 완료 기록만 — `done()` 하나 |
| 서버/DB가 필요, 여러 파일로 구성 | ❌ 그대로는 불가 — 단일 HTML로 합치거나 정적으로 바꾸는 작업을 먼저 제안 |

**3) 수술적 수정.** 기존 화면·디자인·게임 로직은 **그대로** 두고, 아래 API 호출만 끼워 넣으세요.
구조를 리팩터링하거나 스타일을 "개선"하지 마세요.
- 문항이 동적/무한 생성되면: 증가 카운터로 고유 id를 만들고(`p1, p2, …`), `text`에는 학생이 본
  문제를 사람이 읽을 수 있는 한 문장으로 조합해 넣으세요 (교사가 대시보드에서 그 문장을 봅니다).
- 재도전이 가능한 앱이면: 같은 문항은 같은 id로 다시 호출 (마지막 시도가 기록됨).

---

## API 계약

### 필수 규칙

1. **단일 HTML 파일.** CSS/JS 모두 인라인. (외부 CDN은 가능하지만 최소화)
2. `window.ClassLog`를 직접 정의하거나 덮어쓰지 마세요. ClassLog가 주입합니다.
3. `window.parent.postMessage`를 직접 호출하지 마세요.
4. 앱이 ClassLog 밖(로컬 파일로 열기)에서도 동작하도록 **안전 래퍼**를 쓰세요.
   반드시 아래처럼 **호출하는 순간에** `window.ClassLog`를 찾는 형태여야 합니다
   (스크립트 로드 시점에 `window.ClassLog || …`로 잡아두는 방식은 주입 순서에 따라 빈 껍데기를 잡을 수 있음):

```js
// 파일 상단에 이 래퍼를 넣고, 이후 CL.item / CL.done / CL.setScore 로 호출
const CL = {
  item:     (x) => window.ClassLog && window.ClassLog.item(x),
  setScore: (s) => window.ClassLog && window.ClassLog.setScore(s),
  done:     (x) => window.ClassLog && window.ClassLog.done(x),
  progress: (x) => window.ClassLog && window.ClassLog.progress(x),
};
```

### 1) 문항이 채점되는 시점마다 — `CL.item(...)`

학생이 문항 하나에 답해서 정답/오답이 결정되는 순간마다 호출하세요. **이게 가장 중요합니다.**

```js
CL.item({
  id: "q1",              // 문항 고유 id (q1, q2, ... / 동적 생성이면 카운터로)
  text: "3/4 + 1/4 = ?", // 문항 내용 그대로 (교사 대시보드에 표시됨)
  correct: false,         // 정답 여부
  response: "2/4"         // 학생이 낸 답 (선택형이면 고른 보기의 내용)
});
```

- `text`는 300자, 문항은 최대 100개까지 기록됩니다.
- 채점이 없는 문항(자유 서술 등)은 `correct`를 생략하고 `response`만 보내도 됩니다.

### 2) 활동이 끝나는 지점에서 — `CL.done()`

```js
CL.done();
```

- `item()`을 호출했다면 **점수는 정답률로 자동 계산**됩니다. 따로 넘기지 마세요.
- 문항 채점이 없는 활동이면 `CL.done({ score: 최종점수 })` 또는 인자 없이 `CL.done()`.
- **끝이 없는 연습형 드릴은 `done()`을 호출할 지점이 없어도 됩니다** — ClassLog가 화면에 항상
  띄우는 "끝내기" 버튼이 대신 마무리하고, 그때까지의 `item()` 기록도 함께 전송됩니다.

### 3) (선택) 진행 중 점수 표시 — `CL.setScore(점수)`

점수가 계속 변하는 앱(연습 드릴 등)이면 점수가 바뀔 때마다 호출하세요.
ClassLog 하단 바에 현재 점수가 표시됩니다.

## 최소 예제

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>분수 덧셈 퀴즈</title></head>
<body>
<div id="quiz"></div>
<script>
const CL = {
  item:     (x) => window.ClassLog && window.ClassLog.item(x),
  setScore: (s) => window.ClassLog && window.ClassLog.setScore(s),
  done:     (x) => window.ClassLog && window.ClassLog.done(x),
  progress: (x) => window.ClassLog && window.ClassLog.progress(x),
};

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

## 완료 전 자가 점검

- [ ] (새로 만들 때) 학년·주제·문항 재료를 **물어봤는가?** 재료 없이 지어낸 문항은 확인받았는가?
- [ ] (기존 앱일 때) 먼저 **분석하고 연동 수준을 보고**했는가? 기존 화면·로직을 건드리지 않았는가?
- [ ] 단일 HTML 파일인가?
- [ ] 안전 래퍼(`const CL = window.ClassLog || …`)가 있는가?
- [ ] 모든 채점 지점에서 `CL.item({ id, text, correct, response })`를 호출하는가?
- [ ] 활동에 끝이 있다면 그 지점에서 `CL.done()`을 호출하는가? (무한 드릴이면 생략 가능)
- [ ] `window.ClassLog` 재정의나 `postMessage` 직접 호출이 없는가?
- [ ] ClassLog 밖에서 파일을 그냥 열어도 에러 없이 동작하는가?
