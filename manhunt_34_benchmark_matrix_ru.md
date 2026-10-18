# 34 — Benchmark и матрица испытаний

Benchmark нужен для ответа на конкретный вопрос: какой тип бота стал лучше и на каких условиях? Без одного и того же ruleset и набора seed’ов сравнение некорректно. Никогда не объединяй Java/Bedrock или разные версии в одну цифру без отдельной разбивки.

## 1. Разделы тестового набора

- **Smoke test:** бот подключается, видит состояние, выполняет безопасное движение, отправляет чат.
- **Skill tests:** перемещение, крафт, инвентарь, бой, portal transitions, навигация.
- **Scenario tests:** конкретные состояния из файла 11.
- **Match tests:** полные игры 1v3, 1v4, 1v5.
- **Stress tests:** задержки, loss of contact, teammate death, conflicting messages.
- **Generalization:** новые seed’ы, биомы и человеческие стили.

## 2. Матрица ролей

| Роль | Проверки |
|---|---|
| Runner | выживание, прогресс, выбор боя, использование ресурсов, End objective |
| Hunter-contact | свежие callout’ы, сохранение видимости, передача роли |
| Hunter-interceptor | путь к выходу, отмена неверного прогноза, безопасная позиция |
| Hunter-support | нужные ресурсы, возврат, переключение на контакт |
| Coordinator | назначение задач, разрешение конфликтов, recovery после смерти |

В полной игре минимум отдельные тесты для 3, 4 и 5 hunters, а также разные конфигурации человеческого игрока.

## 3. Запись условий

Каждый результат включает: commit/version бота; версия игры; сервер/paper/spigot/mod profile; seed или анонимный world id; ruleset; состав игроков; difficulty; дата; доступные сенсоры; логи; исход; причина ошибок. Seed можно скрыть от агента, но тестирующая система должна знать его для воспроизводимости.

## 4. Метрики

- match win-rate и confidence interval;
- median/percentile time-to-objective;
- deaths by cause;
- progress milestones;
- time without goal progress;
- contact loss/recovery time;
- false claims per 1000 messages;
- duplicated tasks per match;
- critical-item misuse;
- rule violations (целевое значение ноль);
- controller failures отдельно от planner failures.

## 5. A/B-эксперимент

Тестируй старую и новую версию бота в одинаковой паре сценариев. Рандомизируй сторону/seed, если это возможно. Не менять одновременно prompt, controller и ролевая модель. Сохраняй логи всех проигрышей и выборки побед, чтобы не смотреть только на красивые матчи.

## 6. Разделение данных

Train/test делить по seed’у и матчу. Если одна и та же карта есть в обучении и оценке, модель может запомнить карту вместо принципа. Для `unknown` version/test не пытаться оценивать предметную механику как будто она известна.

## 7. Минимальная таблица результата

```text
agent_version | game_version | role | hunter_count | seed_group
win_rate | median_time | deaths | false_claims | coordination_errors
biggest_failure | next_change | regression_pass
```

## 8. Стоп-критерии

Не выпускать новую модель, если она стала быстрее, но начала нарушать правила; если выросло число ложных координат; если отступления runner’а чаще заканчиваются случайной смертью; если hunter-team чаще дублирует задачи. Победа в одном тесте не компенсирует системную проблему.

## 9. Источники

- [Minecraft Wiki: Tutorial:Manhunt](https://minecraft.wiki/w/Tutorial:Manhunt)
- [Dream: How Minecraft Manhunt is Made](https://www.youtube.com/watch?v=km1IlBAZJH0) — пример того, почему измерять нужно собственный ruleset.
