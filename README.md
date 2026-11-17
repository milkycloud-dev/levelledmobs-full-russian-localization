# LevelledMobs Full Russian Localization

<p align="center">
  <img src="assets/icon.png" width="128" height="128" alt="LevelledMobs Russian Localization Logo">
</p>

<p align="center">
  <strong>Comprehensive Russian localization, rule architecture, and bugfix documentation for LevelledMobs v4</strong><br>
  <strong>Полная русская локализация, архитектура правил и документация исправлений для LevelledMobs v4</strong>
</p>

<p align="center">
  <a href="https://github.com/milkycloud-dev/levelledmobs-full-russian-localization/actions"><img src="https://github.com/milkycloud-dev/levelledmobs-full-russian-localization/workflows/Build/badge.svg" alt="Build Status"></a>
  <a href="https://github.com/milkycloud-dev/levelledmobs-full-russian-localization/releases"><img src="https://img.shields.io/github/v/release/milkycloud-dev/levelledmobs-full-russian-localization?color=blue" alt="Latest Release"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-lightgrey" alt="License: MIT"></a>
</p>

<p align="center">
  <a href="#english">English</a> | <a href="#русский">Русский</a>
</p>

---

<a id="english"></a>

## English

### Overview

LevelledMobs is one of the most versatile RPG mob scaling plugins for Minecraft Paper, Spigot, and Purpur servers. In production environments, standard setups often suffer from naming bugs: mobs display raw English internal identifiers (such as `Pillager` or `Zombie`) for Bedrock players (via Geyser) or whenever translation component packets fail to deserialize. Furthermore, default rules frequently clobber custom nametags and health bars of external boss plugins.

This repository provides a production-hardened configuration and translation suite for **LevelledMobs v4**, featuring an exhaustive 100+ entity Russian dictionary, resolution of the internal override shadowing bug, and seamless compatibility with external boss systems.

---

### Detailed Technical Fixes

#### 1. Resolution of the `all_entities` Shadowing Bug
- **The Issue**: Despite having custom entity overrides, mob nametags consistently displayed English identifiers like `Pillager` instead of `Разбойник`.
- **Bytecode Root Cause**: Disassembly of `io.github.arcaneplugins.levelledmobs.rules.RulesManager.getRuleEntityOverriddenName()` revealed the following precedence:
  ```java
  if (map.containsKey("all_entities")) {
      return map.get("all_entities");
  }
  ```
  Because default configurations contained `all_entities: '%displayname%'`, this catch-all branch executed first, intercepting every mob lookup before `getNameIfBaby(entity)` could inspect specific types (such as `PILLAGER`). In `replaceStringPlaceholders$lambda$0`, `%displayname%` resolved directly to Bukkit's capitalized English type name.
- **The Fix**: Completely removed `all_entities` from `entity-name-override`. Populated all active nametag presets (`nametag-using-numbers`, `nametag-using-indicator`, and `nametag-minimized`) with explicit translations for all Minecraft 1.21 entities (hostile, neutral, passive, bosses, and baby variants like `BABY_ZOMBIE`, `BABY_PIG`, plus fallback `baby_`).

#### 2. Isolation of External Bosses and NPCs
- **The Issue**: LevelledMobs would detect custom bosses from plugins like Theosis and BloodMoonPlus as standard monsters, overwriting their elaborate titles, rarities, and custom boss health displays with standard level tags.
- **The Fix**: Injected `nametag-disabled: true` rules matching entities with custom names, plugin-assigned metadata, and boss flags.

#### 3. Spawner Mob Normalization
- **The Issue**: Mobs spawned from spawner blocks were prepended with raw English tags.
- **The Fix**: Adjusted the `Spawner Cube Entities` rule to inherit localized entity names cleanly without raw prefix concatenation.

---

### File Structure

- `rules.yml`: Core leveling rules, complete 1.21 Russian entity dictionaries, boss exclusions, and tiered health bars.
- `messages.yml`: 100% Russian translations of all admin commands, help dialogues, summons, and limit notifications.
- `settings.yml`: Optimized thread pooling, update checkers disabled, and performance tuning for high-concurrency servers.

---

### Protection Against Auto-Updaters

