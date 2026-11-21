# Практика: командный сайт на Vue 3 + Git + GitHub Actions + GitHub Pages

**Формат:** групповая работа. Вся группа работает в одном репозитории и делает один сайт.  
**Стек:** Vue 3 (Composition API, `<script setup>`), Vite, TypeScript, Vue Router.  
**Главная цель:** освоить Git, ветки, слияние, pull request и CI/CD на GitHub. Сам сайт — полигон для этого.

> В примерах используется ветка `main`. Если в вашем репозитории основная ветка называется `master`, замените `main` на `master` во всех командах и workflow.

---

## 1. Что должно получиться

Один публичный сайт на GitHub Pages. В нём два типа страниц:

1. **Главная `/`** — сетка карточек разработчиков. Каждая карточка содержит краткую информацию и ведёт на личную страницу автора.
2. **Страница разработчика `/developers/<slug>`** — расширенная информация о студенте. Оформление свободное.

Ключевое условие: **карточка — один общий компонент**. Техлид создаёт его один раз, все студенты используют его с разными данными. Формат карточек одинаковый, личные страницы — разные.

```
Главная / [карточка 1] [карточка 2] [карточка 3] ...
                |            |            |
                v            v            v
        /developers/ivan  /developers/anna  /developers/oleg
```

---

## 2. Роли

| Роль | Кто | Что делает |
|---|---|---|
| **Техлид** | Один студент | Создаёт репозиторий и проект, общий компонент карточки, главную страницу, пример личной страницы и маршрута. Настраивает GitHub Actions, GitHub Pages и защиту `main`. Приглашает группу, показывает механизм. |
| **Студент** | Каждый участник | Создаёт свою ветку, добавляет карточку на главную, делает личную страницу и маршрут, открывает PR, проходит CI, мержит и проверяет деплой. |
| **Преподаватель** | — | Принимает работу, проверяет понимание механизма по вопросам из раздела 9. |

Техлид тоже добавляет свою карточку и страницу. Он не делает карточки за других.

---

## 3. Техлид: создать проект и инфраструктуру

### 3.1. Репозиторий и первый push

1. Создайте **пустой публичный** репозиторий на GitHub — без README, `.gitignore` и лицензии. Иначе первый push может конфликтовать с автоматическим коммитом GitHub.
2. Локально создайте проект (3.2), сделайте первый commit и push в `main`.
3. Добавьте студентов в Collaborators (или в команду организации с правом Write).
4. Создайте общий компонент, главную страницу, пример личной страницы и настройте `base` (3.3–3.6).
5. Настройте CI/CD и GitHub Pages (3.7–3.8), запушьте workflow и дождитесь первого зелёного деплоя.
6. **После первого успешного деплоя** защитите `main`:
   - Settings → Branches → Add branch protection rule (или Rulesets);
   - require a pull request before merging;
   - require 1 approval;
   - require status checks: `build` (в списке может отображаться как `CI / build`);
   - require branches to be up to date;
   - запретить force push и удаление ветки.

Прямой push в `main` должен быть невозможен. Все изменения — только через PR. Защита включается после первого push, иначе техлид не сможет залить исходный каркас.

Если чек `build` ещё не появлялся в списке доступных, сначала откройте тренировочный PR (например, из ветки `chore/test-ci`), дайте CI отработать, затем добавьте `build` в required checks и закройте PR.

### 3.2. Каркас Vue 3 + Vite + TypeScript

Нужен Node.js 22 LTS или новее.

```bash
npm create vite@latest team-site -- --template vue-ts
cd team-site
npm install
npm install vue-router
npm install -D @types/node
npm run dev
```

Первый commit и push:

```bash
git init
git add .
git commit -m "chore: init Vue 3 project"
git branch -M main
git remote add origin <url-репозитория>
git push -u origin main
```

Убедитесь, что `package-lock.json` закоммичен: без него `npm ci` в CI не сможет установить зависимости.

Структура:

```
src/
├─ components/
│  └─ DeveloperCard.vue
├─ views/
│  ├─ HomeView.vue
│  └─ developers/
│     └─ ExampleView.vue
├─ router/
│  └─ index.ts
├─ App.vue
├─ main.ts
└─ style.css
```

