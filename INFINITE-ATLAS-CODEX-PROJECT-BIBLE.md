# Infinite Atlas — Project Bible for Codex

**Version:** 1.0  
**Date:** 2026-06-24  
**Status:** стартовая рабочая основа проекта  
**Audience:** Андрей как разработчик и владелец производственного пайплайна; Codex как основной coding-agent; Степан как создатель продукта и колод

---

## 0. Зачем существует этот документ

Этот файл нужен, чтобы новый Codex-сеанс мог быстро и правильно войти в контекст Infinite Atlas без пересказа всей истории исследований и споров.

Это **не разовый промпт** и не подробное техническое задание на одну задачу. Это проектная библия:

- зачем существует продукт;
- что мы пытаемся доказать первым прототипом;
- какие продуктовые и инженерные решения уже приняты;
- какие идеи пока остаются гипотезами;
- где проходит граница автоматизации;
- какие первые карточки строим;
- какой runtime и AI-пайплайн нужен;
- какие правила Codex обязан соблюдать при изменении репозитория;
- по каким признакам мы считаем результат успешным или требующим переделки.

После чтения Codex должен понимать не только **что писать**, но и **почему именно так**, что нельзя незаметно усложнить и какие реальные доказательства должны появиться после работы.

---

# 1. Контекст продукта

## 1.1. Что такое Infinite Atlas

Infinite Atlas — фиджитал-платформа обучения детей 7–12 лет:

- ребёнок получает физические коллекционные карточки;
- каждая физическая карточка открывает короткий интерактив в бесплатном мобильном PWA-компаньоне;
- приложение должно работать офлайн после установки контента колоды;
- карточка должна быть привлекательным физическим объектом сама по себе;
- цифровой слой должен не заменять карточку, а заметно усиливать её ценность;
- темы могут быть любыми: динозавры, океан, роботы, космос, тело, растения, история, инженерия и другие.

Продукт не должен ощущаться как электронный учебник или коллекция QR-ссылок на ролики.

Целевое ощущение:

> «Я приложил/открыл настоящую карту, и из неё возникло короткое игровое событие, в котором я сам что-то сделал, понял и пережил».

## 1.2. Бизнес-тезис

Продукт успешен, если одновременно работают две оси:

### Для ребёнка

- карта вызывает сильную эмоцию;
- ребёнок действует, а не только смотрит;
- он хочет повторить интерактив;
- он запоминает событие от первого лица: «я спас», «я нашёл», «я починил»;
- ему интересно открыть другие карты и следующую колоду.

### Для производства

- новые карты можно выпускать быстро;
- AI создаёт большую часть арта, видео, аудио и черновых конфигураций;
- игровой код и производственные инструменты переиспользуются;
- человеческое время не тратится на ручную анимацию и уникальную разработку каждой мини-игры;
- качество контролируется автоматическими проверками и короткими human approval-гейтами.

Главное напряжение проекта:

> Переиспользование снижает стоимость, но легко делает карты одинаковыми.  
> Задача архитектуры — переиспользовать невидимую фабрику, не повторяя одно и то же детское переживание.

---

# 2. Текущий уровень уверенности

Исходное исследование в репозитории было создано в основном через AI-рассуждение и не проверялось на реальных детях.

Поэтому:

- идеи механик — полезный backlog;
- архитектурные решения — рабочие гипотезы;
- конкретные числа вроде «ровно 4–5 hero», «60–70% library», «минимум пять жестов» не являются доказанными законами;
- главный источник истины после прототипа — наблюдение реальных детей и фактические производственные метрики;
- красивый работающий код не считается доказательством fun;
- улыбка ребёнка в одном прохождении не считается доказательством learning или retention.

Мы строим первый прототип не для подтверждения заранее любимой идеи, а для получения данных, которые могут заставить нас изменить архитектуру.

---

# 3. Что выяснили о детском опыте

## 3.1. Ребёнок должен быть причиной события

Анти-паттерн старого POC:

```text
тап
→ ролик
→ вопрос с вариантами ответа
→ reward
```

Желаемый цикл:

```text
ребёнок замечает ситуацию
→ принимает решение
→ совершает действие
→ мир реагирует именно на это действие
→ ребёнок понимает причину результата
→ появляется желание повторить лучше или иначе
```

Tap, drag или swipe сами по себе не превращают контент в игру. Важны решение и причинная модель.

## 3.2. Fun contract

У каждой карты до разработки должен быть записан **fun contract**:

1. **Детская фантазия**  
   Что ребёнок сможет сказать после игры?  
   Примеры: «Я починил робота», «Я включил зрение звуком», «Я оживил Рим водой».

2. **Главная эмоция**  
   Любопытство, комедия, тихое напряжение, спасение, мастерство, магия превращения, разрушение, гордость.

3. **Эмоциональная дуга**  
   Завязка → напряжение/неопределённость → решение → почти-провал → развязка.

4. **Реальная агентность**  
   Что зависит от ребёнка, кроме запуска ролика?

5. **Причина повторить**  
   Новый дефект, иной маршрут, другая раскладка, более точное решение, Challenge-профиль, новый исход.

Красивый AI-клип может усиливать развязку, но не заменяет эти пять вещей.

## 3.3. Learning contract

