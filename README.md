# epkg-packets

Пакетный репозиторий проекта [epkg-tools](https://github.com/mel0k1) —
универсального пакетного менеджера уровня apk-tools для хобби-ОС.
Формат пакетов: `.epkg` = `tar.gz` + `PKGINFO`, индекс — `index.json`,
подписи — signify/ed25519.

## Структура репозитория

```
├── index.json          индекс: имя, версия, sha256, размер, deps
├── index.sig           ed25519-подпись index.json (ключ official)
├── official.pub        публичный ключ официального подписанта
├── demo.pub            публичный ключ демо/тестового подписанта
└── packages/
    ├── foo-1.0.epkg            тестовый пакет (программа /usr/bin/foo)
    ├── foo-1.0.epkg.sig        его отсоединённая подпись
    ├── bar-2.1.epkg            тестовый пакет, зависит от foo
    └── bar-2.1.epkg.sig        его отсоединённая подпись
```

## Использование с epkg

```sh
cat > epkg.conf <<'EOF'
mirror = https://raw.githubusercontent.com/mel0k1/epkg-packets/main
db     = /var/lib/epkg
cache  = /var/cache/epkg
pubkey = /путь/к/official.pub     # проверка подписи индекса при update
EOF

epkg update            # скачает index.json и проверит index.sig
epkg install bar       # поставит bar, зависимость foo подтянется сама
epkg verify packages/foo-1.0.epkg --index index.json
```

## Проверка отдельного пакета офлайн

```sh
epkg verify packages/foo-1.0.epkg                    # структура+манифест+подпись
epkg verify packages/foo-1.0.epkg --index index.json # + сверка с индексом
epkg audit --file index.json --sig index.sig --pubkey official.pub
```

`epkg verify` выполняет: структурную проверку (gzip+tar+PKGINFO),
пересчёт манифест-хеша содержимого, сверку с индексом (`--index`) и
проверку ed25519-подписи (ключ: `--pubkey`, `pubkey =` в epkg.conf
либо `official.pub`/`demo.pub` из текущего каталога).

## Ключи

* `official.pub` — комментарий `epkg-packets official signing key`,
  keynum `daec141d06f04d55`. Подписывает `index.sig` и пакеты.
* `demo.pub` — тестовый ключ для экспериментов.

Секретные ключи хранятся офлайн у мейнтейнера и в репозиторий не
попадают никогда. Публичные файлы и подписи совместимы с OpenBSD
signify (`signify -V -p official.pub -m index.json`).

## Пересборка индекса

Индекс генерируется по `packages/*.epkg` (одна запись на имя, свежая
версия) и подписывается:

```sh
./epkg-key sign -s official.sec -m index.json -x index.sig
```
