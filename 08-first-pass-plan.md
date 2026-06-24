# 08 — Первый заход: план реализации (3 кросс-тема карты на тонком движке)

> **Операционный план — фиксирует принятые решения:**
> - **Объём:** РАЗНЫЕ колоды, по 1 карте (кросс-тема) — проверяет бизнес-тезис «один пайплайн тянет много тем».
> - **Пайплайн:** ТОНКИЙ движок → потом закалить (не строим фабрику до валидации на детях).
> - **Движок:** «Interactive AI-Clip Stage + reusable plugins» (`07`), граница `07` §0, Factory Rule `07` §2.
> **Замещает `06`** для выбранного объёма (`06` оставлен как within-deck-альтернатива).

---

## 0. Что и почему (+ осознанный размен)

3 карты в 3 темах на одном тонком движке. **Размен, который берём осознанно:** больше арт-вариативности; в этот заход **НЕ** тестируем внутри-колодную связность и коллекционный мета-слой (для них нужно несколько карт в одной колоде — отдельный within-deck пас позже).
**Митигация:** один нейтральный гид **БИП** (дрон-исследователь) во всех 3 темах → заодно проверяем, что «гид переезжает» между темами (это центр retention-стратегии серии).

---

## 1. Три карты

| # | Тема | Карта | `mechanic` | Тракт | Эмоция / темп | Что тестирует |
|---|---|---|---|---|---|---|
| 1 | **Динозавры** | «Сбеги от T-Rex» | `branching-choice` | AI-клип | страх / громко | clip-ветка, решение-по-факту; **reuse арта+roar POC** (дешевле всего) |
| 2 | **Океан** | «Спаси стаю от удильщика» | `catch-dodge` | **Arcade-physics** | спасение / frantic | physics-плагин reuse; «физика даёт replay»; **сигнатурный паттерн Степана** |
| 3 | **Космос** | «Разбуди сверхновую» | `drag-scrub` | AI-клип | контроль→вау / тихо→взрыв | scrub-хендлер; «вау» из одного клипа, почти даром |

3 темы · 3 сигнатуры · 3 ввода (tap-ветка / drag-физика / scrub) · 2 clip + 1 physics · 3 эмоции.

---

## 2. Манифесты (3 карты, по схеме `07` §2–3)

**Карта 1 — T-Rex (branching-choice):**
```json
{
  "id": "DINO-TREX-ESCAPE-001", "theme": "dinosaurs", "title": "Сбеги от тираннозавра",
  "guide": "beep", "mechanic": "branching-choice", "pressure": "quest",
  "learning": { "fact": "T-Rex наводится на ДВИЖЕНИЕ; неподвижную добычу теряет",
                "misconception": "надо убегать быстрее", "win_requires_understanding": true },
  "assets": { "art": "art/trex.png",
              "clips": { "intro": "clips/trex-notices.mp4", "freeze_win": "clips/trex-loses.mp4", "run_caught": "clips/trex-almost.mp4" },
              "tts": { "intro": "tts/beep-trex.mp3" }, "sfx": { "roar": "sfx/roar.mp3" } },
  "flow": [
    { "state": "card", "on": { "tap": "intro" } },
    { "state": "intro", "play": "intro", "then": "choice" },
    { "state": "choice", "prompt": "Он видит движение. Что делаешь?",
      "inputs": [ { "type": "tap-hotspot", "label": "Замри", "branch": "freeze_win" },
                  { "type": "tap-hotspot", "label": "Беги в поле", "branch": "run_caught" } ] },
    { "state": "freeze_win", "play": "freeze_win", "result": "win", "then": "reward" },
    { "state": "run_caught", "play": "run_caught", "result": "soft-fail", "then": "retry" }
  ],
  "a11y": { "reduced_motion": "poster+caption", "timer": false, "voice_first": true }
}
```

**Карта 2 — Удильщик (catch-dodge, плагин):**
```json
{
  "id": "OCEAN-ANGLER-SAVE-001", "theme": "ocean", "title": "Спаси стаю от удильщика",
  "guide": "beep", "mechanic": "catch-dodge", "pressure": "quest",
  "learning": { "fact": "удильщик приманивает добычу светящимся «фонариком» в темноте",
                "misconception": "свет в глубине = безопасно", "win_requires_understanding": true },
  "plugins": [
    { "type": "arcade-physics", "spawn": "fish-school", "attractor": "angler-lure",
      "control": "drag-barrier", "win": { "rule": "saved >= 8 of 10", "fail_outcome": "soft" } },
    { "type": "particle-fx", "preset": "bubble-burst", "trigger": "save" },
    { "type": "camera-fx", "preset": "dim-pulse", "trigger": "lure-glow" }
  ],
  "assets": { "bg": "art/deep-sea.png",
              "sprites": { "fish": "sprites/fish-loop.webm", "angler": "sprites/angler-loop.webm" },
              "tts": { "intro": "tts/beep-angler.mp3" }, "sfx": { "save": "sfx/blip.mp3" } },
  "a11y": { "reduced_motion": "slower-spawn", "timer": false, "voice_first": true }
}
```