У каждой образовательной карты должен быть отдельный **learning contract**:

```text
goal
misconception
winRequires
feedback
nearTransfer
laterTransfer
```

### `goal`

Что ребёнок должен понять, а не просто услышать.

### `misconception`

Какую типичную ошибочную модель мы предотвращаем.

### `winRequires`

Какое понимание необходимо, чтобы стабильно выиграть.

Главный тест:

> Можно ли уверенно пройти карту, не поняв целевое правило?

Если да — обучение приклеено сверху.

### `feedback`

Как игра показывает причину успеха или ошибки, а не просто красный крест.

### `nearTransfer`

Новый похожий пример сразу после основной задачи.

### `laterTransfer`

Как этот принцип всплывёт на другой карте или в финальной миссии колоды.

## 3.4. Возрастные слои

Одна карта должна масштабироваться по сложности.

### Для 7–8 лет

- одна ясная цель;
- крупные touch targets;
- voice-first;
- минимум текста;
- сильный визуальный причинный feedback;
- мягкое восстановление после ошибки;
- меньше одновременно действующих правил.

### Для 9–12 лет

- меньше подсказок;
- несколько признаков;
- конфликтующие сигналы;
- оптимизация;
- ограниченные попытки или ресурсы;
- дополнительная цель;
- перенос на новый объект.

Не надо строить две отдельные игры. Различия должны задаваться параметрами и контентом.

## 3.5. Три профиля давления

Не каждая карта должна угрожать ребёнку и торопить его.

```text
Explore
Quest
Challenge
```

### Explore

- проиграть нельзя;
- можно спокойно экспериментировать;
- подходит для первого запуска, младших детей и исследовательских карт.

### Quest

- основной режим;
- ошибка видна, но обратима;
- ребёнок быстро возвращается в действие;
- провал информативен.

### Challenge

- таймер, рекорд, ограниченные попытки;
- открывается позже или выбирается добровольно.

Здоровый провал должен быть:

- контролируемым;
- понятным;
- восстановимым;
- пропорциональным;
- информативным;
- без стыда и чрезмерно тяжёлых последствий.

---

# 4. Целостность будущих колод

## 4.1. Карты должны работать в любом порядке

Физические карты могут попадаться и сканироваться в произвольной последовательности. Поэтому колода не должна зависеть от жёсткой сюжетной цепочки «карта 1 → карта 2 → карта 3».

Нужна модульная структура:

- каждая карта работает самостоятельно;
- все карты участвуют в общей миссии;
- результаты попадают в общий журнал или карту мира;
- некоторые знания и способности используются повторно;
- несколько карт открывают финальную миссию.

## 4.2. Разнообразие измеряется не названием механики

Для каждой карты хранится **experience signature**:

```text
роль ребёнка
цель
доступная информация
тип решения
тип управления
динамика системы
темп
давление
последствие
эмоция
причина replay
learning rule
```

Две карты с одинаковым drag могут быть разными, если в одной ребёнок диагностирует систему, а в другой управляет спасением.

Tap и swipe могут быть одинаково скучными, если везде нужно коснуться очевидной цели и получить одинаковый салют.

## 4.3. Что переиспользуем и что меняем

### Переиспользуем агрессивно

- runtime;
- state machine;
- input plugins;
- physics plugins;
- FX;
- камеры;
- offline shell;
- UI;
- actors;
- scenes;
- audio banks;
- schema;
- tests;
- asset processors.

### Обязано меняться для ребёнка

- роль;
- решение;
- причинная модель;
- темп;
- эмоциональный регистр;
- пространство;
- реакция персонажа;
- последствия;
- условия повторного прохождения.

## 4.4. Hero и library

Hero/library-разделение полезно как бюджетная модель, но не является точной формулой.

Рабочие уровни:

### Library

Существующий plugin + существующий production route + новый manifest и ассеты.

### Enhanced library

Тот же plugin, но более богатая постановка, новый actor pack или дополнительные состояния.

### Hero

Особая композиция, сильный эмоциональный момент, возможно полноэкранное AI-видео. Hero не обязан быть самым дорогим и не обязан быть видео.

Точное количество карт каждого уровня определится после mini-deck playtest.

## 4.5. Коллекция и «купи ещё»

Коллекционный слой не должен держаться только на редкости, дубликатах, ежедневных наградах и страхе пропустить.

Более содержательная гипотеза:

- каждая карта открывает новое знание, способность, инструмент или способ взаимодействия;
- эти способности используются в финальной миссии;
- ребёнок хочет следующую карту не только ради пустого слота, но и ради новой возможности.

Пример:

```text
эхо-сканер
камуфляж
управление потоком
полевой скан следов
```

Это пока гипотеза для mini-deck, не часть первого технического critical path.

---

# 5. Принятая производственная доктрина

## 5.1. Что запрещено

### Никакой ручной organic-анимации персонажей

Не используем как обязательную производственную операцию:

- покадровую анимацию;
- ручной персонажный rig;
- ручную анимацию в Rive/Spine;
- ручную правку каждого кадра.

Характерное движение органических персонажей создаётся:

- AI image-to-video;
- AI-video-derived sprite atlas;
- готовыми outcome/hero clips;
- статичными AI-состояниями с простым переходом, если этого достаточно.

