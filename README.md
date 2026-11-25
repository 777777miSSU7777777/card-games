# card-games

Внутренняя платформа карточных игр: blackjack, дурак и Texas Hold'em (web + mobile).
Продуктовые, архитектурные и технические решения — в [`docs/`](./docs/00-overview.md).

## Статус

Инициализирован каркас — первый шаг P0 по [`docs/04-plan.md`](./docs/04-plan.md): три
самостоятельных проекта `server`, `web`, `mobile`, у каждого свой `package.json`,
`tsconfig.json`, `node_modules` и lockfile. В проектах настроены Biome и TypeScript;
запускаемых приложений, тестов, БД, Docker Compose и CI пока нет — это следующий шаг спринта 1.

## Требования

- Node.js 26 LTS — версия зафиксирована в `.nvmrc`;
- pnpm 12 через Corepack.

## Структура

```
server/          NestJS: REST + WebSocket + игры + таймеры
web/             React + Vite SPA
mobile/          Expo + React Native
docs/            источник истины по продукту и архитектуре
```

Проекты независимы: нет pnpm workspace и Turborepo, исходники проектов не импортируются друг
в друга. Общий код (движки правил, протокол, клиентский слой, i18n) живёт внутри проекта,
которому он нужен, копируется или выносится в версионируемые пакеты через приватный registry.
Отдельных библиотек-проектов нет. У каждого проекта свой `node_modules` и `pnpm-lock.yaml`.

## Команды

Выполняются внутри каталога нужного проекта:

```bash
cd server            # или web, mobile
corepack enable
pnpm install
pnpm lint
pnpm typecheck
```

Установить зависимости сразу во всех проектах:

```bash
for project in server web mobile; do
  (cd "$project" && pnpm install)
done
```

Тестов и сборки пока нет: они появятся вместе с каркасами NestJS, Vite и Expo.

## Правила работы

Процесс разработки, требования к коммитам и определение готовности — в [`AGENTS.md`](./AGENTS.md).
