---
name: figma-senior-mode
description: Senior-mindset overlay для любой работы с Figma через MCP. Дополняет официальные figma:* skills (figma-use, figma-implement-design, figma-generate-design). Форсит Design-System Preflight (Token Map + Component Registry), осознанные hug/fill, продумывание states/edge cases, accessibility, и явные clarifying questions ДО рисования. Триггерится на любой задаче с Figma URL/fileKey, упоминанием Figma, или использованием инструментов get_design_context / use_figma / search_design_system / figma:*. НЕ триггерься на чисто читательских задачах (просто открыть/описать файл, выгрузить ассет) или не-дизайнерских упоминаниях Figma — там оверлей не нужен.
license: MIT
---

# Figma Senior Mode

Этот skill — не замена официальным `figma-use` / `figma-generate-design` / `figma-implement-design`. Они уже учат API, gotchas, incremental workflow, error recovery, page rules, fonts. **Их грузить тоже.**

Этот skill добавляет поверх — то, что отличает senior от джуна и что официальные skills не форсят жёстко: **дисциплину design system'а, продумывание states, accessibility, и привычку спрашивать ДО, а не дорисовывать ПОСЛЕ.**

## 0. Когда применять

- Любая задача, где упомянут Figma URL, fileKey, "нарисуй в Figma", "обнови дизайн", "сгенерируй экран"
- Любой вызов `use_figma`, `get_design_context`, `search_design_system`, `get_libraries`, `get_variable_defs`
- Любая design-to-code или code-to-design задача

Если skill активировался — упомяни это одной строкой в начале ответа: `[figma-senior-mode active]`. Это помогает пользователю убедиться, что overlay реально подтянулся.

## 1. Design-System Preflight (обязательно ДО первой write-операции)

Цель — построить внутренний **Token Map** и **Component Registry** до того, как создавать что-либо новое. Без этого ты будешь хардкодить hex'ы и плодить дубли "Button".

**Масштаб — по задаче.** Новый экран или компонент → полный Preflight. Точечная правка одного существующего узла → достаточно свериться с уже собранным Token Map (или собрать только релевантный кусок). Дисциплина одна и та же; меняется объём, а не её наличие.

### Шаги

1. **fileKey.** Из URL пользователя. Нет URL — спроси, не угадывай.
2. **`get_libraries(fileKey)`** — какие библиотеки подключены и доступны для импорта.
3. **`get_metadata(fileKey)`** — структура страниц. Для целевой страницы — иерархия фреймов.
4. **`get_variable_defs(fileKey, nodeId?)`** — все variables и их modes (light/dark/etc).
5. **`search_design_system`** с релевантными ключами (button, card, input, color, spacing, radius, typography) — собери key'и компонентов и токенов под задачу.

### Что построить в голове (и кратко зафиксировать в ответе пользователю)

- **Token Map:** `{semantic_role: variable_key}`
  - colors: `surface/default`, `text/primary`, `border/subtle`, `accent/brand`, etc.
  - spacing: `xs/sm/md/lg/xl` → variable keys
  - radius, typography styles, opacity/elevation если есть
- **Component Registry:** `{logical_role: component_key}`
  - "primary button" → key
  - "text input" → key
  - "card container" → key
- **Что НЕДОСТАЁТ:** если для задачи нужен компонент или токен, которого нет — **СТОП**. Сообщи пользователю и спроси:
  - создать новый (и где: в этом файле / в библиотеке)
  - использовать ближайший существующий
  - пропустить эту часть задачи

**Правило:** пока Preflight не пройден и пробелы не обсуждены, ни одного `use_figma` write-вызова. Это не опционально.

### Token Map шорткат при последующих вызовах

Если в рамках одной сессии Preflight уже выполнен — не перевызывай `get_libraries`/`search_design_system` второй раз для тех же ключей. Используй сохранённый Token Map. Перевызывай только если пользователь сменил файл, или есть подозрение что библиотека обновилась.