### Никакого bespoke per-card runtime-кода

Нельзя:

```ts
if (cardId === "special-card") {
  // уникальная логика
}
```

Новая карта в норме:

```text
card spec
+ manifest
+ AI assets
+ configuration of existing plugins
```

### Никакой ручной per-card векторной фабрики

Не планируем рисовать для каждой карты вручную:

- SVG-скелеты;
- сложные схемы;
- наборы ручных путей;
- персонажей по частям;
- уникальные векторные анимации.

Универсальные UI-примитивы допустимы:

- рамки;
- стрелки;
- шкалы;
- зоны;
- маски;
- текст;
- простая геометрия runtime.

Точные научные схемы позже получают отдельный маршрут на основе проверенных источников. Нельзя считать свободно сгенерированную AI-картинку каноническим источником точной анатомии или инженерной схемы.

## 5.2. Что разрешено и желательно

- простые transforms;
- tweens;
- camera effects;
- particle effects;
- Arcade Physics;
- позже Matter Physics;
- mask reveal;
- path validation;
- audio-driven effects;
- state machines.

Но только как reusable config-driven plugins, написанные один раз.

## 5.3. Правило нового plugin

Новый runtime-модуль разрешён, когда одновременно выполнено следующее:

1. Есть минимум две реалистичные карточки, которые его используют.
2. Желательно есть применение в разных темах.
3. У plugin есть Zod-схема конфигурации.
4. Он не знает card IDs.
5. Он покрыт unit-тестами.
6. Есть автоматический E2E-сценарий.
7. Он проверен на слабом устройстве.
8. Следующая карта может использовать его без программиста.
9. Его первоначальная стоимость оправдана будущим reuse.

Если это не выполняется, нужно упростить идею карты или отложить её.

---

# 6. Архитектура AI-first производства

## 6.1. Общий поток

```text
Human brief
      ↓
card.spec.json
      ↓
AI asset plan
      ↓
generation jobs
      ↓
automatic asset processing
      ↓
card.manifest.json
      ↓
Stage + plugins
      ↓
automated tests
      ↓
human approval
      ↓
child playtest
      ↓
deck pack
```

## 6.2. Не смешивать план и факт

Нужны разные артефакты.

### `card.spec.json`

Что мы хотим получить:

```text
product role
fun contract
learning contract
pressure profile
production route
required plugins
asset list
budget envelope
acceptance criteria
known risks
fallbacks
```

### `card.manifest.json`

Что реально исполняет приложение:

```text
states
events
inputs
actors
assets
plugins
parameters
branches
outcomes
accessibility fallbacks
```

### `asset.recipe.json`

Как производятся ассеты:

```text
provider
model
reference assets
prompt templates
negative constraints
expected outputs
post-processing route
```

### `run.jsonl`

Что реально произошло при производстве:

```text
timestamp
provider/model/version
generation id
attempt count
cost
processing duration
test results
human review minutes
accept/reject/regenerate
reject reason
```

Плановая скорость или стоимость не считается фактом, пока не появилась в run-данных.

## 6.3. Маршруты ассетов

### Route A — Static AI asset

Для:

- backgrounds;
- food;
- obstacles;
- modules;
- scenery;
- still character states.

```text
reference/style pack
→ image generation/edit
→ background removal if needed
→ crop
→ resize
→ alpha QA
→ optimize
```

### Route B — Reusable animated actor pack

```text
approved canonical image
→ AI i2v states
→ matte
→ alpha QA
→ crop
→ scale normalization
→ anchor normalization
→ loop check
→ frame extraction
→ sprite atlas
→ actor.json
```

Используется для существ и повторяющихся персонажей.

Правило:

> Плохой ассет сначала регенерируется или заменяется более простым route, а не чинится вручную покадрово.

### Route C — Hero/outcome video

Для:

- появления;
- эмоционального исхода;
- большого reveal;
- финала;
- reward.

Лучше полноэкранное видео или видео на контролируемом фоне, чем обязательная сложная альфа.

### Route D — Reusable code motion and FX

Для:

- управления;
- столкновений;
- полей притяжения;
- камеры;
- пыли;
- искр;
- вспышек;
- trails;
- success/failure feedback.

Codex пишет модуль один раз; карта использует его JSON-конфигом.

## 6.4. Fallback-лестница

Для сложного визуального эффекта выбирать самый простой приемлемый маршрут:

1. Static states + crossfade/tween.
2. Sprite atlas.
3. Full-frame outcome clip.
4. Alpha video только после device testing.

Не надо доказывать технологию ценой срыва карточки.

---

# 7. Технологический стек

## 7.1. Основной стек первого пилота

```text
TypeScript strict
pnpm workspaces
Vite
Phaser 3.90.x — версия должна быть pinned
XState 5
Zod
vite-plugin-pwa (injectManifest)
Workbox
IndexedDB
Cache Storage
Vitest
Playwright
ffmpeg / ffprobe
sharp
tsx for tooling scripts
```

## 7.2. Почему Phaser

Phaser отвечает за:

- сцены;
- touch input;
- sprite atlases;
- audio;
- cameras;
- particles;
- Arcade Physics;
- позднее Matter Physics;
- lifecycle.

