---
layout: post
title: "Cmake aufräumen & git submodule nutzen"
date: 2023-10-08 10:30
categories: STM32
tags: [Software, STM32, Cmake, git, git submodule]
---
Nachdem im vorherigem [Artikel]({%post_url 2023-10-07-STM32 Entwicklung mit Visual Studio Code %}) wurde das von der [STM32 VS Code Extension](https://marketplace.visualstudio.com/items?itemName=stmicroelectronics.stm32-vscode-extension) generierte CMake bereits kurz analysiert.
Der generierte Code soll nun aufgeräumt werden und statt stumpfer Datei Kopien sollen externe Bibliotheken nun per git submodule funktion eingebunden werden.


> Die Schritte in diesem Tutorial sind auch auf GitHub verfügbar. [STM32FirstSteps](https://github.com/jensboe/STM32FirstSteps)
{: .prompt-tip }

## CMake Generator Expressions aufräumen
In der Datei `cmake/st-project.cmake` tummeln sich zahlreiche Generator Expressions, diese werden im ersten Schritt aufgeräumt.

### target_compile_definitions
Durch den Wegfall von überflüssigen Unterscheidungen kürzen sich die target_compile_definitions auf eine Generator Expressions für eine Debug-Definition herunter.
Ansonsten ist `USE_HAL_DRIVER` und `STM32F446xx` immer definiert

```cmake
target_compile_definitions(
    ${TARGET_NAME} PRIVATE
    $<$<CONFIG:Debug>:DEBUG>
    USE_HAL_DRIVER
    STM32F446xx
)
```

### target_include_directories
Sämtliche Unterscheidungen sind überflüssig.
Die Mischung von Slashes und Backslashes (mit Backslash als Escape-Zeichen) macht alles sehr unansehnlich.
Cmake kommt auch bei Windows-Systemen mit normalen Slashes zurecht.

```cmake
target_include_directories(
    ${TARGET_NAME} PRIVATE
    ${PROJECT_SOURCE_DIR}/Core/Inc
    ${PROJECT_SOURCE_DIR}/Drivers/STM32F4xx_HAL_Driver/Inc
    ${PROJECT_SOURCE_DIR}/Drivers/STM32F4xx_HAL_Driver/Inc/Legacy
    ${PROJECT_SOURCE_DIR}/Drivers/CMSIS/Device/ST/STM32F4xx/Include
    ${PROJECT_SOURCE_DIR}/Drivers/CMSIS/Include
)
```

### target_compile_options

Die target_compile_options unterscheiden zwischen Debug und nicht Debug Builds.
Im generierten Code wird bei "Nicht Debug" via `-Os` Flag die Größe optimiert.

```cmake
target_compile_options(
    ${TARGET_NAME} PRIVATE
    $<$<CONFIG:Debug>:-g3>
    $<$<NOT:$<CONFIG:Debug>>:-g0>
    $<$<NOT:$<CONFIG:Debug>>:-Os>
    -mcpu=cortex-m4
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
)
```

### target_link_options

Kürzen sich runter auf
```cmake
target_link_options(
    ${TARGET_NAME} PRIVATE
    -mcpu=cortex-m4
    -mfpu=fpv4-sp-d16
    -mfloat-abi=hard
    -T
    ${PROJECT_SOURCE_DIR}/STM32F446ZETX_FLASH.ld
)
```
> Alle Änderungen dieses Abschnitts in einem [git commit](https://github.com/jensboe/STM32FirstSteps/commit/fcdf9a1cd9e0282fdffe99f7473e3381256577a2)
{: .prompt-info }

## Bibliothekscode abtrennen

Statt das Projekt in der `cmake/st-project.cmake` zu definieren, wird das Projekt nun aufgeteilt.
In `CMakeLists.txt` im `Root` Verzeichnis wird eine `HAL`-Bibliothek definiert, in dieser sind alle Includes und Sources aus dem `Driver` Verzeichnis enthalten.
In `CMakeLists.txt` im `Core` Verzeichnis wird eine `CORE`-Executable definiert, in dieser sind Core Includes und Sources enthalten.
Zusätzlich wird die `CORE`-Executable mit der `HAL`-Bibliothek verknüpft.
Die in der st-project.cmake definierte `add_st_target_properties`-Funktion wird in `add_common_target_properties` unbenannt.
Aus der `add_common_target_properties` werden alle Includes und Sources definitionen entfernt.
Auch die Costom Commands werden aus der funktion entfernt und seperate für die `CORE`-Executable wieder eingefügt.

> Alle Änderungen dieses Abschnitts in einem [git commit](https://github.com/jensboe/STM32FirstSteps/commit/af4eca32f44f8d942fdfb4ceaae7784bd81c2dc5)
{: .prompt-info }

## stm32f4xx_hal_driver als git submodule einfügen
Aktuell liegt die von STM bereitgestellte HAL noch im `Drivers`-Verzeichnis als "echte" Datei.
Externe Bibliotheken will ich jedoch in einem am liebsten nur als submodule oder anders als Paket einbinden.
Deshalb klone ich mir das [Repository von Github](https://github.com/STMicroelectronics/stm32f4xx_hal_driver) als Submodule via
```shell
git submodule add https://github.com/STMicroelectronics/stm32f4xx_hal_driver ext/stm32f4xx_hal_driver
```
In der `CMakeLists.txt` werden alle `Drivers/STM32F4xx_HAL_Driver` durch `ext/stm32f4xx_hal_driver` ersetzt.
Abschließend kann das `Drivers/STM32F4xx_HAL_Driver`-Verzeichnis gelöscht werden.

> Alle Änderungen dieses Abschnitts in einem [git commit](https://github.com/jensboe/STM32FirstSteps/commit/27a622b437c9be010b3b07f50248193ffb0fa255)
{: .prompt-info }

## stm32f4xx_hal_driver in seperater cmake Datei definieren
Statt weiterhin die `CMakeLists.txt` zu benutzen wird eine weitere `CMakeLists.txt`im `ext` Verzeichnis angelegt, sowie eine `ext/stm32f4xx_hal_driver.cmake`.
In letzter landet nun die `HAL` bzw. `stm32f4xx_hal_driver`-Bibliotheks-Definition.
Da die Datei nun in einem Unterverzeichnis liegt, müssen die Pfade angepasst werden.

## cmsis_device_f4 als git submodule einfügen
Auch `cmsis_device_f4` wird durch ein git submodule ersetzt.

```shell
git submodule add https://github.com/STMicroelectronics/cmsis_device_f4 ext/cmsis_device_f4
```

Ebenso wird eine separate Bibliothek in `ext/cmsis_device_f4.cmake` erzeugt. 
Da nur Header relevant sind, wird die Bibliothek als `INTERFACE` definiert.
```cmake
add_library(cmsis_device_f4 INTERFACE)

target_include_directories(
    cmsis_device_f4 INTERFACE
    cmsis_device_f4/Include
)
```
In `ext/stm32f4xx_hal_driver.cmake` werden die entsprechenden Includes entfernt.
Auch die Dateien im Verzeichnis `Drivers/CMSIS/Device/ST/` werden gelöscht.

> Alle Änderungen dieses Abschnitts in einem [git commit](https://github.com/jensboe/STM32FirstSteps/commit/c057a9cf148131cf12ae7feed0b39056f6c58f6d)
{: .prompt-info }

## cmsis_5 submodule einfügen
Wie schon bei `cmsis_device_f4` das Submodule hinzufügen.

```shell
git submodule add https://github.com/ARM-software/CMSIS_5 ext/cmsis_5
```

`ext/CMSIS_5.cmake` enthält das passende Include Verzeichnis, während es in `ext/stm32f4xx_hal_driver.cmake` gelöscht wird.
Der Ordner `Drivers` kann nun vollständig gelöscht werden.

```cmake
add_library(cmsis_5 INTERFACE)

target_include_directories(
    cmsis_5 INTERFACE
    CMSIS_5/CMSIS/Core/Include
)
```
> Alle Änderungen dieses Abschnitts in einem [git commit](https://github.com/jensboe/STM32FirstSteps/commit/126f253eb84f4dccfc1058e536cc77ddc36e0a9a)
{: .prompt-info }

## Startup Datei ersetzen
Aktuelle wird `Core/Startup/startup_stm32f446zetx.s` als Startup Datei verwendet.
Die Datei `ext/cmsis_device_f4/Source/Templates/gcc/startup_stm32f446xx.s` enthält den selben Inhalt.

Daher entferne ich die Datei aus `Core/CMakeLists.txt` und füge in `ext/cmsis_device_f4.cmake` die Datei `cmsis_device_f4/Source/Templates/gcc/startup_stm32f446xx.s` hinzu.
Dazu muss der Bibliothekstyp von `INTERFACE` auf `STATIC` geändert werden.
> Alle Änderungen dieses Abschnitts in einem [git commit](https://github.com/jensboe/STM32FirstSteps/commit/fff848f5f658ed02cc24987ecc37fca937a8d576)
{: .prompt-info }

## system_stm32f4xx.c Datei ersetzen
Die `Src/system_stm32f4xx.c` Datei ist ebenfalls in cmsis_device_f4 enthalten.
Um sie zu nutzen, muss sie in `ext\cmsis_device_f4.cmake` hinzugefügt werden.
Der `target_include_directories` muss von `INTERFACE` auf `PUBLIC` geändert werden.
Und die `cmsis_5` Bibliothek muss verlinkt werden.
Ebenfalls müssen nun `common_target_properties` hinzugefügt werden.

```cmake
add_library(cmsis_device_f4 STATIC
    cmsis_device_f4/Source/Templates/gcc/startup_stm32f446xx.s
    cmsis_device_f4/Source/Templates/system_stm32f4xx.c
)

target_include_directories(
    cmsis_device_f4 PUBLIC
    cmsis_device_f4/Include
)
target_link_libraries(cmsis_device_f4 PUBLIC
    cmsis_5
)
add_common_target_properties(cmsis_device_f4)
```

> Alle Änderungen dieses Abschnitts in einem [git commit](https://github.com/jensboe/STM32FirstSteps/commit/ef481bae643652d14c6988f00eba2e4d5da5a5b0)
{: .prompt-info }