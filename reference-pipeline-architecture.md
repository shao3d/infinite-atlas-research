# Infinite Atlas — сквозной AI-конвейер (intake → печать + мобильный интерактив)

> Архитектура, блоки/коннекторы, «пульт» оператора, минимальные человеческие точки и buy-vs-build.
> Построение и управление через OpenAI Codex.

---

## Главный вывод: готового бэкбона нет — строим тонкий спайн Codex'ом, адаптируем point-tools

Ни одна платформа не закрывает конвейер целиком. «Контент-фабрики» (Scenario.gg, Layer.ai, Rosebud) — это **только узлы генерации арта**: они не знают про игровую логику, не делают препресс и не собирают PWA. Ядро IP (параметрический **Шаблон × контент-схема × два выхода**) никто не продаёт. Поэтому архитектура одна:

> **BUILD тонкий спайн (Codex) + ADOPT готовые куски на каждый тяжёлый шаг.**
> Поэтапно: начни с самого дешёвого, наращивай по мере боли — слои **композируются**, это не «или-или».

---

## 1. Схема конвейера: блоки и как они соединяются

```
┌──────────── ОДИН источник истины (git manifest.json + card.json) ────────────┐
│ status: intake→content→art→mechanic→assets→qa→approved→print-exported→        │
│         app-bundled   + provenance + cost                                     │
└───────────────────────────────────────────────────────────────────────────────┘

[INTAKE]  тема + стиль + механика
   │  (форма / Telegram → REST trigger)
   ▼
[CONTENT FILL]  LLM заполняет схему (факты = first-class, с цитатой + verified_by)
   │  OpenAI / Agents-SDK, Helicone-proxy
   ▼
[CONTENT QA]  Zod-валидно + NSFW/CLIP + safety-judge          ── auto-pass / escalate
   ▼
[CARD ART]  master (GPT Image direct) + per-card batch (fal: Flux/SD)
   │  REST submit → webhook (НЕ polling)
   ▼
[IMAGE QA]  Pillow / CLIP / NSFW / стиль
   │  subprocess; байты → R2, в манифест только sha256 + URL
   ▼
[MECHANIC CONFIG]  ← ТВОЁ ЯДРО: параметрический блок в манифесте, кормит ОБА выхода
   ▼
[PER-CARD ASSETS — роутинг по механике]
   │  2D / Rive / sprite-sheet / AI-video-alpha / параметрика / glTF
   │  через fal.ai (Kling 3.0 i2v, Wan, 3D); idempotency = hash(card+stage+style+content)
   ▼
[MATTE / ENCODE]  MatAnyone2 + ffmpeg stacked-alpha + gltf-validator
   │  subprocess (или fal green-screen, если нет GPU)
   ▼
[ASSET QA]  ffprobe + vision-LLM judge + gltf-validator + Playwright  ── auto-pass / escalate
   ▼
[PROVENANCE]  C2PA-подпись каждого ассета (EU AI Act Art.50)
   │
   ├──► [PRINT EXPORT]  JSON → 822×1122@300dpi (react-pdf, K-only black)
   │         → press-ready / Ghostscript PDF/X-1a CMYK (FOGRA39) → preflight
   │         └──► ⏸ ЧЕЛОВЕК: print-proof (единственный необратимый шаг) ──► в типографию
   │
   └──► [PWA BUILD]  per-deck Vite + Workbox precache
             ──git push──► Cloudflare Pages (авто, Playwright-гейт, БЕЗ человека)

[сквозное]  Langfuse (трейс + стоимость)  +  deck_ledger БЮДЖЕТ-ПРЕДОХРАНИТЕЛЬ (твой, не провайдера)
```

### 🔑 Золотое правило коннекторов

Всё дорогое и долгое (gen image/video/3D, matte) — **async submit → webhook resume** через waitpoint.
**Никогда** блокирующий вызов, **никогда** внутри Codex-turn (его tool-timeout = 60 с, ломается на видео), **никогда** через MCP, **никогда** внутри n8n-ноды (n8n таймаутит >5 мин). Стадии передают управление через статус-поле / очередь. **MCP — только тонкий пульт управления** (запросить статус / перезапустить стадию / одобрить), не data-path.

---

## 2. Спайн (оркестратор): фазированно, A → B → C