**Карта 3 — Сверхновая (drag-scrub):**
```json
{
  "id": "SPACE-SUPERNOVA-001", "theme": "space", "title": "Разбуди сверхновую",
  "guide": "beep", "mechanic": "drag-scrub", "pressure": "explore",
  "learning": { "fact": "массивная звезда в конце жизни коллапсирует и взрывается сверхновой",
                "misconception": "звёзды просто медленно гаснут", "win_requires_understanding": true },
  "assets": { "art": "art/giant-star.png", "clips": { "scrub": "clips/supernova.mp4" },
              "tts": { "intro": "tts/beep-supernova.mp3" } },
  "flow": [
    { "state": "card", "on": { "tap": "intro" } },
    { "state": "intro", "play_tts": "intro", "then": "scrub" },
    { "state": "scrub", "scrub_clip": "scrub", "drive": "drag-x", "on_complete": "reward",
      "fx": [ { "preset": "flash", "at": 0.85 }, { "preset": "shake", "at": 0.85 } ] }
  ],
  "a11y": { "reduced_motion": "tap-to-advance", "timer": false, "voice_first": true }
}
```

Все три читает один Stage-движок; новая карта = JSON + ассеты, **ноль нового кода** (кроме arcade-плагина, который строится раз).

---

## 3. Ассеты на карту + Kling-промпты (i2v)

Правило консистентности (`07` §5): **один input-кадр (арт карты) на ВСЕ ветви**; короткие 3–5 с; camera static/locked; нейтральный фон для альфа-актёров; `generate_audio` для рёва.

- **Карта 1 (reuse + догенерить):** арт T-Rex (есть в POC) + roar (есть). Догенерить: `intro` «T-Rex медленно поворачивает голову, фиксирует взгляд, низкий рык, 4с, generate_audio» · `freeze_win` «нюхает воздух, не находит неподвижную добычу, фыркает, уходит, 4с» · `run_caught` (мягко) «комичный near-miss, экран-шейк, без крови, 3с».
- **Карта 2 (alpha-актёры):** фон `deep-sea.png` (gen) + спрайт-лупы на тёмном фоне для matte → `fish-loop` «стайка мелкой рыбы плывёт, бесшовный луп, изолировано на чёрном, 3с» · `angler-loop` «глубоководный удильщик, светящийся фонарик покачивается, бесшовный луп, на чёрном, 3с». Matte → webm/hevc alpha.
- **Карта 3 (один scrub-клип):** арт `giant-star.png` (gen) → `supernova` «гигантская звезда сжимается, коллапсирует и взрывается сверхновой, плавно непрерывно, плоский фон, 6с» → скрабится пальцем; вспышка/шейк на 85% = camera-fx (не из видео).

Негативы (общие): `blurry, extra limbs, text, watermark, fast camera, scene cut, gore`.

---

## 4. Тонкий движок — build-list (ровно под 3 карты)

| Движковый кусок | Карта 1 | Карта 2 | Карта 3 |
|---|:--:|:--:|:--:|
| Stage-ядро (state-машина + загрузчик манифеста, Phaser) | ✓ | ✓ | ✓ |
| Манифест-схема (Zod) + линтер границы (`07` §0) | ✓ | ✓ | ✓ |
| Хендлер `tap-hotspot` | ✓ | | |
| Хендлер `drag-barrier`/`drag-follow` | | ✓ | |
| Хендлер `drag-scrub` | | | ✓ |
| **arcade-physics плагин** (config-driven) | | ✓ | |
| FX-пресеты: particle (`bubble-burst`,`impact`) | | ✓ | |
| FX-пресеты: camera (`shake`,`flash`,`dim-pulse`) | | ✓ | ✓ |
| Clip-плеер + alpha-sprite слой + audio | ✓ | ✓ | ✓ |
| Offline deck-pack shell + a11y + privacy-телеметрия | ✓ | ✓ | ✓ |
| Слой гида (БИП — AI-спрайт, один на все 3) | ✓ | ✓ | ✓ |