Не подключать PixiJS параллельно без доказанной необходимости.

## 7.3. Почему XState

XState используется только для flow карты:

```text
intro
playing
feedback
success
failure
retry
complete
```

Phaser управляет сценой. XState управляет состояниями и переходами. Не дублировать одну логику в обоих слоях.

## 7.4. Offline PWA

Архитектура:

```text
one PWA shell
+
installable versioned deck packs
```

### Shell

- UI;
- runtime;
- plugin code;
- deck installer;
- offline status;
- settings;
- local progress.

### Deck pack

- manifests;
- images;
- atlases;
- clips;
- audio;
- localization;
- hashes.

Установка:

```text
check available storage
→ download into staging
→ verify hashes
→ atomically activate manifest
→ mark offline-ready
```

Большие media-assets не должны автоматически попадать в общий app-shell precache.

Storage может быть очищено браузером. Приложение обязано:

- обнаружить потерянный pack;
- не считать карту offline-ready при отсутствующих assets;
- предложить восстановление;
- не терять сам PWA shell.

## 7.5. Provider abstraction

Доменные schemas и runtime не зависят от конкретного провайдера.

```ts
interface ImageProvider {}
interface VideoProvider {}
interface MatteProvider {}
interface SpeechProvider {}
```

В первом пилоте достаточно одной реализации каждого интерфейса.

Не строить multi-provider router до появления реальных объёмов и failure modes.

---

# 8. Рекомендуемая структура репозитория

```text
infinite-atlas/
  PROJECT-BIBLE.md
  AGENTS.md
  PLANS.md
  package.json
  pnpm-workspace.yaml

  apps/
    pwa/

  packages/
    runtime/
    schema/
    tooling/
    plugins/
      drag-zone/
      clip-branch/
      flow-path/
      pulse-scan/
      fx/
      arcade-field/
      mask-reveal/

  content/
    cards/
      robot-repair/
        card.spec.json
        card.manifest.json
        asset.recipe.json
        assets/
        tests/
      roman-aqueduct/
      whale-echo/
      cuttlefish-camo/
      anglerfish-save/
      track-detective/

    actors/
    scenes/
    audio/
    localization/

  generated/
    manifests/
    atlases/
    previews/
    packs/

  runs/
    .gitkeep

  tests/
```

`runs/` с большими и чувствительными actuals может быть gitignored; утверждённые recipes, specs и manifests — versioned.

---

# 9. Первые карточки и порядок производства

Текущий рекомендуемый порядок для одного разработчика:

1. **Почему робот не едет?**
2. **Вода для Рима**
3. **Эхо в полной темноте**
4. **Исчезни, каракатица!**
5. **Спаси стаю от удильщика**
6. **Следовой детектив**

Первые четыре доказывают базовую фабрику. Пятая добавляет более выраженную аркадную физику. Шестая добавляет дешёвую исследовательскую механику.

T-Rex POC сохраняется как техническая fixture и источник ассетов, но не является обязательной первой учебной картой.

---

# 10. Карта 1 — «Почему робот не едет?»

## 10.1. Роль в программе

Первая техническая канарейка и главный ранний learning-proof.

Она проверяет:

- Stage;
- XState flow;
- `drag-zone`;
- branching;
- реакционные клипы;
- static assets;
- manifest assembly;
- automated branches;
- offline deck card.

## 10.2. Детская фантазия

> «Смешной робот сломался. Я понял почему, починил его — и он ожил».

## 10.3. Fun contract

Главная эмоция:

```text
эмпатия
→ комедия
→ любопытство
→ «ага!»
→ гордость
```

Fun создают:

- робот пытается выполнить миссию и смешно ломается;
- симптом имеет характер;
- неправильный ремонт вызывает информативную комедию;
- правильный ремонт даёт сильное оживление;
- дефект меняется при повторе;
- после починки робот совершает короткий победный номер.

Неправильная деталь не показывает просто красный крест.

Пример:

> Антенна вместо батареи заставляет робота ловить радио, но он всё ещё не едет.

## 10.4. Learning contract

### Goal

Понять функциональную цепочку:

```text
energy source
→ control
→ motor
→ transmission
→ wheel
```

### Misconception

«Если робот не едет, надо просто менять случайные детали».

### Win requires

Ребёнок должен соотнести наблюдаемый симптом с функциональным звеном.

### Feedback

Реакция робота показывает, что изменилось и что осталось неисправным.

### Near transfer

Другой робот с иной внешностью, но той же функциональной цепочкой.

### Later transfer

Принцип диагностики используется позже в сердце, водной системе или космическом аппарате.

## 10.5. Production route

- статичный/анимированный AI-робот;
- 3–4 reaction clips;
- AI-generated modules;
- `drag-zone`;
- `clip-branch`;
- simple FX.

Никаких схем и ручных SVG в v1.

## 10.6. Acceptance

- карту нельзя стабильно пройти случайным первым выбором;
- неправильная реакция информативна;
- минимум два разных дефекта;
- runtime не содержит robot-specific branch;
- автоматический тест проходит success и wrong-part paths;
- ребёнок после карты объясняет причину неисправности.

---

# 11. Карта 2 — «Вода для Рима»

## 11.1. Роль в программе

Проверяет reuse `drag-zone` и новый маленький структурный plugin без жидкостной симуляции.