| Фаза | Что | Когда брать | Почему |
|---|---|---|---|
| **A (v1)** | **Git-as-truth harness**: manifest-машина состояний + cron + идемпотентные скрипты + `openai/codex-action` | **СТАРТ** | Паттерн самих OpenAI («0 строк ручного кода»). Ноль новой инфры, max автономия Codex, манифест = аудит-леджер для EU AI Act. Resumability бесплатно (skip где `status==done && sha256` совпал) |
| **B** | **Trigger.dev v4** (durable spine) | когда краш в середине колоды переплачивает за уже сгенерённое, или нужны настоящие waitpoint'ы | `wait.createToken/forToken` — **один примитив и для медленного gen-callback, и для человеческого аппрува**: без polling, на паузе не биллится, при ретрае не перегенерит готовое. Plain TS без determinism-правил → Codex-код «just works». Тот же код Cloud ↔ self-host |
| **C** | **n8n + Telegram «Send and Wait»** + тонкий **Next.js + shadcn admin** | когда одобрять с ноута/PR стало неудобно и нужны богатые превью | approve-с-телефона в один тап; rich-превью (alpha-video над шахматкой, glTF, live-PWA iframe) |

**Отвергнуто (с обоснованием):** Temporal (determinism воюет с AI-кодом + тяжёлая эксплуатация), raw BullMQ (придётся самому строить waitpoints/observability/бюджет), Supabase-as-truth ($25/проект бьёт solo + вес), Retool-as-core (per-seat + JSON-lock-in), и **любые провайдерские spend-caps** (см. ниже).

---

## 3. Истина, бюджет, провенанс — три неотменяемых правила

1. **Источник истины = git-манифест** (diffable, PR-gated, Codex-нативный). БД (Postgres/PocketBase) — только **регенерируемое зеркало** для UI, никогда наоборот. **Байты — никогда в git** → R2/S3 по `sha256 + URL` (или DVC-указатели).

2. **Бюджет-предохранитель живёт в ТВОЁМ леджере**, не у провайдера. ⚠️ Критично: OpenAI org/project-капы в 2026 ненадёжны (есть репорты, как $1000-кап пробило в 2×, показывая «зелёный»), а у **fal/Replicate/Kling жёсткого капа нет вообще**. Решение: строка `deck_ledger {cap_usd, spent_usd, status}`, **pre-flight-проверка перед каждым метеред-вызовом** → при превышении `throw BUDGET_EXCEEDED` + пауза очереди.

3. **Провенанс/комплаенс:** C2PA-подпись каждого сгенерённого ассета на шаге QA (`c2patool` / `c2pa-python`). **EU AI Act Art.50 с 2 авг 2026** (штрафы до €15M / 3%): C2PA закрывает машиночитаемый слой, но **не** невидимый вотермарк (нужен SynthID-class), и C2PA **стирается при растеризации в CMYK-PDF** → на физической карте нужен видимый AI-лейбл. *(Лайфхак: US-only старт откладывает этот комплаенс.)*

---

## 4. Два конца конвейера (самые «жёсткие» места)

**Печать** (наименее автоматизируемое звено): `react-pdf/Puppeteer → press-ready (Ghostscript)` = PDF/X-1a CMYK, бесплатно, скриптуемо. **НЕ** InDesign Server (тяжёлая лицензия). ⚠️ Ghostscript при RGB → FOGRA39 **теряет чистый чёрный (K)** на тексте/линиях → верстай **чёрный как K-only + overprint** и подавай **точный ICC твоей типографии**. **Возьми ICC у типографии и физический пруф ДО автоматизации** — это и есть твой единственный необратимый человеческий гейт.

**Приложение** (решённая задача): per-deck `Vite + vite-plugin-pwa/Workbox` precache → `git push` → **Cloudflare Pages** (дешевле всех по трафику). Человеческого гейта нет — только авто Playwright-скриншот. Capacitor-обёртка опциональна (только под EU-iOS DMA или магазин).

---

## 5. Твой «пульт» — это композиция, а не одно приложение

| Слой | Что | Build/Buy |
|---|---|---|
| **Статус-доска (v1)** | Codex-генерируемый **статический дашборд** из манифестов (грид карт + статус + стоимость + провенанс), пересобирается на каждый push | build (генерится) |
| **Богатые превью (потом)** | тонкий **Next.js 16 + shadcn admin** — единственное, что честно рисует **все 4 типа**: картинка, **alpha-video над шахматкой** (переиспользуешь свой Pixi-шейдер), **glTF** (model-viewer + вывод gltf-validator), **живая мини-игра в iframe** | **build** (off-the-shelf билдеры это не умеют → embed кастома всё равно) |
| **Approve-с-телефона** | **Telegram «Send and Wait»** (HMAC-ссылки + expiry). Gmail — fallback. **Slack избегать** (баг кнопки в n8n) | adopt |
| **Ops-консоль** | дашборд **Trigger.dev / n8n** (трейс, replay, пауза очереди) | adopt |
| **Стоимость/качество** | **Langfuse** (self-host) | adopt |
| **Escape hatch** | **Baserow/Directus** грид над той же БД (правка данных, если кастом-UI «сгниёт») | adopt |

