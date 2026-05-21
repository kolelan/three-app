# Оружие и бой

## Реестр оружия

Константа `WEAPON` и метаданные `WEAPON_INFO`:

```javascript
const WEAPON = { STANDARD: 1, SNIPER: 2, AREA: 3, MINE: 4, FLAMETHROWER: 5, LASER: 6, TELEPORT: 7 };
```

| ID | Ключ | Слот | ЛКМ | ПКМ |
|----|------|------|-----|-----|
| 1 | `STANDARD` | ① | `shootStandard` | `fireImpulse` |
| 2 | `SNIPER` | ② | `shootSniper` | — |
| 3 | `AREA` | ③ | `fireAreaStrike` | `markArea` |
| 4 | `MINE` | ④ | `detonateMineClick` | `placeMine` |
| 5 | `FLAMETHROWER` | ⑤ | удержание `ЛКМ` | — |
| 6 | `LASER` | ⑥ | `startLaserCharge` → луч | — |
| 7 | `TELEPORT` | ⑦ | `executeTeleport` | `markTeleport` |

Переключение: `setWeapon(id)`, клавиши `1`–`7`, `B` → `cycleWeapon()`.

## Режим «Оружие на поле»

Чекбокс `inp-weapons-on-field` → `arenaSettings.weaponsOnField`.

| Режим | Поведение |
|-------|-----------|
| Выключен (по умолчанию) | Все слоты ①–⑤ доступны сразу |
| Включен | Старт только с ①; ②–⑥ — пикапы на карте (`weaponPickups`) |

**Состояние:** `weaponUnlocks` (`Set`), сброс в `resetWeaponProgress()` при старте и в меню.

**Пикапы:** `spawnWeaponPickups()` после `spawnWorldEntities()`; сбор в `updateWeaponPickups(time)` при дистанции ≈ 1.75 м. Заблокированные слоты в HUD помечены 🔒; `setWeapon` / `cycleWeapon` / клавиши `2`–`5` работают только для разблокированного оружия.

## Точки входа (компонент Weapons)

```mermaid
flowchart TD
    Mousedown[mousedown ЛКМ/ПКМ]
    Mousedown --> Primary[handlePrimaryClick]
    Mousedown --> Secondary[handleSecondaryDown]
    Primary --> W1[shootStandard / sniper / area / mine]
    Primary --> Flame[startFlamethrower]
    Secondary --> Impulse[fireImpulse]
    Secondary --> Mark[markArea / placeMine]
    Animate[animate] --> UpdateFlame[updateFlamethrower]
```

| Функция | Когда вызывается |
|---------|------------------|
| `handlePrimaryClick(event)` | Клик ЛКМ (кроме огнемёта) |
| `handleSecondaryDown(event)` | ПКМ |
| `startFlamethrower` / `stopFlamethrower` | ЛКМ вниз/вверх для слота 5 |
| `updateFlamethrower(delta, time)` | Каждый кадр при удержании |

## Режим дальнего боя (G)

**Состояние:** `rangedMode`, углы `rangedCamPitch`, `rangedCamYaw`, FOV `rangedFov`.

| Действие | Поведение |
|----------|-----------|
| `G` | `toggleRangedMode()` |
| `W` / `S` | Наклон прицела вверх / вниз (`updateRangedAimCamera`) — **не** ходьба |
| `A` / `D` | Поворот прицела влево / вправо (`updateRangedAimCamera`) — **не** поворот тела |
| `Q` / `E` | Стрейф игрока влево / вправо (`applyPlayerStrafeMove`, `useAimYaw: true`) — **не** камера |

> Не смешивать ветку `rangedMode` в `animate()` с обычной ходьбой: иначе WASD снова двигают тело.
| Колесо | Изменение `rangedFov` (8°–45°) |
| Мышь | Прицел (`#scope-reticle`) и луч попадания следуют за курсором |

**Направление выстрела в режиме G:** `getRangedAimDirection()` — raycast из камеры через `aimMouse` по целям и рельефу.

**Ограничения в режиме G:**