## 11.2. Детская фантазия

> «Город остался без воды. Я поймал правильный уклон — и оживил весь Рим».

## 11.3. Fun contract

Главная эмоция:

```text
ответственность
→ инженерная догадка
→ ожидание запуска
→ почти остановка
→ большой триумф
```

Fun создают:

- город визуально ждёт воду;
- после сборки есть короткая пауза перед запуском;
- поток бежит по пути ребёнка;
- при ошибке происходит смешной, понятный сбой;
- при успехе оживают фонтаны, термы и жители;
- новый рельеф при повторе.

## 11.4. Learning contract

### Goal

Понять:

- вода движется вниз;
- канал должен иметь непрерывный общий уклон;
- восходящий сегмент блокирует поток;
- арка поддерживает канал над долиной, а не «делает воду сильнее».

### Misconception

«Акведук толкал воду вверх как насос».

### Win requires

Ребёнок собирает непрерывный маршрут с допустимым изменением высоты.

### Feedback

Вода останавливается именно перед неправильным сегментом или переливается на слишком резком перепаде.

### Near transfer

Новая долина с другой высотой слотов.

### Later transfer

Принцип потока используется в ирригации, трубах, кровотоке или конвейере.

## 11.5. Production route

Не строить fluid simulation.

Сегменты имеют данные:

```json
{
  "inHeight": 0.72,
  "outHeight": 0.63,
  "capacity": 1,
  "connections": ["left", "right"]
}
```

Plugin `flow-path` проверяет:

- continuity;
- slope;
- capacity;
- goal reachability.

Визуальная вода:

- moving texture;
- последовательные короткие сегменты;
- outcome clip.

## 11.6. Acceptance

- plugin не содержит Rome-specific code;
- тот же plugin можно описать для ирригации или трубы;
- ребёнок может объяснить, почему вода остановилась;
- success sequence эмоционально заметен;
- first playable не требует ручного вектора или симуляции жидкости.

---

# 12. Карта 3 — «Эхо в полной темноте»

## 12.1. Роль в программе

Проверяет новый сенсорный вкус с небольшим количеством дорогих AI-ассетов.

## 12.2. Детская фантазия

> «Я ничего не видел, но щелчком нашёл проход и объект в абсолютной темноте».

## 12.3. Fun contract

Главная эмоция:

```text
тайна
→ догадка
→ напряжение
→ точное обнаружение
```

Fun создают:

- почти чёрный экран;
- ребёнок сам на мгновение проявляет мир;
- неизвестно, что вернёт следующий импульс;
- stereo audio, визуальное кольцо и vibration fallback;
- ограниченный, но не стрессовый запас импульсов;
- разные скрытые раскладки при повторе.

## 12.4. Learning contract

### Goal

Понять:

- звуковой импульс распространяется;
- объект отражает сигнал;
- близкий объект даёт ранний отклик;
- дальний — поздний;
- направление влияет на обнаружение.

### Misconception

«Эхолокация — это особое ночное зрение без сигнала и отражения».

### Win requires

Ребёнок интерпретирует задержку и направление echo response.

### Feedback

Объект проявляется с задержкой, соответствующей расстоянию.

### Near transfer

Та же логика с новой конфигурацией объектов.

### Later transfer

Тот же plugin используется у летучей мыши, подлодки, робота и пещеры.

## 12.5. Production route

Не нужна полноценная акустическая симуляция.

Объекты задаются данными:

```json
{
  "angle": 35,
  "distance": 0.42,
  "kind": "wall",
  "echoDelayMs": 330,
  "echoStrength": 0.8
}
```

`pulse-scan`:

```text
direction
→ objects in sector
→ delay and strength
→ audio/visual response
→ temporary reveal
```

## 12.6. Acceptance

- карта понятна с визуальным fallback без хороших наушников;
- audio не является единственным каналом;
- plugin применяется минимум к двум будущим темам;
- задержка связана с расстоянием;
- ребёнок объясняет принцип отражённого сигнала.

---

# 13. Карта 4 — «Исчезни, каракатица!»

## 13.1. Роль в программе

No-new-runtime-code acceptance card.

Она проверяет:

- новую тему;
- AI consistency;
- visual wow;
- manifest-driven assembly;
- reuse `drag-zone`/branch/show-hide.

## 13.2. Детская фантазия

> «Хищник смотрел прямо на каракатицу, но я заставил её исчезнуть».

## 13.3. Fun contract

Главная эмоция:

```text
магия превращения
→ suspense
→ близкий промах
→ облегчение
```

Fun создают:

- моментальное изменение кожи;
- медленный проход хищника;
- плохой выбор вызывает ink escape;
- хороший выбор заставляет хищника проплыть рядом;
- разные backgrounds и patterns;
- желание попробовать другую комбинацию.

## 13.4. Learning contract

### Goal

Понять:

- камуфляж — не просто «стать зелёным»;
- животное подбирает общий pattern и contrast к среде;
- один pattern не подходит всем фонам;
- иногда важно разрушить заметный контур.

### Misconception

«Достаточно совпасть с одним цветом».

### Win requires

Ребёнок выбирает и укрытие, и pattern.

### Feedback

Хищник замечает контур при неподходящей комбинации.

