# UI-компоненты (DOM)

Все элементы объявлены в `<body>` файла `web/index.html`. Стили — в блоке `<style>`. Большинство HUD с `pointer-events: none`, чтобы не перехватывать клики по canvas.

## Слои (z-index)

| z-index | Элемент | Роль |
|---------|---------|------|
| 10 | `#info`, `#stats`, `#controls` | Информация и статистика |
| 11 | `#crosshair`, `#enemy-radar` | Прицел и радар |
| 12 | `#scope-reticle`, `#restart-btn` | Прицел дальнего боя, кнопка |
| 9 | `#scope-overlay` | Затемнение по краям (режим G) |
| 15–16 | `#game-over`, `#victory` | Экраны конца игры |
| 20 | `#setup-panel` | Меню настроек |

## Компонент: HUD информации

| ID | Описание |
|----|----------|
| `#info` | Заголовок и краткая подсказка; текст меняется в `startGame` / победе |
| `#controls` | Управление; частично устарело — см. также панель оружия и `G` |

**Обновление:** вручную через `infoEl.innerHTML` в ключевых функциях (`startGame`, `checkVictory`, `openSetupPanel`).

## Компонент: статистика и жизни

| ID | JS-переменная | Обновление |
|----|---------------|------------|
| `#lives` | `livesEl` | `damagePlayer`, `startGame` |
| `#stat-points` | `stats.points` | `updateStatsUI` — 1 балл за 1 HP урона по врагу/боссу/мишени |
| `#stat-shots` | — | `updateStatsUI` |
| `#stat-hits` | — | `updateStatsUI` |
| `#stat-destroyed` | — | `updateStatsUI` |
| `#persist-wins` и др. | `persistedStats` | `updatePersistedStatsUI`, `recordVictory` |
| `#victory-points` | забег | экран победы |
| `#stat-enemies-alive` | `statEnemiesAliveEl` | `updateEnemyRadar` |
| `#stat-enemies-total` | `statEnemiesTotalEl` | `updateEnemyRadar` |
| `#stat-enemies-danger` | `statEnemiesDangerEl` | счётчики боссов/опасных |

## Компонент: панель оружия (`#weapon-panel`)

Структура:

```
#weapon-panel.weapon-{1..5}
├── #weapon-panel-label
├── #weapon-panel-main
│   ├── #weapon-icon
│   ├── #weapon-title
│   └── #weapon-desc
└── #weapon-slots
    └── .weapon-slot[data-slot="1..5"]
```

**API в коде:**

| Функция | Действие |
|---------|----------|
| `updateWeaponUI()` | Синхронизирует иконку, название, описание из `WEAPON_INFO` |
| `setWeapon(id)` | Меняет `currentWeapon`, сбрасывает режим G, обновляет UI |
| `cycleWeapon()` | Следующий слот по кругу (`B`) |

Класс `weapon-N` на панели задаёт цвет рамки (CSS в `<style>`).

## Компонент: прицел

| ID | Поведение |
|----|-----------|
| `#crosshair` | Обычный режим; следует за мышью (`updateCrosshair`) |
| `#crosshair-dot` | Красная точка в центре перекрестия |
| `#scope-overlay` | Активен при `rangedMode`; vignette вокруг курсора |
| `#scope-reticle` | Круглый прицел в режиме G; позиция = курсор |

**Режим G:** `updateRangedScopeAtMouse(clientX, clientY)` — вызывается из `mousemove` и при включении G.

Функции: `toggleRangedMode()`, `updateRangedVisual()`.

## Компонент: радар (`#enemy-radar`)

| Элемент | Назначение |
|---------|------------|
| `#radar-canvas` | 2D-канвас 132×132, рисуется в `updateEnemyRadar` |
| `#radar-count-alive` / `total` | Счётчики |
| `#radar-behind-hint` | Предупреждение о врагах сзади |

Класс `visible` добавляется при старте игры. Цвета точек: опасный — красный, патруль — зелёный, босс — фиолетовый.

## Компонент: setup-panel (`#setup-panel`)

Полноэкранная форма настроек до начала матча.

**Секции (`.setup-col`):**

1. Арена и рельеф — размер, сиды, ямы, укрытия  
2. Игрок — жизни, прыжок, радиусы урона  
3. Враги и лабиринт — количество, этажи, боссы  
4. Босс — форма, цвет, свечение  

**Кнопки:**

| ID | Функция |
|----|---------|
| `#btn-regen-terrain` | `regenerateArenaPreview()` |
| `#btn-relocate-player` | `relocatePlayerToSpawn()` |
| `#btn-start-game` | `startGame()` |
| `.diff-btn[data-diff]` | Пресет `DIFFICULTY_PRESETS` |

Связь с логикой: `readArenaSettingsFromUI()`, `syncSetupLabels()`, `updateSetupLabelsFromSettings()`.

## Компонент: оверлеи конца игры

| ID | Показ | Скрытие |
|----|-------|---------|
| `#game-over` | `endGame()` | `openSetupPanel`, `startGame` |
| `#victory` | `checkVictory()` | то же |

`#restart-btn` дублирует `R` (клик → `openSetupPanel`).

## Компонент: мобильное управление (`#mobile-controls`)

Показывается при `gameActive` и условии `shouldUseMobileControls()` (узкий экран ≤767px или сенсор `pointer: coarse`).

| Элемент | Назначение |
|---------|------------|
| `#mobile-joystick` | Виртуальный стик → `KeyW` / `KeyA` / `KeyS` / `KeyD` |
| `#mobile-look-zone` | Правая зона экрана — поворот (`rotate`) и прицел |
| `#mb-fire` | ЛКМ / огнемёт (удержание) |
| `#mb-alt` | ПКМ |
| `#mb-jump` | Прыжок |
| `#mb-weapon` | Смена оружия (`cycleWeapon`) |
| `#mb-scope` | Режим G (`toggleRangedMode`) |

На мобильных скрывается `#controls`, компактится HUD (`body.mobile-game`).

## Canvas WebGL

`renderer.domElement` — фокус для клавиатуры (`tabIndex = 0`). События:

- `mousedown` / `mouseup` (window) — атака, огнемёт
- `wheel` — FOV в режиме G
- `blur` — `resetMovementKeys()`, `stopFlamethrower()`

## Добавление нового UI-элемента

1. Разметка в `<body>` с уникальным `id`.
2. Стили в `<style>` (z-index выше/ниже соседей).
3. `const el = document.getElementById('...')` рядом с другими ссылками (~строка 1540).
4. Функция обновления, вызов из `animate` или по событию.
5. Запись в этот документ.
