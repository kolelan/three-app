# Мир и сущности

## Компонент: генерация мира

Точка сборки — `buildWorld()`:

```
buildTerrain()
buildArenaShell()
buildCovers()
buildMaze()          // если arenaSettings.mazeEnabled
```

Полная пересборка геометрии: `rebuildArenaGeometry()` → `clearArenaMeshes()` + build.

**Превью без боя:** `regenerateArenaPreview()` — новый мир + `relocatePlayerToSpawn()`.

### Террейн (`buildTerrain`)

- `PlaneGeometry` с сегментами по размеру карты
- Высота вершин: `getTerrainHeight(x, z)` — синусоиды × `terrainSeed`, впадины ям `PITS`
- Меш: `terrain` (один `Mesh`)

### Ямы

- Данные: `PITS[]` из `BASE_PITS`, масштаб `scaleLayout`
- Визуал: `buildPitHazardVisuals()` — кольца, подсветка
- Логика: `isInsidePitZone`, `recoverPlayerFromVoid`

### Укрытия (`buildCovers`)

- `COVER_LAYOUT` → боксы `covers[]`
- `userData.isCover`, `blocksBullets`, `halfW`, `halfD`, `height`

### Оболочка арены (`buildArenaShell`)

Стены и потолок за пределами игровой зоны (`shellMeshes`).

## Компонент: лабиринт

Включение: `arenaSettings.mazeEnabled`.

| Этап | Функция |
|------|---------|
| Сетка | `generateMazeGrid(cols, rows, maxFloors)` |
| Пол | `createMazeFloorTile` — плитка = ячейка |
| Стены | `buildMazeCellWalls`, `createMazeWall` |
| Регистрация | `registerMazePiece`, `addMazeMesh` |

**Поле `mazePiece` (на mesh):**

| Поле | Смысл |
|------|--------|
| `blocksBullets` | Блокирует пули и LOS |
| `blocksMovement` | Стена для игрока |
| `jumpable` | Можно перепрыгнуть |
| `isWindow` | Окно — пули проходят |
| `hasRoof` / `roofY` | Крыша — укрытие от взрывов сверху |

**Коллизии игрока:** `resolveMazeCollision(x, z, footY, headY)` — стены нижних этажей не блокируют голову на верхнем.

**Высота хода:** `getMazeWalkHeight`, `floorStep` от `jumpHeight`.

**Спавн в лабиринте:** `arenaSettings.spawnInMaze`, `collectSpawnCandidates`, `isSafeSpawnAt`.

## Компонент: спавн сущностей

`spawnWorldEntities()` после `buildWorld()`:

1. `spawnStaticTargets()` — декоративные/тренировочные цели
2. `getEnemySpawnList()` + `spawnEnemy(cfg)` на поле
3. `getMazeEnemySpawns()` — враги в лабиринте
4. `bossesToSpawn = arenaSettings.bossCount` (волна `spawnBossWave`)

**Сброс боя:** `clearCombatState()` — пули, враги, мины, ракеты, метки.

## Компонент: враги

Массивы: `enemies[]`, все цели также в `shootables[]`.

### Типы (поле `userData.type`)

| type | Поведение (упрощённо) |
|------|-------------------------|
| `patrol` | Патруль `homePos`, не стреляет |
| `shooter` | Стрельба в игрока на дистанции |
| `chaser` | Преследование + контактный урон |

Опасные: `dangerous: true` (красные). Боссы: `isBoss: true`, отдельный меш `createBossMesh`.

### Кадр AI

`updateEnemies(time, delta)`:

- Патруль / преследование / стрельба
- `placeEnemyAt` — привязка к полу `getEnemyGroundY`
- Knockback: `knockbackX/Z/Y`, `stunnedUntil`
- `enemyShoot` — пуля в `enemyBullets`

### Боссы

`spawnBossWave()` — кольцо вокруг игрока, HP из `bossHealth`, пульсация `bossPulse`.

Победа: уничтожены все `dangerous` и боссы, `bossesToSpawn === 0` → `checkVictory()`.

## Компонент: статические цели

`spawnStaticTargets()` — объекты из `BASE_STATIC_TARGETS` для отработки стрельбы (не враги).

## Масштабирование карты

`applyArenaDimensions()`:

- `WORLD.width/depth` из `arenaSettings`
- `MAP_SX`, `MAP_SZ` — множители для раскладки ям/укрытий/врагов

`scaleLayout`, `scalePoints` — перенос базовых координат на размер арены.

## Добавление нового типа врага

1. Запись в `BASE_ENEMY_LAYOUT` или спавн в `getMazeEnemySpawns`.
2. Ветка в `updateEnemies` для AI.
3. Визуал в `createEnemyVisual(type)`.
4. Параметры: `health`, `fireRate`, `chaseSpeed`, `contactDamage`.
5. Радар: цвет/легенда в `updateEnemyRadar` при необходимости.

## Связанные файлы документации

- Настройки количества врагов: [configuration.md](configuration.md)
- Урон и пули: [game-systems.md](game-systems.md)