### 3.3. Общий компонент карточки

`DeveloperCard.vue` — единственный компонент карточки. Студенты его **не меняют**, а только передают свои данные.

| Проп | Тип | Обязателен | Что это |
|---|---|---|---|
| `name` | `string` | да | Имя или ФИО |
| `role` | `string` | да | Например, «Frontend-разработчик» |
| `shortDescription` | `string` | да | 1–2 предложения о себе |
| `avatar` | `string` | нет | Путь к фото; если нет — инициалы |
| `to` | `RouteLocationRaw` | да | Маршрут личной страницы |

Минимальный пример:

```vue
<!-- src/components/DeveloperCard.vue -->
<script setup lang="ts">
import type { RouteLocationRaw } from 'vue-router'

defineProps<{
  name: string
  role: string
  shortDescription: string
  avatar?: string
  to: RouteLocationRaw
}>()
</script>

<template>
  <RouterLink class="card" :to="to">
    <img v-if="avatar" class="card__avatar" :src="avatar" :alt="name" />
    <span v-else class="card__avatar card__avatar--initials" aria-hidden="true">
      {{ name.charAt(0) }}
    </span>

    <h3 class="card__name">{{ name }}</h3>
    <p class="card__role">{{ role }}</p>
    <p class="card__description">{{ shortDescription }}</p>
    <span class="card__more">Подробнее →</span>
  </RouterLink>
</template>

<style scoped>
.card {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 20px;
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  color: inherit;
  text-decoration: none;
  background: #fff;
}
.card__avatar {
  width: 96px;
  height: 96px;
  border-radius: 50%;
  object-fit: cover;
}
.card__avatar--initials {
  display: grid;
  place-items: center;
  background: #deeaf6;
  color: #1f3864;
  font-size: 28px;
  font-weight: 700;
}
.card__name { margin: 4px 0 0; font-size: 20px; }
.card__role { margin: 0; color: #2e74b5; font-weight: 600; }
.card__description { margin: 0; color: #475569; }
.card__more { margin-top: auto; padding-top: 8px; color: #1f3864; font-weight: 700; }
</style>
```

Вся карточка — ссылка `RouterLink`. Внутри не должно быть вложенных ссылок и кнопок.

### 3.4. Главная страница

`HomeView.vue` — сетка карточек. Каждый студент добавляет сюда **одну** карточку в конец списка.

```vue
<!-- src/views/HomeView.vue -->
<script setup lang="ts">
import DeveloperCard from '../components/DeveloperCard.vue'
</script>

<template>
  <main class="home">
    <h1>Наша команда</h1>

    <section class="grid">
      <!-- Пример. Техлид заменяет эту карточку своей, студенты добавляют свои ниже. -->
      <DeveloperCard
        name="Иван Петров"
        role="Frontend-разработчик"
        short-description="Пример карточки: краткая информация о разработчике."
        :to="{ name: 'example' }"
      />
    </section>
  </main>
</template>

<style scoped>
.home { max-width: 1200px; margin: 0 auto; padding: 32px 20px 64px; }
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
}
</style>
```

### 3.5. Пример личной страницы и маршрута

Создайте `src/views/developers/ExampleView.vue` — образец для студентов. Внутри минимум три блока:

- «Обо мне»;
- «Навыки и проекты»;
- «Контакты / увлечения».

Добавьте маршрут в `src/router/index.ts`:

```ts
import { createRouter, createWebHashHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'

export const router = createRouter({
  history: createWebHashHistory(import.meta.env.BASE_URL),
  routes: [
    { path: '/', name: 'home', component: HomeView },
    {
      path: '/developers/example',
      name: 'example',
      component: () => import('../views/developers/ExampleView.vue'),
    },
  ],
})
```

Карточка-пример в `HomeView.vue` ведёт на маршрут `example`. Техлид заполняет эту карточку и страницу своими данными — они же служат образцом для студентов.

Подключите роутер в `src/main.ts`:

