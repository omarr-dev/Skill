---
name: exam-sprint
description: Turn any course's slides, past exams, and classmates' notes into a fun, mobile-first interactive study app that targets full marks in the least time. Use when a student shares exam material (slides, past papers, summaries, an exam announcement) and wants to study fast, especially "I know nothing and the exam is tomorrow". Works for any subject and any language.
---

# Exam Sprint

Build a single-page study app that takes a student from zero to exam-ready in the shortest time. The app teaches only what is on the exam, drills it the way the exam asks it, and keeps the student motivated.

The student's goal is always the same: **full marks, least time, least effort.** Every decision serves that.

## 0. Before building: check the material (and say what is missing)

1. Read the exam announcement. List exactly which chapters or lectures are covered. Material outside that scope is never taught.
2. Read every page of every file. Extract the text (`pdftotext -layout`, per page), then render the pages as thumbnails and look at them. Text extraction misses images, diagrams, and **handwritten annotations**.
3. Hunt for handwritten notes on the slides (iPad/PDF annotations): "came in mid before", "very important", "Q: list two…", "may come as put circle". These are the strongest exam signals you will get. Zoom in to read them, and transcribe each one next to its slide.
4. Cross-check classmates' notes ("slide 7 is important") against the actual slide numbers. Report mismatches, such as off-by-one numbering caused by a blank slide.
5. Check for gaps: blank slides, missing chapters, past-exam questions with no matching slide. Then tell the student briefly what is complete, what is missing, and what you will do about it. Ask for anything that blocks building. Do not stop for cosmetic gaps.
6. If past exams or question collections exist, they drive everything. If the student has none, ask once. Otherwise use the handwritten notes and classmates' notes as the "bank".

## 1. Analyze how the exam asks

From the past questions and annotations, classify the question styles (multiple choice, "list N", "define", "explain and give an example", "compare", "identify the picture", scenario questions). Count them. The app trains exactly these styles, in roughly these proportions.

For every past question:
- Keep the **original text** as it appears on the paper. Fix spelling typos only.
- Check the given answer against the slides. When it is wrong, incomplete (for example, it lists points but the question asks for an explanation), or garbled, write the corrected answer. Mark it ⚠️ and keep the original visible.
- Map it to the slide and lesson it comes from.
- Group repeated questions (same concept asked several times) and mark them 🔁 ×N.

## 2. Teaching rules (non-negotiable)

- Assume the student knows nothing. Every question uses only what was already explained.
- A **Basics** mission comes first: what the course is about, the key vocabulary used everywhere, the exam format, and how to answer each question style (command word = answer size).
- Every lesson card follows this order:
  1. **The fact**, in one simple sentence.
  2. **Why / how.**
  3. **An example.**
  4. **A trick** (mnemonic, story, acronym) when something must be memorized.
  5. Optionally, "**write it like this in the exam**": the exact phrasing to reproduce.
- Card titles are statements of the fact, never questions about something not yet taught.
- Put the exam's own question right after the card that teaches it.
- Anything that must be memorized: understand it first (card), then memorize it fast (drill).
- Badges: ⭐ important · ✍️ handwritten note on the slide · 🔁 repeated in past exams · 📣 classmates' note.
- Do not build a lesson that has no slide behind it. If past questions need content with no slide, add a short clearly-labeled card and tell the student.

## 3. Language and display

- Talk to the student only in their language. Keep technical terms in the exam's language.
- In right-to-left languages, wrap every left-to-right island (terms, numbers, percentages, years) in `<bdi dir="ltr">`. Do not let short terms wrap mid-phrase (`white-space: nowrap` for short islands). Show exam questions and model answers in an LTR block.
- **Every abbreviation** (for example, IEEE) is rendered as a small stack everywhere it appears: the abbreviation, its full English name below in small type, and its translation below that in smaller type. Do this automatically with a dictionary and a regex, so nothing is missed. Avoid writing abbreviations you do not want to expand (use full words).
- Every model answer is shown with the original question above it.

## 4. App features (the checklist)

- **Missions** (one per chapter, Basics first), each a sequence of cards and questions with a progress bar, **Previous** (goes to the previous step, never home), **Skip**, and **Next**.
- **Question bank**: every past question, original text, sortable by paper and by lesson, filters (unsolved / wrong / repeated / written / MCQ), status per question, **Redo mistakes**, Arabic/native explanation, link to its lesson.
  - MCQ: auto-graded, instant feedback.
  - Written: optional textarea, reveal model answer, keyword detector ("you mentioned 2 of 3 required points"), then self-grade (got it / half / missed).
- **Picture identification**: extract the actual images from the slides (`pdfimages`) and ask "identify which type of … is this".
- **Memorize**: one drill per list that must be written in the exam. Stages: learn (with mnemonic) → pick the right items among distractors (in order for ordered lists; matching for term↔definition) → recall from memory and self-check. Stars 0–3.
- **60-second sprint** (MCQ rush, combo, time penalty for mistakes) and a **mock exam** (same mix as the real exam, score at the end, self-grade written parts).
- **Time estimates everywhere**: per card, per question, per mission, per list, remaining time, and two routes: full route and fast route (bank + core lists). Compute from content length; don't hardcode.
- **Gamification**: points, level titles from the subject's own world, combo streaks with rising sound pitch, confetti, an "approved" stamp at mission end, readiness % ring.
- **Day / night theme chosen by the student**, not the phone setting.
- **Mark-as-done button** for missions (students lose progress or study elsewhere); awards the points too.

## 5. Technical lessons learned (avoid these bugs)

- **Progress sync**: store per-user state in the artifact `db` under `data/users/<id>/progress` with the `user` capability. On load, `get()` the remote doc and merge before subscribing. Merge per item by timestamp (never overwrite a whole state with an older one). On any write error **retry with backoff and show it**; never silently fall back to local-only. Tell the student sync needs them signed in (in Safari, a signed-out public view is local only, and Safari may wipe local storage).
- **Sound on iPhone**: Web Audio is muted by the silent switch. Generate short WAV blobs and play them with `new Audio(blobURL)`; fall back to Web Audio if that fails. Unlock on the first tap.
- Test at 360px and 390px width, in both themes, and check `scrollWidth === innerWidth`.
- Build from small source files (data split by chapter, app split by area) with a tiny build script. Syntax-check with `node --check`, then run one scripted click-through with Playwright that collects console errors.

## 6. Hand-off

Give the link, then a short report: what was complete, what you corrected in the past answers (with numbers), what had no slide, the fastest route with its time, and anything the student must do (sign in for sync, unmute). Keep it short.