### Near transfer

Смешанная среда с двумя визуальными признаками.

### Later transfer

Сравнение с другим способом маскировки или поиск скрытого объекта.

## 13.5. Production route

Не строить real-time skin shader.

Один canonical master, затем согласованные states через image editing:

```text
uniform
mottle
disruptive
```

Переход:

- crossfade;
- small pre-baked transformation clip;
- static state swap with FX.

Predator:

- один pass clip;
- один detect outcome;
- один ink escape outcome.

## 13.6. Acceptance

- после карт 1–3 не появляется новый runtime plugin;
- canonical shape/pose сохраняется;
- ручной покадровой правки нет;
- плохой asset регенерируется;
- ребёнок объясняет, почему один рисунок сработал лучше другого.

---

# 14. Карта 5 — «Спаси стаю от удильщика»

## 14.1. Роль в программе

Первая более выраженная physics/replay-карта.

## 14.2. Детская фантазия

> «Стая почти поплыла прямо в пасть — я заметил ловушку и спас всех».

## 14.3. Fun contract

Главная эмоция:

```text
тайна
→ красота
→ тревога
→ забота
→ спасение
```

Fun создают:

- красивый опасный свет;
- огромный хищник долго остаётся скрытым;
- стая реагирует как живой объект;
- есть отставшая рыбка;
- несколько безопасных путей;
- быстрый rescue climax;
- вариативная раскладка.

## 14.4. Learning contract

### Goal

Понять, что светящаяся приманка может быть частью охотничьей стратегии.

### Misconception

«Свет в темноте всегда безопасен и полезен».

### Win requires

Ребёнок распознаёт связь между светом и силуэтом хищника и противодействует притяжению.

### Feedback

Стая начинает тянуться к источнику; опасность становится видимой.

### Transfer

Безопасный и опасный свет различаются по контексту, а не правилу «любой свет плохой».

## 14.5. Production route

Первый `arcade-field` plugin:

```text
controlled actor
attractor/repulsor fields
hazards
safe zones
collectable stragglers
win rule
```

В v1 стая может быть одним составным actor, без сложного flocking.

## 14.6. Acceptance

- plugin используется ещё минимум в одной теме;
- learning не сводится к текстовой подсказке;
- есть реальный replay;
- Quest failure не показывает гибель рыб;
- fixed-step или иной детерминированный режим для тестов.

---

# 15. Карта 6 — «Следовой детектив»

## 15.1. Роль в программе

Недорогая спокойная карта с сильной образовательной ценностью.

## 15.2. Детская фантазия

> «Я увидел только следы, но восстановил событие прошлого».

## 15.3. Fun contract

Главная эмоция:

```text
тайна
→ обнаружение улиц
→ гипотеза
→ большой ghost-reveal
```

Fun создают:

- физическое смахивание слоя;
- улика меняет первоначальную версию;
- финальная реконструкция;
- несколько возможных интерпретаций;
- коллекционный слепок следа.

## 15.4. Learning contract

### Goal

Понять, что следовая дорожка даёт больше информации, чем один отпечаток:

- направление;
- шаг;
- примерный тип движения;
- один или несколько животных.

### Misconception

«По одному следу всегда можно точно назвать вид и всю сцену».

### Win requires

Ребёнок использует последовательность признаков и выбирает осторожную реконструкцию.

### Feedback

Плохая реконструкция явно противоречит следам.

## 15.5. Production route

Reusable `mask-reveal`:

```text
brush
reveal percentage
hotspots
completion threshold
```

Контент:

- raster trackway;
- ground mask;
- hotspots;
- 2–3 ghost reconstruction outcomes.

## 15.6. Acceptance

- никакого ручного skeleton/vector content;
- plugin переносится в археологию, X-ray и hidden-object;
- ребёнок различает наблюдение и гипотезу;
- основной fun не зависит от длинного видео.

---

# 16. Порядок разработки

## Phase 0 — Scaffold

Создать:

- pnpm workspace;
- TypeScript strict;
- schema;
- Stage;
- actor renderer;
- XState adapter;
- plugin interface;
- PWA shell;
- deck pack format;
- offline test fixture;
- asset tooling;
- Playwright harness.

Не строить admin или cloud orchestration.

## Phase 1 — Robot

- `drag-zone`;
- `clip-branch`;
- card spec/manifest split;
- first full asset route;
- first installed offline card.

## Phase 2 — Aqueduct

- `flow-path`;
- reuse drag-zone;
- structural learning;
- outcome sequencing.

## Phase 3 — Echo

- `pulse-scan`;
- audio/visual accessibility;
- config-driven hidden scenes.

## Phase 4 — Cuttlefish

- no new runtime plugin;
- canonical actor consistency;
- image editing states;
- proves factory boundary.

## Phase 5 — Child test of first four

Measure:

- comprehension;
- voluntary replay;
- emotional response;
- learning explanation;
- near transfer;
- production actuals;
- human touches;
- device failures.

## Phase 6 — Anglerfish

- first `arcade-field`;
- stronger physics;
- replay comparison against clip/branch cards.

## Phase 7 — Track Detective

- `mask-reveal`;
- quiet pacing;
- lower-cost content route.

## Phase 8 — Ocean mini-deck