```ts
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
import { router } from './router'

createApp(App).use(router).mount('#app')
```

Замените содержимое `src/App.vue` на:

```vue
<template>
  <RouterView />
</template>
```

Удалите из `App.vue` демо-компонент `HelloWorld`, иначе он останется на всех страницах. В `src/style.css` оставьте только базовые стили (шрифт, фон, `box-sizing`): демо-стили шаблона (`#app { max-width: 1280px; display: flex; ... }`) ломают сетку карточек.

Почему `createWebHashHistory`, а не `createWebHistory`: GitHub Pages — статический хостинг. При прямом заходе на `/developers/ivan` он ищет файл и отдаёт 404. Hash-режим (`/#/developers/ivan`) работает без серверных настроек.

### 3.6. База для GitHub Pages

В `vite.config.ts`:

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

const repo = process.env.GITHUB_REPOSITORY?.split('/')[1]
const base = !repo || repo.endsWith('.github.io') ? '/' : `/${repo}/`

export default defineConfig({
  base,
  plugins: [vue()],
})
```

Два варианта URL:

| Имя репозитория | URL сайта | `base` |
|---|---|---|
| `<user>.github.io` | `https://<user>.github.io/` | `/` |
| `<team>-site` | `https://<user>.github.io/<team>-site/` | `/<team>-site>/` |

Если `base` неверный, сайт откроется пустым: JS и CSS будут грузиться по неправильному пути.

### 3.7. CI: проверка pull request

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run build
```

CI запускается на каждый PR в `main`. После настройки branch protection красный CI блокирует merge.

### 3.8. CD: публикация сайта

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/configure-pages@v5
        with:
          enablement: true
      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

Закоммитьте оба workflow-файла и запушьте в `main`. Без этого GitHub не увидит их, а пункт **Run workflow** не появится.

Затем:

1. Settings → Pages → Build and deployment → **Source: GitHub Actions**.
2. После push `deploy.yml` запустится сам. Если нет — Actions → Deploy to GitHub Pages → Run workflow.
3. Дождитесь зелёного деплоя и откройте URL.

`deploy.yml` запускается только после merge в `main`. PR-код на сайт не попадает.

### 3.9. README и демо

В README репозитория укажите:

- URL сайта;
- как запустить локально (`npm ci`, `npm run dev`);
- правило веток и PR;
- кто техлид.

Проведите короткое демо (10–15 минут): покажите карточку, страницу, PR, зелёный CI, merge и обновление сайта. После этого студенты начинают свои ветки.

---

## 4. Каждый студент: добавить себя

### Шаг 1. Подготовка

```bash
git clone <url-репозитория>
cd <repo>
npm ci
npm run dev
```

Откройте `http://localhost:5173` и убедитесь, что главная страница и страница-пример открываются.

### Шаг 2. Своя ветка

```bash
git switch main
git pull
git switch -c feature/<фамилия-имя>
```

Пример: `feature/ivan-petrov`.

### Шаг 3. Карточка на главной

Откройте `src/views/HomeView.vue` и добавьте **в конец** списка карточек свою:

```vue
<DeveloperCard
  name="Иван Петров"
  role="Frontend-разработчик"
  short-description="Пишу интерфейсы на Vue 3, люблю TypeScript и чистый CSS."
  :to="{ name: 'ivan-petrov' }"
/>
```

Если есть фото, положите его в `src/assets/`, импортируйте в `HomeView.vue` и передайте в карточку:

```ts
import avatar from '../assets/ivan.jpg'
```

```vue
<DeveloperCard
  name="Иван Петров"
  role="Frontend-разработчик"
  short-description="Пишу интерфейсы на Vue 3, люблю TypeScript и чистый CSS."
  :avatar="avatar"
  :to="{ name: 'ivan-petrov' }"
/>
```

Если фото нет, компонент покажет инициалы.

### Шаг 4. Личная страница

Создайте `src/views/developers/IvanPetrovView.vue`. Оформление свободное, но должны быть:

- имя и краткая шапка;
- блок «Обо мне»;
- блок «Навыки и проекты»;
- блок «Контакты / увлечения».

