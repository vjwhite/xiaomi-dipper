# Ubuntu Touch 24.04 для Xiaomi Mi 8 (dipper)

Порт Halium 9. Ядро остаётся ветка `halium-9.0` из `kernel-xiaomi-sdm845`, собирается GCC 4.9. Корень — Ubuntu Touch 24.04-1.x. Раздел `system` на этом аппарате ровно 3 ГиБ, и туда не влезает ни 24.04-2.x, ни 24.04-1.x. Образ кладётся файлом `ubuntu.img` на userdata, там около 50 ГиБ. Параметра `systempart` в командной строке ядра нет: иначе initramfs игнорирует файл и монтирует тесный раздел.

TWRP уже стоит в разделе recovery. Эта сборка его не заменяет и не трогает vendor Android 9.

## Что ставится

Из артефакта CI нужны два файла:

- `boot.img` — ядро и initramfs, меньше 64 МиБ, прошивается в раздел Boot
- `rootfs.img` — файл корня, на телефоне он должен называться `/data/ubuntu.img`

Разделы System, Vendor и Recovery не прошивать.

## Сборка

GitHub Actions по пушу в `main` выполняет `build.sh`, затем собирает `system.img` из общего корня 24.04-1.x и архива устройства. Готовые файлы лежат в артефакте `dipper-24.04-images`.

Локально на компьютере с Ubuntu 22.04:

```bash
./build.sh
./build/prepare-fake-ota.sh out/device_dipper.tar.xz ota
./build/system-image-from-ota.sh ota/ubuntu_command out
```

Каталог `build/` при первой сборке клонируется из общих инструментов UBports и в репозиторий не входит.

## Прошивка через установленный TWRP

1. Пока телефон ещё загружен в 16.04, скопировать нужные файлы из `/home/phablet`.
2. Перезагрузиться в TWRP. С компьютера снять копии разделов `boot`, `system`, `recovery` и `vendor`.
3. Wipe → Format Data → `yes`, затем снова загрузка в TWRP. Файлы на userdata после этого стёрты.
4. `adb push boot.img /tmp` и `adb push rootfs.img /data/ubuntu.img`.
5. Install → Install Image: только `boot.img` в Boot.
6. Reboot System. TWRP остаётся в recovery.

## Оверлей

Задания Upstart удалены. После старта Android-контейнера выполняется `device-hacks`: cpuset, bind cache на userdata, metadata, прошивка IPA и блокировка клавиш сканеров отпечатка. Bluetooth ждёт сервис через drop-in `bluebinder.service`, подсветка идёт через sysfs. USB-идентификаторы взяты из `init.qcom.usb.rc` этого аппарата: vendor `2717`, MTP `FF40` / `FF48`, RNDIS `FF80` / `FF88`.
