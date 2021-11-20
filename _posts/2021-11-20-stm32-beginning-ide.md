---
layout: post
title: "[RU] STM32 Discovery: Часть 0: Подготовка"
published: true
---
## Оглавление
* [Введение](#Введение)
* [IDE](#IDE)
  * [Скачивание](#Скачивание)
  * [Установка](#Установка)
* [Документация](#Документация)

## Введение <a name="Введение"></a>
В данном цикле статей будет расмотрена работа с платой STM32 DISCOVERY (STM32F407VG MCU)  

## IDE <a name="IDE"></a>
### Скачивание <a name="Скачивание"></a>
Идём на сайт [STM32](https://www.st.com/content/st_com/en.html). Через поиск находим страницу Cube IDE и следуем указаниям(в процессе потребуется зарегистрироваться).  
Примерные шаги: Get Software -> STM32CubeIDE-Lnx: Get latest -> License Agreement: Accept -> Скачивание

### Установка <a name="Установка"></a>
[Предпологается, что CubeIDE скачана и распакована]  
Запускаем установку:
```bash
chmod +x st-stm32cubeide_1.7.0_10852_20210715_0634_amd64.sh
./st-stm32cubeide_1.7.0_10852_20210715_0634_amd64.sh
# y
# y
# Enter
# Ждём...
# y
# y
# y
# y
# В конце должны увидеть: STM32CubeIDE installed successfully
```
Поздравляю, установка завершена.

## Документация <a name="Документация"></a>
Документация для данной платы [STM32F407/417](https://www.st.com/en/microcontrollers-microprocessors/stm32f407-417.html#documentation)
* [STM32F407 Reference Manual (PDF)](https://www.st.com/resource/en/reference_manual/rm0090-stm32f405415-stm32f407417-stm32f427437-and-stm32f429439-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)  
Данный документ предоставляет информацию о взаимодействии с регистрами.  
* [STM32F407xx Datasheet (PDF)](https://www.st.com/resource/en/datasheet/stm32f405rg.pdf)
* [STM32F407VG MCU User Manual (PDF)](https://www.st.com/resource/en/user_manual/dm00039084-discovery-kit-with-stm32f407vg-mcu-stmicroelectronics.pdf)  
Описывает встроенную периферию на устройстве, например кнопки: куда и как они подключены.