Не копируйте чужие страницы и не используйте чужие фото.

### Шаг 5. Маршрут

В `src/router/index.ts` добавьте маршрут **в конец** массива `routes`:

```ts
{
  path: '/developers/ivan-petrov',
  name: 'ivan-petrov',
  component: () => import('../views/developers/IvanPetrovView.vue'),
},
```

Имя маршрута (`name`) должно совпадать с тем, что вы указали в `:to` у карточки.

### Шаг 6. Проверка и коммит

```bash
npm run build
git add src/views/HomeView.vue src/router/index.ts src/views/developers/IvanPetrovView.vue
git add src/assets/   # если добавляли фото
git commit -m "feat: add Ivan Petrov card and page"
git push -u origin feature/ivan-petrov
```

Сделайте 2–3 осмысленных коммита, если удобно. Не коммитьте `node_modules`, `dist` и `.env`.

### Шаг 7. Pull request

Откройте PR из своей ветки в `main`. В описании укажите:

- что добавлено;
- ожидаемый URL страницы (например, `https://<user>.github.io/<repo>/#/developers/ivan-petrov`); фактическую ссылку добавите в отчёт после деплоя;
- скриншот главной с вашей карточкой и скриншот личной страницы;
- чек-лист: сборка проходит, чужие карточки и маршруты не удалены.

### Шаг 8. CI

Дождитесь проверки. Если CI красный:

1. Actions → упавший run → job `build` → шаг с ошибкой.
2. Прочитайте ошибку.
3. Исправьте локально, сделайте коммит и push.
4. Добейтесь зелёного CI.

Пока CI красный, merge заблокирован — это нормально.

### Шаг 9. Конфликты

Так как все редактируют `HomeView.vue` и `router/index.ts`, конфликты **ожидаемы**. Это часть задания.

Если GitHub пишет, что ветка конфликтует с `main`:

```bash
git fetch origin
git switch feature/ivan-petrov
git rebase origin/main
```

Откройте конфликтные файлы. В `HomeView.vue` оставьте **все** карточки, включая чужие. В `router/index.ts` оставьте **все** маршруты. Уберите маркеры `<<<<<<<`, `=======`, `>>>>>>>`, затем:

```bash
git add src/views/HomeView.vue src/router/index.ts
git rebase --continue
git push --force-with-lease
```

Если при `git rebase --continue` откроется редактор с сообщением коммита, сохраните и закройте его.

Не удаляйте чужую карточку или маршрут, чтобы «быстрее смержить».

### Шаг 10. Merge и проверка деплоя

После approval (его ставит техлид или другой студент; свой PR approve нельзя) и зелёного CI:

1. Нажмите **Squash and merge**.
2. Удалите ветку.
3. Дождитесь зелёного **Deploy to GitHub Pages**.
4. Откройте сайт и проверьте свою карточку и страницу.

---

## 5. Как это работает: CI/CD

| Действие | Что запускается | Результат |
|---|---|---|
| Push в `feature/...` | Ничего автоматически | Ветка появилась на GitHub |
| Открытие/обновление PR в `main` | `ci.yml`: `npm ci`, `npm run build` | Красный CI блокирует merge |
| Merge PR в `main` | `deploy.yml`: build, artifact, deploy | Сайт обновляется на GitHub Pages |
| Ошибка деплоя | — | Сайт остаётся на предыдущей версии |

Главная ветка `main` — это ветка публикации. Прямой push в неё запрещён. Поэтому изменение попадает на сайт только так:

```
ветка студента → PR → review → зелёный CI → merge в main → deploy → GitHub Pages
```

Именно эту цепочку нужно уметь объяснить.

---

## 6. Правила Git

- Один студент — одна ветка — один PR.
- Ветка: `feature/<фамилия-имя>`.
- Коммиты осмысленные: `feat: add Ivan Petrov card`, `feat: add Ivan Petrov page`, `fix: resolve route conflict`.
- PR только в `main`, после зелёного CI и approval.
- Merge — **Squash and merge**, ветку удалить.
- Не удалять и не искажать чужой контент: при конфликте сохраняйте все карточки и маршруты.
- Не коммитить `node_modules`, `dist`, `.env`, секреты и большие файлы.
- Не делать force push в `main`. `--force-with-lease` — только в своей ветке после rebase.