The first natural coherent mini-deck candidate:

```text
Echo
Cuttlefish
Anglerfish
+ 3–5 additional ocean cards
```

This is where we first test:

- guide;
- journal;
- collection;
- within-deck pacing;
- transfer between cards;
- desire to complete the set.

---

# 17. Codex working rules

## 17.1. Source precedence

When documents conflict, use this order:

1. This Project Bible.
2. `05-revised-architecture.md`.
3. `07-clip-pipeline.md`.
4. `EXTERNAL-REVIEW.md`.
5. `reference-pipeline-architecture.md`.
6. `01`–`04` only as idea backlog and historical research.

`08-first-pass-plan.md` and old `09-plain-summary.md` contain an earlier T-Rex/anglerfish/supernova plan. That first-card selection is superseded by this Bible unless explicitly restored by a new decision record.

## 17.2. Before changing code

Codex should:

1. Read root `AGENTS.md`.
2. Read the nearest nested `AGENTS.md`.
3. Inspect relevant schemas and tests.
4. State the object being changed: runtime, plugin, card spec, manifest, tooling, or asset recipe.
5. Identify whether the change is reusable or card-specific.
6. If reusable plugin is proposed, list at least two card uses.
7. Define «done when».
8. Preserve plan/run separation: do not present planned cost or throughput as actual data.

## 17.3. After changing code

Codex should:

- run targeted tests;
- run `pnpm check`;
- add/update tests;
- update schema docs if contract changed;
- update decision record for durable architectural changes;
- report files changed;
- report what was not verified;
- never claim fun or learning was proven without child evidence.

## 17.4. Root `AGENTS.md` invariants

The root file should include rules equivalent to:

```text
- Runtime must never branch on cardId.
- Content directories must not contain runtime TypeScript.
- A new plugin requires two planned card uses.
- Every plugin has a Zod schema and automated tests.
- No manual organic character animation is part of the production method.
- No manual per-card vector-content pipeline.
- Pin framework and tooling versions.
- Do not add dependencies without explaining the reusable need.
- Every behavior change needs a test.
- Run pnpm check before claiming completion.
- Preserve approved manifests and recipes in git.
- Keep generation actuals separate from plans.
- Never use unverified scientific claims as game rules.
```

## 17.5. Nested instructions

Recommended:

```text
packages/runtime/AGENTS.md
packages/plugins/AGENTS.md
packages/tooling/AGENTS.md
content/cards/AGENTS.md
```

`content/cards/AGENTS.md` should explicitly say:

```text
No TypeScript in card folders.
A card may contain specs, manifests, recipes, assets and tests.
If a card requires new behavior, request a reusable plugin rather than adding local code.
```

---

# 18. Командный интерфейс фабрики

Codex должен постепенно привести tooling к таким командам:

```text
pnpm atlas validate <card>
pnpm atlas plan-assets <card>
pnpm atlas process-assets <card>
pnpm atlas build-card <card>
pnpm atlas test-card <card>
pnpm atlas preview-card <card>
pnpm atlas build-pack <deck>
pnpm check
```

На первом этапе это обычные TypeScript scripts через `tsx`.

Не нужна отдельная admin-панель.

---

# 19. Automated QA

## 19.1. Schema and flow

- specs/manifests valid;
- required assets exist;
- all states reachable;
- no dead-end transitions;
- success/failure paths complete;
- pressure profile has fallback;
- plugin configs valid.

## 19.2. Visual and asset

- no cropped actor;
- alpha not obviously broken;
- anchors stable;
- atlas dimensions valid;
- file sizes known;
- video/audio codec inspected;
- preview generated;
- canonical actor identity review requested.

## 19.3. Runtime

- touch works;
- card completes;
- restart works;
- deterministic test seed;
- offline mode works;
- missing pack handled;
- reduced motion path works;
- no more simultaneous decoders than tested;
- memory/FPS captured on target devices.

## 19.4. Learning

Automated checks can only verify structural requirements:

- learning contract present;
- winRequires not empty;
- transfer task defined;
- wrong outcomes mapped.

They cannot prove the child learned.

---

# 20. Human gates

## Gate 1 — Product and learning

Human approves:

- child fantasy;
- emotional arc;
- learning contract;
- factual basis;
- misconception;
- transfer task.

## Gate 2 — Art, safety and IP

Human checks:

- visual consistency;
- unwanted or frightening details;
- obvious third-party likeness;
- alpha quality;
- age suitability.

## Gate 3 — Device and offline

Human checks real target devices:

- low-end Android;
- older iPhone/iPad where available;
- touch;
- audio;
- storage;
- offline restart;
- update/rollback behavior.

## Gate 4 — Child playtest

This is the main product gate.

Observe:

- does the child understand the goal without a lecture;
- does the intended emotion occur;
- does the child voluntarily retry;
- does strategy change after error;
- can the child explain why;
- can the child solve a transfer example;
- does the child say the cards feel the same;
- which card the child chooses again.

Do not rely only on «понравилось?».

---

# 21. Minimal local telemetry

No child account or PII.

Start with local exportable events:

```text
card_started
instruction_repeated
attempt_started
choice_made
wrong_outcome
success
hint_used
retry
quit
duration
transfer_answer
card_selected_again
```

