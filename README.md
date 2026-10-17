# DeepSeek Studio Pro

Desktop-приложение на базе **Tauri** (Rust) с фронтендом на **Next.js**, объединяющее работу с DeepSeek, генерацию изображений через **ComfyUI Portable** и распознавание речи через **Whisper**. Готовые установщики собираются автоматически через GitHub Actions.

## Возможности

- 🤖 Работа с моделями DeepSeek (чат, анализ, код)
- 🎨 Генерация изображений через интегрированный ComfyUI Portable с набором готовых workflow'ов
- 🎙️ Распознавание речи локально с помощью Whisper
- 🖥️ Нативные приложения для Windows (NSIS-установщик) и macOS (DMG)
- 📦 Автономная поставка: все ресурсы (ComfyUI, Whisper, модели) включаются в дистрибутив

## Архитектура проекта

| Компонент | Технология | Расположение |
|---|---|---|
| Фронтенд | Next.js (статический export в `out/`) | `src/` |
| Desktop-оболочка | Tauri (Rust) | `src-tauri/` |
| Ресурсы ИИ | ComfyUI Portable, Whisper, workflow'ы | `comfyui-portable/`, `whisper/`, `workflows/` |
| Скрипты сборки | PowerShell / Bash / NSIS | `scripts/` |
| CI/CD | GitHub Actions | `.github/workflows/` |

## Структура репозитория

```
.
├── .github/workflows/release.yml   # CI: сборка Windows-установщика и macOS DMG
├── src/                            # Фронтенд (Next.js)
├── src-tauri/                      # Бэкенд (Tauri / Rust)
└── scripts/                        # download-resources.(ps1|sh), installer.nsi, create-dmg.sh
```

## Сборка из исходников

### Требования

- [Node.js](https://nodejs.org/) 20+
- [Rust](https://www.rust-lang.org/tools/install) (стабильный тулчейн)
- [Tauri CLI](https://tauri.app/start/prerequisites/) (`cargo install tauri-cli`)
- Windows: NSIS 3.09 + плагин nsProcess; macOS: `brew install create-dmg`

### Шаги

1. **Загрузка ресурсов** (ComfyUI Portable, Whisper, модели):

   ```powershell
   # Windows
   ./scripts/download-resources.ps1
   ```

   ```bash
   # macOS / Linux
   ./scripts/download-resources.sh
   ```

   При необходимости передайте ссылки на приватные архивы через переменные окружения
   `RESOURCES_URL_COMFYUI` и `RESOURCES_URL_WHISPER`.

2. **Сборка фронтенда:**

   ```bash
   cd src
   npm ci
   npm run build   # генерирует статическую папку out/
   ```

3. **Сборка Tauri-бинарника:**

   ```bash
   cd src-tauri
   cargo tauri build --release
   ```

4. **Упаковка:**

   - Windows: `makensis.exe ./scripts/installer.nsi` → `DeepSeekStudio_Setup_<версия>.exe`
   - macOS: `./scripts/create-dmg.sh` → `DeepSeekStudio_<версия>.dmg`

## Выпуски (GitHub Actions)

Релизный воркфлоу запускается автоматически при создании тега вида `v*`
(например, `v0.7.2`) либо вручную через **Run workflow**:

```bash
git tag v0.7.2
git push origin v0.7.2
```

Пайплайн выполняет две параллельные сборки:

- **build-windows** (windows-latest): Node 20 → Rust → загрузка ресурсов → сборка Next.js → `cargo tauri build` → подготовка каталога `build/DeepSeekStudio` → NSIS-установщик.
- **build-macos** (macos-latest): те же шаги + ad-hoc подпись `.app` и создание DMG через `create-dmg`.

Готовые артефакты публикуются как artifacts: `DeepSeekStudio-Windows-Setup` и `DeepSeekStudio-macOS-DMG`.

### Необходимые секреты репозитория

| Секрет | Назначение |
|---|---|
| `RESOURCES_URL_COMFYUI` | Ссылка на архив ComfyUI Portable |
| `RESOURCES_URL_WHISPER` | Ссылка на архив Whisper |

Текущая релизная версия задаётся в `PRODUCT_VERSION` внутри `release.yml` (сейчас — **0.7.2**).

## Лицензии

Сторонние лицензии компонентов (ComfyUI, Whisper и др.) включаются в дистрибутив (папка `licenses/`).
