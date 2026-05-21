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
    spawnInMaze: false,
    weaponsOnField: false,
    treeCount: 12,
    treeLayers: 4,
    treeHeight: 5.5,
    treeBaseRadius: 1.4,
    treeLayerScale: 0.72,
    treeSeed: 77,
    treeColor: '#1a6b2e'
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
| `inp-enemy-hp` | `enemyDangerHealth` |
| `inp-enemy-patrol` | `enemyPatrolSpeed` |
| `inp-enemy-chase` | `enemyChaseRange` |
| `inp-enemy-chase-spd` | `enemyChaseSpeed` |
| `inp-enemy-shoot` | `enemyShootRange` |
| `inp-enemy-fr-sh` | `enemyFireRateShooter` |
| `inp-enemy-fr-ch` | `enemyFireRateChaser` |
| `inp-enemy-contact` | `enemyContactDamage` |
| `inp-enemy-bullet` | `enemyBulletLives` |
| `inp-enemy-charge-t` | `enemyChargeTime` |
| `inp-enemy-color-main` | `enemyColorMain` |
| `inp-enemy-color-ex1` | `enemyColorExtra1` (кольцо) |
| `inp-enemy-color-ex2` | `enemyColorExtra2` (шип) |
| `inp-enemy-color-patrol` | `enemyColorPatrol` |
| `inp-enemy-color-charge` | `enemyColorCharge` |
| `inp-enemy-color-bullet` | `enemyColorBullet` |
| `inp-static-hp` | `staticTargetHp` |
| `inp-enemy-radius` | `enemyDamageRadius` |
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
| `inp-boss-dangerous` | `bossDangerous` |
| `inp-boss-charge` | `bossChargeTime` |
| `inp-boss-cooldown` | `bossFireCooldown` |
| `inp-boss-bullet-lives` | `bossBulletLives` |
| `inp-boss-chase` | `bossChaseRange` |
| `inp-boss-chase-speed` | `bossChaseSpeed` |
| `inp-boss-shoot-range` | `bossShootRange` |
| `inp-spawn-maze` | `spawnInMaze` |
| `inp-weapons-on-field` | `weaponsOnField` |
| `inp-tree-count` | `treeCount` |
| `inp-tree-layers` | `treeLayers` |
| `inp-tree-height` | `treeHeight` |
| `inp-tree-radius` | `treeBaseRadius` |
| `inp-tree-scale` | `treeLayerScale` |
| `inp-tree-seed` | `treeSeed` |
| `inp-tree-color` | `treeColor` |

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
| `mazeFootSnap` | Радиус прилипания ног к полу лабиринта у края плиты (`footXZOnMazeWalkPiece`) |
| `mazeFootInset` | Сжатие площади плиты внутрь — меньше «висения» на краю без опоры снизу |
| `playerDamageRadius` | Зона попадания по игроку (пули, взрывы) |
| `enemyDamageRadius` | Дистанция урона по опасным врагам/боссам |
| `mazeFloors` | Этажи в `generateMazeGrid` |
| `mazeSeed` | RNG лабиринта `mazeRngState` |
| `bossShape` | `box` / `sphere` / `octahedron` в `createBossMesh` |

## Уровни (прогрессия после победы)

| Поле | По умолчанию | Эффект за каждый уровень после 1-го |
|------|--------------|--------------------------------------|
| `levelEnemyPerStep` | 5 | + врагов на поле (`enemyCount`, макс. 18) |
| `levelBossPerStep` | 1 | + боссов каждые **2** уровня: ур. 1–2 — база, ур. 3 — +1, ур. 5 — +2… (макс. 5) |
| `levelEnemyRadiusStep` | 0.04 | − `enemyDamageRadius` (мин. 0.35) |
| `levelPlayerRadiusStep` | 0.03 | + `playerDamageRadius` (макс. 2.2) |

База уровня 1 — снимок настроек при «Начать игру» (`runBaseSettings`). Уровень N применяется в `applySettingsForLevel(N)` перед перегенерацией арены у двери.

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
