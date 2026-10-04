# Шаблон транспорта OpenFlux

Форкните репозиторий, чтобы написать свой JS-транспорт для [OpenFlux](https://github.com/p1neappleXpress/OpenFlux).
Поставьте тег - CI подпишет и опубликует релиз, а приложения, где транспорт уже установлен, получат обновление.

## Состав

| путь | что |
|---|---|
| `src/main.js` | ваш транспорт (изначально - полный аннотированный контракт) |
| `manifest.json` | id, name, version, author, `wire`, `api` |
| `sdk/` | [OpenFluxSDK](https://github.com/p1neappleXpress/OpenFluxSDK) сабмодулем: документация и справочник host API (`sdk/docs`) |
| `.github/workflows/check.yml` | на каждый push/PR: упаковка временным ключом - расхождение имени/версии или скрипт, который не грузится, упадут здесь |
| `.github/workflows/release.yml` | по тегу `v*`: подпись, релиз, обновление `update.json` |

```bash
git clone --recurse-submodules <ваш форк>    # sdk/ большой: добавьте --depth 1 --shallow-submodules
```

## Один раз

1. Сгенерируйте ключ автора. **Не коммитьте его.**
   ```bash
   go run github.com/p1neappleXpress/OpenFlux/transport/script/cmd/scriptsign@nightly genkey author.priv author.pub
   ```
2. Содержимое `author.priv` - в секрет репозитория **`SIGNING_KEY`** (Settings → Secrets and variables → Actions).
3. Опубликуйте отпечаток `author.pub` (SHA-256; приложение показывает его при первой установке), например здесь, в README.
   Приложение закрепляет ключ при первом доверии; обновления, подписанные другим ключом, отклоняются.
4. Поправьте `manifest.json` (`id`, `name`, `author`, `description`) и `info().name` в `src/main.js`.

## Выпуск версии

```bash
# поднимите "version" в manifest.json И info().version в src/main.js, закоммитьте, затем:
git tag v0.2.0 && git push --tags
```

- `v1.4.0` → канал **stable**; `v1.5.0-beta.1` → канал **nightly** (GitHub pre-release).
- CI не пропустит тег, не совпадающий с версией в `manifest.json`.
- Подписанный `<id>-<version>.flux` - ассет релиза; `update.json` (его опрашивают приложения) коммитится в ветку `updates`, по записи на канал.
- `update` в подписанный манифест CI подставляет сам. Свой `"update": [url, зеркало…]` в `manifest.json` - для другого хостинга и зеркал.

## Правила версий

- `version` - семантическая (`MAJOR.MINOR.PATCH`). Приложения идут только вперёд.
- **`wire`** - поколение вашего формата обмена. Клиент и нода запускают один и тот же транспорт: пока старая и новая версии понимают друг друга, `wire` не меняйте
  (официальные транспорты обновятся сами, остальные - одним нажатием). Если не понимают - поднимите `wire`: приложение задержит обновление до подтверждения, ведь ноду тоже надо обновить.
- **`api`** - поколение host API, под которое вы писали; оставьте как в шаблоне.
- Смена ключа подписи - это не обновление: пользователям придётся импортировать и доверить новый ключ заново.

> Пока новые команды `scriptsign` (проверки в `pack`, `index`) не попали в `main` OpenFlux, CI собирает его из ветки `nightly`.
> Переменная репозитория `SCRIPTSIGN_REF` задаёт другую ветку или тег.