The schema must be versioned.

Cloud analytics is not in the first critical path.

---

# 22. What not to build yet

- full 16–30 card deck;
- animated guide rig;
- Rive/Spine pipeline;
- admin panel;
- Trigger.dev/n8n;
- multi-provider router;
- cloud telemetry;
- auto LLM card factory;
- C2PA orchestration;
- realtime 3D;
- world-model gameplay;
- full AR;
- image recognition of cards;
- Matter slingshot/stack before demand;
- tower defense;
- fluid/cloth/ragdoll;
- free physics sandbox;
- five-language localization;
- daily rewards;
- random rarity/FOMO economy.

Use QR or manual card code for the prototype. Do not mix gameplay validation with computer-vision risk.

---

# 23. Decision records

Durable changes require a short Markdown decision record:

```text
Context
Decision
Alternatives considered
Why chosen
Expected benefit
Known costs/risks
Evidence required
Reopen trigger
Date/status
```

Examples that require a record:

- Phaser version change;
- new plugin family;
- schema breaking change;
- alpha-video accepted as default;
- new storage architecture;
- change in first-card order;
- introduction of a guide;
- introduction of cloud services.

Not every small refactor needs a record.

---

# 24. Success criteria for the first program

## Product success

- children can start without long instruction;
- at least some cards cause voluntary replay;
- intended emotions are observable;
- robot card produces causal explanation;
- aqueduct produces slope explanation;
- echo produces reflection/delay explanation;
- cuttlefish produces pattern/background explanation;
- transfer tasks work better than chance/guessing in the small test sample;
- cards do not all feel like the same drag-and-reward loop.

## Factory success

- no manual character animation;
- no manual per-card vector production;
- no card-specific runtime code;
- Cuttlefish adds no plugin;
- asset processing is scripted;
- manifests build cards;
- tests cover flows;
- cards run offline;
- real generation attempts and human review time are logged;
- Codex can add a second card to a plugin without modifying plugin code.

## Business signal

- physical card feels like a key to an experience, not an advertisement;
- parent can identify learning value;
- child asks what other cards do;
- there is a credible path from cross-theme prototype to coherent mini-deck.

---

# 25. Stop and redesign conditions

Stop or simplify when:

- a plugin takes several days and still serves only one card;
- a card needs `if cardId`;
- AI asset requires manual frame-by-frame repair;
- fun survives only the first video reveal;
- learning can be bypassed through random tapping;
- a failure is frightening but not informative;
- low-end device cannot maintain acceptable interaction;
- offline install is unreliable;
- asset consistency requires manual art production;
- the project starts building infrastructure not needed by current four cards.

---

# 26. Current open questions

These are not decided by architecture alone:

1. Which image/video/matte/TTS providers give the best acceptance rate?
2. Is sprite atlas consistently better than alpha video on target devices?
3. How many generated states per reusable actor are economically justified?
4. Which card creates the strongest voluntary replay?
5. Does `pulse-scan` remain understandable on poor speakers?
6. How much complexity can 7–8-year-olds handle in robot diagnosis?
7. Can Cuttlefish consistency be achieved by image editing without manual repair?
8. What is the true solo throughput after human QA?
9. Which ocean cards best form the first mini-deck?
10. What physical card activation method is reliable enough beyond prototype QR?

Each question must eventually receive evidence, not only an opinion.

---

# 27. Immediate recommended first tasks for Codex

When Андрей asks to begin implementation, the recommended sequence is:

1. Audit the existing repository and POC.
2. Create root `AGENTS.md`.
3. Create a short `PLANS.md` for scaffold.
4. Pin dependency versions.
5. Scaffold pnpm workspace.
6. Define `card.spec` and `card.manifest` schemas.
7. Implement minimal Stage and plugin contract.
8. Implement a fixture card with generated placeholders.
9. Implement deck-pack install/offline smoke test.
10. Implement Robot card before expanding infrastructure.
11. Record actual effort and pain points.
12. Refine tooling only from observed repetition.

Do not begin by implementing all plugins, all six cards or full production orchestration.

---

# 28. One-screen summary

## Product

Physical educational collectible cards that unlock short offline mobile games for children 7–12.

## Promise

The child does something emotionally memorable and uses understanding to influence the result.

## First cards

```text
Robot repair
Roman aqueduct
Echo in darkness
Cuttlefish camouflage
```

Then:

```text
Anglerfish rescue
Track detective
```

## Runtime

```text
TypeScript + Phaser + XState
manifest-driven Stage
reusable plugins
offline deck packs
```

## AI

```text
AI image/video/audio
automatic matte/crop/atlas/encoding
Codex-built plugins and tooling
human approve/reject/regenerate
```

## Hard boundaries

```text
no manual organic animation
no per-card TypeScript
no per-card vector factory
no infrastructure without current need
```

## Proof required

```text
child fun
child learning
cross-theme reuse
no-new-code card
offline reliability
measured human effort
```

---

# 29. Core project maxim

> **AI creates the world, characters, clips, variants and draft configurations.  
> Reusable plugins create control, physics and replay.  
> The manifest assembles the card.  
> Automated tests catch technical failures.  
> A human approves meaning, safety and quality.  
> Real children decide whether it is actually fun and educational.**
