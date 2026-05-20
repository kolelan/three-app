# Конфигурация арены

## Объект `arenaSettings`

Единый источник параметров матча. Значения по умолчанию — в начале скрипта `web/index.html`; перед игрой перезаписываются из UI.

```javascript
const arenaSettings = {
    mapWidth: 108,
    mapDepth: 144,
    terrainSeed: 1,
    lives: 5,
    pitCount: 13,
    coverCount: 17,
    enemyCount: 18,
    jumpHeight: 1.5,
    playerDamageRadius: 1.0,
    enemyDamageRadius: 1.2,
    mazeEnabled: true,
    mazeFloors: 2,
    mazeEnemyCount: 4,
    mazeSeed: 42,
    bossCount: 1,
    bossHealth: 25,
    bossShape: 'octahedron',
    bossColor: '#aa22ff',
    bossGlow: 1.2,
    bossPulse: 2,
    spawnInMaze: false
};
```

## Пресеты сложности

`DIFFICULTY_PRESETS`: `easy` | `normal` | `hard`.

Кнопки `.diff-btn[data-diff]` применяют объект пресета к `arenaSettings` и синхронизируют слайдеры.

| Параметр | Easy | Normal | Hard |
|----------|------|--------|------|
| lives | 8 | 5 | 3 |
| jumpHeight | 2.2 | 1.5 | 1.0 |
| enemyCount | 8 | 18 | 18 |
| mazeFloors | 1 | 2 | 3 |
| mazeEnemyCount | 1 | 4 | 10 |
| bossCount | 0 | 1 | 2 |
| spawnInMaze | false | false | true |

## Привязка UI → settings

| Input ID | Поле `arenaSettings` |
|----------|----------------------|
| `inp-width` | `mapWidth` |
| `inp-depth` | `mapDepth` |
| `inp-terrain-seed` | `terrainSeed` |
| `inp-pits` | `pitCount` |
| `inp-covers` | `coverCount` |
| `inp-lives` | `lives` |
| `inp-jump` | `jumpHeight` |
| `inp-player-radius` | `playerDamageRadius` |
| `inp-enemy-radius` | `enemyDamageRadius` |
| `inp-enemies` | `enemyCount` |
| `inp-maze-enabled` | `mazeEnabled` |
| `inp-maze-floors` | `mazeFloors` |
| `inp-maze-enemies` | `mazeEnemyCount` |
| `inp-maze-seed` | `mazeSeed` |
| `inp-boss-count` | `bossCount` |
| `inp-boss-hp` | `bossHealth` |
| `inp-boss-shape` | `bossShape` |
| `inp-boss-color` | `bossColor` |
| `inp-boss-glow` | `bossGlow` |
| `inp-boss-pulse` | `bossPulse` |
| `inp-spawn-maze` | `spawnInMaze` |

**Функции:**

| Функция | Когда |
|---------|--------|
| `readArenaSettingsFromUI()` | Перед build / start |
| `updateSetupLabelsFromSettings()` | Открытие панели |
| `syncSetupLabels()` | `input` на слайдерах |
| `applyJumpFromSettings()` | Пересчёт `JUMP_VELOCITY` |
| `randomizeArenaSeeds()` | Перегенерация — новые `terrainSeed`, `mazeSeed` |

## Влияние параметров на геймплей

| Параметр | Эффект |
|----------|--------|
| `mapWidth` / `mapDepth` | Размер террейна, туман, границы движения |
| `terrainSeed` | Амплитуда рельефа `getTerrainHeight` |
| `pitCount` | Число опасных зон провала |
| `coverCount` | Число укрытий на карте |
| `jumpHeight` | Прыжок, `floorStep` лабиринта, `canReachMazeTop` |
| `playerDamageRadius` | Зона попадания по игроку (пули, взрывы) |
| `enemyDamageRadius` | Дистанция урона по опасным врагам/боссам |
| `mazeFloors` | Этажи в `generateMazeGrid` |
| `mazeSeed` | RNG лабиринта `mazeRngState` |
| `bossShape` | `box` / `sphere` / `octahedron` в `createBossMesh` |

## Константы вне `arenaSettings`

Баланс оружия и боя задаётся отдельными `const` (см. [weapons.md](weapons.md)):

- `SNIPER_COOLDOWN`, `IMPULSE_FORCE`, `FLAME_TICK`, …
- Меняются только в коде, не в setup-panel.

## PWA `manifest.json`

| Поле | Текущее |
|------|---------|
| `name` | 3D Room Explorer |
| `start_url` | `.` |
| `display` | standalone |

При публикации обновите название под «3D Арена» для согласованности с игрой.

## Рекомендуемый порядок настройки теста

1. Выберите пресет сложности.
2. «Перегенерировать арену» — проверить рельеф/лабинт.
3. «Переместить игрока» — точка спавна.
4. «Начать игру».

## Расширение конфигурации

1. Добавьте поле в `arenaSettings`.
2. Добавьте control в `#setup-panel` (с `id="inp-..."`).
3. Обработайте в `readArenaSettingsFromUI` и `updateSetupLabelsFromSettings`.
4. Используйте в `buildWorld` / геймплее.
5. При необходимости — ключ в каждом пресете `DIFFICULTY_PRESETS`.
6. Обновите таблицу в этом файле.