**Retool как ядро — нет** (per-seat + lock-in, и 4 типа превью он всё равно не рисует). Держи admin **намеренно тонким** (view над БД, вся логика — в спайне/скриптах), иначе он за полгода превратится в «internal-tool liability».

---

## 6. Минимальные человеческие точки: 3 (+1 лёгкая)

| Гейт | Когда | Обязателен? |
|---|---|---|
| **G-Intake** | один креативный ввод: тема + стиль + механика | да (это и есть твоя работа) |
| **G-Safety + Likeness/IP** | на арте: факты, детская безопасность, сходство, не-нарушение IP | **да, на каждую карту** для детского продукта (vision-LLM-судья только пред-фильтр; до заключения юриста считай «человек-на-каждую-карту») |
| **G-Print-proof** | PDF/X-1a + preflight перед типографией | да — **единственный необратимый шаг** |
| G-Play-test | iframe живой PWA-карты | лёгкий, амортизированный |

Всё машинное QA (Pillow/CLIP/NSFW, gltf-validator, ffprobe+vision-LLM, Playwright) — **auto-pass, эскалация к человеку только на фейле**. Деплой PWA — без гейта.

---

## 7. Codex-роль и buy-vs-build (сжато)

**Codex = только build-and-maintain** через `openai/codex-action` (gated PR, вложенные `AGENTS.md` по подсистемам). **Никогда не рантайм карт** — метеред-gen всегда скрипты по триггеру очереди, не Codex-turn. ⚠️ Проверь: разрешает ли $200-план headless/SDK-автоматизацию под подписочной аутентификацией или нужен отдельный metered API-биллинг (открытый вопрос).

| Блок | Вердикт |
|---|---|
| **Спайн** | ADOPT Trigger.dev v4 (или n8n визуально). Не строить свой, не Temporal/BullMQ |
| **Source of truth** | BUILD тонко: git manifest + Zod/JSON-Schema. Не хостинг-БД как истину |
| **Model gateway** | ADOPT fal.ai (Kling/Wan/Flux/3D, один async API) + GPT Image direct. Replicate — fallback |
| **Matte/QA/prepress** | ADOPT CLI (MatAnyone2, ffmpeg, gltf-validator, c2patool) + build тонкий glue |
| **Print** | ADOPT press-ready/Ghostscript + build react-pdf-шаблон |
| **App** | BUILD Vite-PWA + ADOPT Cloudflare Pages |
| **Пульт** | BUILD тонкий Next.js admin + ADOPT Trigger.dev/n8n dashboard + Telegram |
| **MCP** | BUILD один маленький «Atlas MCP» (~6 узких тулов: `start_card`, `get_status`, `list_failed_qa`, `retry_stage`, `request_print_export`, `approve_for_print`) + adopt GitHub/R2/Telegram MCP. Только control-surface |
| **Budget kill-switch** | BUILD (провайдерам нельзя доверять) |
| **Observability** | ADOPT Langfuse + Helicone-proxy на OpenAI-текст |

---

## 8. ⚠️ Честный вердикт критика: для v1 это переинженерено

> «Realistic but over-built for v1. Build a thin spine, adopt point tools, **prove one card end-to-end first**.»

**Что строить на этой неделе (не раньше платформы):**

1. Сначала **два конца на ОДНОЙ карте, вручную**: `manual JSON → press-ready PDF/X-1a (K-only) → физический пруф у типографии`; и `JSON → Vite PWA → Cloudflare Pages`. Это де-рискует расходящиеся концы до генеративной середины.
2. Арт — **один image-вызов, статика** (видео/3D отложи).
3. **Restartable fan-out скрипт + kill-switch ДО любого fan-out.**
4. **3 гейта** на статической странице.
5. **ICC у типографии — взять сейчас.**
6. **US-only первым** — откладывает EU-комплаенс.
7. **Repo = истина**, манифест держи «тупым».
8. **Пороги апгрейда задай числами** (декад/неделю → Trigger.dev; трение аппрува → n8n+Telegram).
9. **Пока без MCP, без Langfuse, без кастом-admin** — только когда заболит.
10. **Агент строит, не гоняет карты.**

---

### Дальше можно собрать
- **(а)** скелет репозитория v1 (структура папок + `AGENTS.md` + Zod-схема манифеста + статус-машина) — готовый ТЗ-стартер под Codex;
- **(б)** детальный план «одна карта end-to-end» (конкретные команды press-ready/Ghostscript, Vite-PWA, kill-switch) — то, что критик советует сделать первым;
- **(в)** one-pager «архитектура + buy/build + минимальные гейты» для Степана.