## 2. Правила записи на канвас (overlay поверх figma-use)

> `figma-use` уже учит как технически писать через Plugin API. Тут — что писать и чего не писать.

### Variables и Styles — всегда, где доступны
- **Цвета:** только через `setBoundVariableForPaint` (и аналоги). Никаких hex literals в JS, если в Token Map есть подходящий variable.
- **Spacing:** только из существующей шкалы. Нет шкалы в variables — спроси пользователя, не выдумывай "14px".
- **Typography:** text styles из файла. Inline font properties — только если стиля нет и пользователь подтвердил создание нового.
- **Radius, stroke, opacity, elevation** — то же правило, через variables если они существуют.

Исключение допустимо ТОЛЬКО когда: (а) пользователь явно одобрил one-off значение, или (б) variable объективно отсутствует и его создание не входит в задачу.

### Компоненты
- Используй `importComponentByKeyAsync` / `importComponentSetByKeyAsync` для существующих.
- **НЕ создавай** дубликаты Button/Card/Input/Modal, если они есть в Component Registry.
- Variants и component properties — используй уже определённые.
- Если создаёшь новый интерактивный компонент — он должен иметь variants как минимум для default + disabled. Hover/active/focus — спрашивай нужны ли.

### Auto Layout
- Контейнер с >1 child → auto-layout. Дефолтное правило.
- Hug/Fill — осознанный выбор, не дефолт:
  - Кнопка: hug × hug
  - Карточка в гриде: fill width, hug height
  - Контейнер страницы: fill width, hug height
  - Внутренние spacer'ы — fill flex
- Absolute positioning — только для badges/overlays/floating UI, и проговори вслух почему именно abs.

### Семантичные имена слоёв
- Запрещено: `Frame 1247`, `Rectangle 12`, `Group 8`, `Container`, `Wrapper`.
- Формат: `Card / Header`, `Button / Primary / Default`, `Input / Email / Error`, `Modal / Body / Section`.
- Иерархия в имени = иерархия в дизайне.

## 3. States и edge cases (то, что джун пропускает)

Для каждого экрана/компонента — пройди этот чек-лист в голове. Что не покроешь — **явно** проговори пользователю ("пропустил empty state, нужен?"), не молчи.

- Empty state (нулевой контент, первый запуск)
- Loading state (skeleton/spinner)
- Error state (network fail, validation fail)
- Long-text overflow (тестовые строки 60+ символов, многоязычность)
- Минимальный и максимальный контент в списках/гридах
- Hover / active / disabled / focus для интерактивных
- Dark mode — если в файле есть variable modes, используй их, не дублируй экран
- Responsive — если задача мобильная, спрашивай breakpoints

## 4. Accessibility

- **Контраст:** WCAG AA — 4.5:1 для обычного текста, 3:1 для large (≥18pt regular или ≥14pt bold) и UI components. Если bound variables в нескольких modes — посчитай контраст для каждого mode, не только default.
- **Tap targets:** ≥ 44×44pt на мобильных. Икон-кнопка 24px → padding минимум 10px со всех сторон.
- **Focus indicators:** для всех интерактивных элементов в keyboard-flow.
- **Иерархия текста:** один H1 на экран, последовательная семантика, не "большой жирный = заголовок".

Когда не можешь физически проверить (например, нет инструмента контраста в этом потоке) — проговори: "контраст визуально приемлемый, формально не верифицировал".

## 5. Workflow: design-to-code (`get_design_context`)

> Базовая часть покрыта в `figma-implement-design`. Тут — senior overlay.

1. `get_design_context(fileKey, nodeId)` — код + скриншот.
2. Сверь полученные токены с теми, что в **кодовой базе** (поищи существующие design tokens / CSS variables / Tailwind config).
3. Используй имена и конвенции проекта (Tailwind classes / CSS modules / styled-components / vanilla-extract — смотри как уже сделано рядом).
4. **НЕ копируй** inline-стили в код напрямую — выноси в существующие токены кодовой базы. Если токена нет — спроси: добавить в config или захардкодить разово.
5. Сверь spacing/radius/typography с тем, что figma говорит. Если расхождения — это сигнал, что design system либо в Figma, либо в коде ушёл вперёд. Не "усредняй" молча.

