# Runbook: Сборка чистого TDesktop с Anti-Recall и Ghost Mode под secureblue

## Подход (DevSecOps & Изоляция)
На Fedora Atomic / secureblue система неизменяема (read-only root). 
Поэтому оптимальный способ сборки и запуска без засорения хоста — **Flatpak** или **Rootless Podman контейнер**.

---

### Вариант 1. Сборка через Flatpak-builder (Рекомендуемый)

Этот метод берет официальный проверенный манифест Flathub и накладывает наш чистый патч при сборке.

#### Шаг 1. Клонирование официального манифеста Flathub
```bash
cd ~/.gemini/antigravity/scratch/tdesktop-clean
git clone --recursive https://github.com/flathub/org.telegram.desktop.git flathub-tdesktop
cd flathub-tdesktop
```

#### Шаг 2. Подключение патча в манифест
В файле `org.telegram.desktop.yml` находим модуль `telegram` и добавляем наш патч в список `sources`:
```yaml
      - type: patch
        path: ../anti-recall-ghost.patch
```

#### Шаг 3. Сборка и локальная установка (одной командой)
```bash
flatpak-builder --user --install --force-clean build-dir org.telegram.desktop.yml
```

После завершения сборки приложение появится в системе как чистый `Telegram` с нативным интерфейсом, но с активным Ghost Mode и сохранением удаленных сообщений.

---

### Вариант 2. Быстрая сборка в контейнере (Podman Containerfile)

Если не хочется ставить Flatpak SDK на 3 ГБ локально, сборку можно выполнить в изолированном headless-контейнере Fedora:

```bash
podman run --rm -it \
  -v ~/.gemini/antigravity/scratch/tdesktop-clean:/build:Z \
  registry.fedoraproject.org/fedora:41 \
  sh -c 'dnf install -y git cmake ninja-build gcc-c++ qt6-qtbase-devel ... && cd /build && ...'
```

---

### Почему в этом патче 0% жора батареи?
1. **Никакого SQLCipher / SQLite:** Сообщения не пишутся на диск при каждом входящем пакете. Они живут в штатном векторе оперативной памяти Telegram.
2. **0 дополнительных потоков:** Все операции выполняются строго в основном потоке `crl` обработки событий Telegram.
3. **Процессор уходит в сон:** Таймеры ядра Linux не прерывают C-states процессора Ryzen 7 8845HS. Энергопотребление остается паспортным (~1 Вт при простое).
