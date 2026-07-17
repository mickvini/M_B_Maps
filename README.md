# M&B Maps — репозиторий карт Supreme Commander: Forged Alliance

Хранилище карт для лаунчера. Карты хранятся **архивами `.zip`** в **Git-LFS**
(не распакованными папками). Каталог `index.json` и миниатюры `previews/` лежат в
обычном git и потому лёгкие; сами архивы (тяжёлое) — в LFS.

## Структура

```
M_B_maps/
├─ .gitattributes         # *.zip и *.scmap → Git-LFS
├─ index.json             # каталог всех карт (см. схему ниже)
├─ previews/<id>.png      # миниатюры ~128px, в обычном git
└─ maps/<id>.<version>.zip# архивы карт, в Git-LFS
```

## Схема index.json

```json
{
  "version": 1,
  "updated": "2026-07-17",
  "maps": [
    {
      "id": "setons_clutch",
      "name": "Setons Clutch",
      "description": "Классическая морская карта на 8 игроков.",
      "size": "1024 × 1024",
      "players": 8,
      "mapVersion": 3,
      "fileSize": 4231212,
      "sha256": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
      "preview": "previews/setons_clutch.png",
      "archive": "maps/setons_clutch.v0003.zip"
    }
  ]
}
```

- `id` — стабильный идентификатор = имя папки карты без версии.
- `mapVersion` — число из суффикса версии (например `3` для `.v0003`).
- `sha256` — SHA-256 архива. Совпадает с LFS-`oid`, служит контролем целостности при скачивании.
- `fileSize` — размер архива в байтах.
- `preview` — путь к миниатюре внутри репо (обычный git, НЕ LFS).
- `archive` — путь к архиву внутри репо (LFS).

## Как куратору добавить новую карту

> Требуется установленные `git` и `git-lfs`.

1. Подготовьте архив: запакуйте папку карты в `maps/<id>.v0000.zip`
   (внутри архива — одна папка с `_scenario.lua`, `_save.lua`, `<имя>.scmap`).
2. Положите миниатюру `previews/<id>.png` (~128px, PNG/JPG, < 20 КБ).
3. Посчитайте sha256 и размер архива:
   ```bash
   sha256sum "maps/<id>.v0000.zip"      # → поле sha256
   stat -c%s "maps/<id>.v0000.zip"      # → поле fileSize (Linux/git-bash)
   ```
   В PowerShell:
   ```powershell
   (Get-FileHash "maps\<id>.v0000.zip" -Algorithm SHA256).Hash.ToLower()
   (Get-Item "maps\<id>.v0000.zip").Length
   ```
4. Добавьте запись в `index.json` (поля по схеме выше).
5. Закоммитьте и отправьте:
   ```bash
   git add maps/<id>.v0000.zip previews/<id>.png index.json
   git lfs install        # один раз на клоне, если ещё не делали
   git commit -m "Add <id> v0000"
   git push
   ```
   При `git add` архив автоматически уйдёт в LFS.

## Обновление карты

Увеличьте `mapVersion`, положите новый архив `maps/<id>.v000N.zip`, пересчитайте
`sha256`/`fileSize`, обновите запись в `index.json`, закоммитьте и отправьте.
Лаунчер сверяет `mapVersion` + `sha256` с локальным манифестом и помечает карту
как «Доступно обновление».

## Трафик LFS (важно)

Бесплатный лимит GitHub LFS: **2 ГБ хранения + 2 ГБ трафика в месяц** на аккаунт
(суммарно по всем скачивающим, не на пользователя). Лаунчер качает карты по одной
по требованию — именно это позволяет уложиться в лимит. Если сообщество вырастет и
трафик упрётся — переведите `archive`/`preview` на статическое хранилище
(GitHub Releases / Cloudflare R2): достаточно поменять сборку URL в `MapVaultConfig`,
клиент менять не нужно.