3 карты **полностью прогоняют** тонкий движок (оба тракта + 3 ввода + FX + гид + offline/a11y).

---

## 5. Стек + скелет репозитория движка

**Стек:** TS-monorepo · Vite PWA · **Phaser** (Arcade+Matter встроены) · Zod · Workbox · fal.ai/Kling за provider-adapter · Nano Banana/GPT Image · TTS · Playwright · Cloudflare Pages · git-as-truth для манифестов · **один runtime + deck-packs**.

```
infinite-atlas-engine/
  packages/
    stage/        # Interactive AI-Clip Stage (Phaser): state-machine, input/, plugins/{arcade-physics,particle-fx,camera-fx}, fx-presets/
    schema/       # Zod manifest schema + linter (граница 07 §0)
    shell/        # PWA shell, deck-pack loader, offline, a11y, privacy-telemetry, guide layer
  decks/
    dinosaurs/cards/trex-escape/{manifest.json, assets/}
    ocean/cards/angler-save/{manifest.json, assets/}
    space/cards/supernova/{manifest.json, assets/}
  scripts/        # gen: image, kling-i2v, matte->sprite, tts ; assemble (assets+manifest)
  tests/          # Playwright: reachability, offline, a11y, perf/decoders, fixed-timestep
```

---

## 6. Операционно-безопасный слой — ОБЯЗАТЕЛЕН (из `05` §7 / `06` §5)

С первого билда: **доступность** (`reduced_motion`-фоллбэк, `timer:false`, voice-first 7–8, крупные хитбоксы) · **privacy-safe телеметрия** (локально, без аккаунта/PII; попытки/выборы/replay) · **физ-вход** (повреждённая карта/свет/нет камеры/ручной код/дубликат) · **offline lifecycle** (deck-pack, eviction, атомарное обновление) · **device-matrix** (≥1 low-end Android + старый iOS; перф Arcade + число видео-декодеров) · **safety/факт human-чек** 3 карт · **этичный мини-мета** (видимый прогресс 3/3, без FOMO).

---

## 7. Что НЕ строим в этот заход (тонкий выбор)

Авто LLM-контент-тулинг · оркестрация/админка · multi-provider router · **Matter** (slingshot/stack — следующим reusable-модулем) · богатый мета-слой/ежедневки · C2PA-авто · отдельные приложения на колоду · world-models/gen-3D.
**Но** provenance/лицензии/human-approvals/generation-recipe — с первого дня.

---

## 8. Критерии успеха / стопа

**Идём дальше (закаляем пайплайн + расширяем), если:**
- **Reuse (H1):** карты №2 и №3 собраны **конфигом, 0 bespoke-кода**; движок переехал на 3 темы — Factory Rule держит;
- **Physics (H2):** удильщик даёт ощутимо больше replay/мастерства, чем clip-карты;
- **Learning (H3/H5):** на всех 3 ребёнок объясняет причину и переносит; выиграть, не поняв факт, нельзя;
- все 3 бьют POC-кнопки; мягкий профиль не хуже для 7–8; проходит device-matrix.

**Стоп / переделка, если:** карта потребовала bespoke-кода (движок недо-обобщён) · physics не даёт прироста replay · клипы-ветви «не из одной карты» (закрепить «один input-кадр») · жёсткий исход растит тревогу младших (Quest дефолт).

---

## 9. Порядок работ (build order для Codex/Claude Code)

1. Скаффолд monorepo (стек §5).
2. `schema/` — Zod-манифест + линтер границы.
3. `stage/` — state-машина + загрузчик + clip-плеер + audio.
4. Хендлеры: `tap-hotspot` → `drag-scrub` → `drag-barrier`.
5. `arcade-physics` плагин (config-driven) + FX-пресеты.
6. `shell/` — offline deck-pack + a11y + privacy-телеметрия + слой гида.
7. Ген-ассеты 3 карт (полу-вручную: image → kling i2v → matte → tts) + 3 манифеста.
8. Assembly + Playwright-гейты.
9. **Детский плейтест** (страты 7–8 / 9–10 / 11–12) + delayed recall/transfer (+1д/+1нед).
10. Решение по закалке пайплайна и расширению (within-deck пас / кросс-тема дальше).

---

*Это «первый заход на пайплайне, который ре-юзается для всего последующего»: тонкий движок строится раз, 3 карты в 3 темах доказывают и настоящую играбельность, и переиспользование, и перенос знания — до любой фабрики и до полной колоды.*