---

## 7. Что сдать

| Артефакт | Где взять |
|---|---|
| Ссылка на PR | GitHub → Pull requests |
| Ссылка на зелёный CI run | PR → Checks → Actions |
| Ссылка на страницу | `https://.../#/developers/<slug>` |
| Ссылка на deploy run | Actions → Deploy to GitHub Pages |
| Ответы на вопросы | Раздел 9 |

---

## 8. Критерии приёмки

- [ ] Сайт открывается по публичному URL.
- [ ] На главной видны карточки всех студентов.
- [ ] Каждая карточка ведёт на личную страницу автора.
- [ ] На личной странице есть расширенная информация (3+ блока).
- [ ] `main` защищён: прямой push невозможен.
- [ ] Все изменения прошли через PR и review.
- [ ] CI на PR зелёный, deploy на `main` зелёный.
- [ ] Сайт обновился после merge.
- [ ] В репозитории нет секретов, `node_modules` и `dist`.
- [ ] Студент может объяснить путь своего изменения до сайта.

---

## 9. Вопросы для проверки понимания

По этим вопросам будет тест. Ответы должны быть своими словами, с опорой на ваш PR и workflow.

1. Что происходит при push в вашу ветку? Запускается ли деплой?
2. Какие workflow запускаются на PR, а какие — при merge в `main`?
3. Почему прямой push в `main` запрещён? Что включает branch protection?
4. Почему сайт обновляется только после merge, а не после push в ветку?
5. Зачем в CI нужен `npm ci`, а не `npm install`?
6. Что делают `actions/configure-pages`, `upload-pages-artifact` и `deploy-pages`?
7. Зачем нужны `permissions: pages: write` и `id-token: write`?
8. Почему возник конфликт в `HomeView.vue` или `router/index.ts`? Как вы его разрешили, не удалив чужой вклад?
9. Почему выбран hash-режим роутера? Что будет при `createWebHistory` на GitHub Pages?
10. Как проверить, что на сайте именно ваша версия страницы?

---

## 10. Сроки

Дедлайны заполняет преподаватель.

| Этап | Кто | Результат | Дедлайн |
|---|---|---|---|
| Настройка проекта и CI/CD | Техлид | Сайт, общий компонент, пример страницы, зелёный deploy | |
| Ветки и карточки | Каждый студент | PR с карточкой, страницей и маршрутом | |
| Review и merge | Каждый студент | Approval, зелёный CI, merge | |
| Проверка деплоя | Каждый студент | Карточка и страница на сайте | |
| Тест | Все | Ответы на вопросы раздела 9 | |

---

## 11. Если что-то пошло не так

| Проблема | Что проверить |
|---|---|
| Сайт 404 | Settings → Pages → Source: GitHub Actions; последний deploy run; `base` в `vite.config.ts` |
| Пустая страница | DevTools → Network: не 404 ли JS/CSS; правильный ли `base` |
| Прямая ссылка на страницу 404 | Используется ли `createWebHashHistory` |
| CI красный | Actions → run → job `build` → шаг с ошибкой; запустить `npm run build` локально |
| Нельзя запушить в `main` | Это branch protection. Создайте ветку и PR |
| PR конфликтует | `git fetch`, `git rebase origin/main`, сохранить все карточки и маршруты |
| Деплой красный | Лог `deploy.yml`: permissions, Pages Source, `dist` |
| CI не запускается на PR | Файл `ci.yml` есть в `main`; в `on.pull_request.branches` указана ваша основная ветка |
| Картинка не грузится | Импортировать файл в компонент (рекомендуется); для `public/` строить путь через `import.meta.env.BASE_URL` |

---

**Итог:** каждый студент должен не просто «добавить карточку», а провести изменение через ветку, PR, CI, review, merge и деплой — и уметь объяснить каждый шаг.