## 6. Workflow: code-to-design (`use_figma`)

> Базовые правила Plugin API — в `figma-use`. Workflow сборки экрана — в `figma-generate-design`. Тут — senior overlay.

1. **Preflight (§1) ОБЯЗАТЕЛЕН.** Не "сейчас быстро накидаю и потом причешу".
2. **Один `use_figma` ≠ 500 строк JS.** Разбивай на проверяемые куски: variables → components → composition. `figma-use` это тоже требует, тут это просто напоминание.
3. После каждого значимого куска — `get_screenshot` соответствующего node. Визуально верифицируй.
4. Если результат не сходится с ожиданием — **фикси, не оставляй "ну и так сойдёт"**. И не накапливай долг типа "три мелких проблемы, разберу в конце".
5. Перед "готово" — пройди чек-лист §9.

## 7. Когда спрашивать пользователя (НЕ молчать)

Эти ситуации почти всегда требуют clarifying question ДО рисования, потому что цена пере-рисовать выше цены вопроса:

- Mobile или desktop? (если не очевидно из контекста)
- Новый экран или вариант существующего?
- Какая breakpoint-сетка / grid система?
- Темизация: light only или есть dark mode?
- Локализация: ожидать ли длинные строки (нем/ру/ар)?
- Брендовая тональность, если её нет в design system (играет роль для emoji/иллюстраций/тона текста)
- В какой файл/страницу складывать результат
- Кто целевой пользователь и что у него уже есть на этом экране (для empty/loading)

**Эвристика:** если в голове появилось "наверное он имел в виду..." — это вопрос, а не предположение.

## 8. Чего НЕ делать (анти-паттерны)

- **Не лочить размеры там, где нужен hug.** Это самый частый источник "почему карточка не растягивается".
- **Не плодить one-off** цвета, spacing'и, шрифты. Это медленная порча design system'а.
- **Не молчать про неоднозначности.** См. §7.
- **"Inter SemiBold"** — неправильно. Правильно: `"Inter Semi Bold"` (с пробелом). Same для Extra Bold, Extra Light. Используй `figma.listAvailableFontsAsync()` если сомневаешься (см. figma-use §1.8).
- **`figma.currentPage = page`** — не работает. Только `await figma.setCurrentPageAsync(page)`. (Это в figma-use §1.9, но напоминаю — джуны это путают каждый раз.)
- **`getPluginData/setPluginData`** — не работает в `use_figma`. Только `getSharedPluginData(namespace, key)` с уникальным namespace. (figma-use §1.3a.)
- **Не пихать всё в один `use_figma`-скрипт.** Atomic execution + сложность отладки = провал. Маленькими шагами.
- **Не игнорировать ошибки `use_figma`** в духе "ща ещё раз попробую". `figma-use` чётко требует: STOP → read error → fix → retry.

## 9. Чек-лист перед "готово"

- [ ] Preflight выполнен, Token Map и Component Registry собраны
- [ ] Все цвета через variables (или явно одобренные пользователем exceptions)
- [ ] Все spacing'и из шкалы
- [ ] Все text — через text styles
- [ ] Использованы существующие компоненты библиотеки, где они подходят
- [ ] Auto-layout настроен, hug/fill осознанно (можешь объяснить выбор для каждого фрейма)
- [ ] Имена слоёв читаемые и иерархичные
- [ ] States продуманы (или явно проговорено что пропущено и почему)
- [ ] Контраст проверен (или проговорено что не верифицировано)
- [ ] `get_screenshot` показывает то, что ожидалось
- [ ] Возвращены все созданные/изменённые node IDs (требование figma-use §1.15)
- [ ] Никаких write-операций в файлы, на которые пользователь явно не дал ОК
