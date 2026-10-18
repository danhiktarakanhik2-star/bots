# 36 — Шаблон конфигурации и компоновка prompts

Не складывай версию мира, правила, личность бота, роль и все стратегии в один огромный prompt. Раздели стабильные требования и динамические данные. Так проще исправлять поведение и не подменять новую версию старой.

## 1. Конфигурация матча

```yaml
profile_id: local-manhunt-v1
edition: Java
version: SET_EXACT_VERSION
server_plugins: []
mode: manhunt
runner_lives: 1
hunter_respawn: true_or_false
tracking: compass_plugin_or_none
start_rule: SET_EXACT_RULE
portal_rules: SET_EXACT_RULE
trap_rules: SET_EXACT_RULE
win_condition: SET_EXACT_EVENT
allowed_observations:
  - own_position
  - visible_players
  - own_inventory
  - team_chat
  - allowed_tracking_signal
```

Не оставлять `SET_EXACT...` в реальном матче. Loader должен остановить запуск или запросить пользователя, если критические значения пусты.

## 2. Слои prompt’а

1. **Core:** честность, факты/гипотезы, проверка действия, краткость.
2. **Match config:** версия, правила и allowed observations.
3. **Role card:** runner/hunter/team role.
4. **Phase card:** Overworld/Nether/Stronghold/End.
5. **Retrieved tactics:** несколько подходящих примеров с версией и source status.
6. **Live observation:** текущий snapshot с временной меткой.

Не отправляй «сценарии» из датасета как будто это реальные события этого матча.

## 3. Prompt-функция решения

```text
1. Отдели факты от гипотез.
2. Найди непосредственную угрозу.
3. Назови цель на ближайшее действие.
4. Выбери primary и fallback.
5. Проверь версию/правила/инвентарь.
6. Сгенерируй только допустимое action JSON.
7. Создай короткий chat callout, если он нужен.
8. Дождись verification и обнови состояние.
```

## 4. Строгий выход

```json
{
  "goal": "...",
  "facts": ["..."],
  "uncertainties": ["..."],
  "action": {"type":"...","parameters":{}},
  "fallback": {"type":"...","parameters":{}},
  "abort_if": "...",
  "chat": "...",
  "verification_needed": true
}
```

Action schema надо проверять валидатором. Если модель выдаёт несуществующую команду, не отправляй её в сервер. Отправить обратно ограниченную ошибку и безопасный fallback.

## 5. Подбор retrieved документов

Поиск должен фильтровать по role, version, phase, правилам и источнику. Видеопример для 1.16 не должен вытеснить справку по 1.21.11. Если точная версия не совпала, добавить предупреждение и отдать пример только как общий поведенческий принцип.

## 6. Размещение secrets

Prompt не должен содержать пароли, токены, адреса или admin credentials. Эти данные не нужны для тактики и не должны попадать в logs/RAG. Разрешённые команды сервера вводить отдельно в защищённую конфигурацию.

## 7. A/B версионирование

Храни: core prompt version, role card version, ruleset id, retrieval corpus hash, controller build, game version. При изменении одного слоя запускать регрессию. Если win-rate изменился, можно найти причину.

## 8. Источники

- [Minecraft Wiki: Tutorial:Manhunt](https://minecraft.wiki/w/Tutorial:Manhunt)
- [Dream: правила Manhunt 2026](https://www.youtube.com/watch?v=km1IlBAZJH0)