Automatic plugin updaters (such as `vanilla-plugin-updater`) can rewrite `rules.yml` or overwrite configuration files with upstream defaults during automated cycles.
To prevent breaking localizations:
1. Add `LevelledMobs` to the updater ignore list (`IGNORE_DEFAULT` in compose or `state.json`: `"ignore": true, "auto": false`).
2. Maintain local backups of custom configurations before applying plugin jar updates.

---

<a id="русский"></a>

## Русский

### О проекте

LevelledMobs является мощным инструментом прокачки и масштабирования сложности монстров на серверах Paper, Spigot и Purpur. Однако в стандартной конфигурации сервер часто сталкивается с проблемой отображения названий: игроки с Bedrock (через Geyser) и ванильные клиенты часто видят внутренние английские идентификаторы существ (`Pillager`, `Zombie`, `Skeleton`) вместо полноценных русских названий. Кроме того, стандартные правила плагина нередко перехватывают и стирают неймтеги сторонних боссов.

Этот репозиторий содержит проверенную конфигурацию и полный пакет локализации для **LevelledMobs v4**, устраняющий баг перехвата имен, добавляющий исчерпывающий словарь существ Minecraft 1.21 и обеспечивающий бесконфликтную работу с боссами.

---

### Подробный разбор проблем и технических исправлений

#### 1. Устранение бага перехвата имен `all_entities`
- **Проблема**: При появлении монстров неймтеги отображали английские имена (например, `Pillager` вместо `Разбойник`), даже если в конфигурации были прописаны отдельные переводы.
- **Причина в коде**: Декомпиляция метода `io.github.arcaneplugins.levelledmobs.rules.RulesManager.getRuleEntityOverriddenName()` показала логику проверки:
  ```java
  if (map.containsKey("all_entities")) {
      return map.get("all_entities");
  }
  ```
  В конфигурации присутствовала строка `all_entities: '%displayname%'`. Метод проверял ключ `all_entities` в первую очередь, до вызова метода проверки конкретного моба `getNameIfBaby(entity)`. Плейсхолдер `%displayname%` в плагине при отсутствии кастомного имени брал имя типа из Bukkit, выводя стандартное английское название типа моба.
- **Решение**: Параметр `all_entities` полностью удален из всех секций `entity-name-override`. Во все рабочие пресеты (`nametag-using-numbers`, `nametag-using-indicator`, `nametag-minimized`) встроен словарь из более чем 100 существ Minecraft 1.21 (враждебные, нейтральные, мирные, боссы, а также детеныши мобов `BABY_ZOMBIE`, `BABY_PIG` и префикс `baby_`).

#### 2. Защита сторонних боссов и NPC
- **Проблема**: Плагин воспринимал процедурных и кастомных боссов из Theosis и BloodMoonPlus как обычных мобов и заменял их уникальные титулы стандартным неймтегом уровня (`Ур. X | Название`).
- **Решение**: Настроено защитное правило с параметром `nametag-disabled: true` для всех существ с кастомными именами или метаданными сторонних плагинов.

#### 3. Корректный вывод мобов из спавнеров
- **Проблема**: Мобы из спавнер-кубов переопределялись отдельным правилом `Spawner Cube Entities` и теряли локализацию.
- **Решение**: Секция очищена от перехвата, благодаря чему мобы из спавнеров выводят переведенные русские названия.

---

### Состав файлов конфигурации

- `rules.yml`: Тонко настроенные правила уровней, полный словарь русских имен для 1.21, защита боссов.
- `messages.yml`: Полная русская локализация интерфейса, консольных команд, подсказок и сообщений об ошибках.
- `settings.yml`: Оптимизация производительности, отключение фоновых проверок обновлений и согласованность форматов.

---

### Защита от автообновляторов

Автоматические системы обновления плагинов могут перезаписывать файлы конфигурации или заменять jar-файлы, сбрасывая сделанные правки.
Для защиты настроек:
1. Добавьте `LevelledMobs` в список исключений вашего автообновлятора (`vanilla-plugin-updater`): в `state.json` укажите `"ignore": true, "auto": false`.
2. В файлах запуска или `compose.yml` укажите плагин в переменной `IGNORE_DEFAULT`.

---

### Лицензия

MIT License. Разработано и протестировано milkycloud-dev (2026).