- Нет ходьбы вперёд/назад (W/S не двигают `position`); Q/E — медленный стрейф
- Нет прыжка (`canPlayerJump` → false)
- Обычный `#crosshair` скрыт

**Камера:** `applyCameraLook()` → `lookAt(getRangedLookTarget())`.

## ① Стандарт

- Пуля: `bullets`, скорость `BULLET_SPEED`, урон 1.
- ПКМ: `fireImpulse()` — волна, отброс и урон в радиусе `IMPULSE_RADIUS`.
- Игнор коллизий у дула: `blockIgnoreUntil` на 80 ms.

## ② Винтовка

- Кулдаун `SNIPER_COOLDOWN`, скорость `SNIPER_SPEED`, время полёта `SNIPER_LIFE`, урон `SNIPER_DAMAGE`.
- Попадание: `resolvePlayerBulletHit` по отрезку кадра + `SNIPER_SEGMENT_PAD` / `SNIPER_HIT_RADIUS_MUL`, флаг `bullet.userData.sniper`.
- Медленнее движение вне режима G: `SNIPER_MOVE_MULT`.

## ③ Ракетница

1. ЛКМ: `markArea` (по прицелу) + `fireAreaStrike` — ракета и взрыв.
2. ПКМ: только `markArea` — метка без выстрела.
3. `raycastMarkPoint` — hit по миру или fallback на поверхность вдоль луча.
4. Урон с учётом укрытий: `isUnderMazeRoof`, `isExplosionBlockedToTarget`.

## ④ Мины

- Массив `mines[]`: `{ mesh, x, z, y, dead, phase }`.
- ПКМ: `placeMine` — лимит `MINE_MAX_COUNT`, кулдаун `MINE_PLACE_COOLDOWN`.
- ЛКМ: `detonateMineClick` — raycast по мине или ближайшая в радиусе.
- Проксимити: `updateMines` → `detonateMineAt` у врагов.
- Взрыв: `applyExplosionAt` с `chainMines: true`.

## ⑤ Огнемёт

Константы: `FLAME_RANGE`, `FLAME_TICK`, `FLAME_DAMAGE`, `FLAME_MIN_DOT`.

| Функция | Роль |
|---------|------|
| `applyFlameDamage()` | Конус + LOS, `onTargetHit` каждые ~0.11 с |
| `updateFlameVisual(time)` | `flameStream` — узкая струя сегментов от `getFlameMuzzleWorld` (~`FLAME_VISUAL_RANGE`) |
| `getFlameAimDirection()` | Луч: в G — `getRangedAimDirection`, иначе raycast по мыши |

## ⑥ Лазер

- ЛКМ: `startLaserCharge` → `updateLaserCharge` (~`LASER_CHARGE_TIME` с) → `fireLaserBeam`.
- Визуал зарядки: `laserChargeVisual` (сфера + кольца у дула).
- Выстрел: hitscan-луч на `LASER_MAX_RANGE`, длина до первой стены/укрытия (`computeLaserBeamLength`, AABB лабиринта).
- Урон всем врагам на линии до преграды; луч `spawnLaserBeamVisual`, затухание `updateLaserBeams`.
- Кулдаун `LASER_COOLDOWN`; в G прицеливание как у остального оружия.

## ⑦ Телепорт

Как ракетница по метке, но вместо удара — перенос игрока.

| Действие | Поведение |
|----------|-----------|
| ПКМ | `markTeleport` — только метка |
| ЛКМ | метка + `executeTeleport` |
| Прицел в небо / над лабиринтом | точка на **верхней проходимой плите** лабиринта (`getMazeHighestWalkTop`), столб прилёта |
| Прицел в землю/стены | верхняя поверхность в точке (лабиринт приоритетнее рельефа) |

Кулдаун `TELEPORT_COOLDOWN`; проверка `canTeleportTo` (ямы, стены лабиринта). Вспышка `spawnTeleportFlash` на старте и финишe.

## Связанные системы

- Попадания: [game-systems.md](game-systems.md) → `onTargetHit`, `updateBullets`
- Прицел: [ui-components.md](ui-components.md) → scope / crosshair
- Настройки урона: [configuration.md](configuration.md) → радиусы в `arenaSettings`
