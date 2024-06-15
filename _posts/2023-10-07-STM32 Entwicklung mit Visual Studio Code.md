---
layout: post
title: "STM32 Entwicklung mit Visual Studio Code"
date: 2023-10-07 15:40
categories: STM32
tags: [Software, STM32, STM32CubeIDE, VS Code, Cmake]
---

> Diese Anleitung ist veraltet. ST hat die Erweiterung aktualisiert und der Weg ist nun etwas anders und vielleicht auch etwas vereinfacht. Sofern ich zeit habe, werde ich die Anleitung aktualisieren.
{: .prompt-warnung }

STM32CubeIDE, basierend auf Eclipse und bereits über 20 Jahre alt, erschien mir immer etwas schwerfällig, aber dennoch besser als Keil. Mein aktueller Favorit für die Entwicklung von STM32-Mikrocontrollern ist Visual Studio Code (VS Code), den ich bereits in anderen Projekten einsetze, einschließlich der Erstellung dieses Blogs.

Zu Beginn dieses Jahres hat STMicroelectronics die [STM32 VS Code Extension](https://marketplace.visualstudio.com/items?itemName=stmicroelectronics.stm32-vscode-extension) veröffentlicht, die die STM32-Entwicklung mit VS Code stark vereinfacht.

> Die Schritte in diesem Tutorial sind auch auf GitHub verfügbar. [STM32FirstSteps](https://github.com/jensboe/STM32FirstSteps)
{: .prompt-tip }

## Projekt in Visual Studio Code öffnen

Nach der Installation der [STM32 VS Code Extension](https://marketplace.visualstudio.com/items?itemName=stmicroelectronics.stm32-vscode-extension) findest du ein STM32-Schmetterlings-Symbol in der linken Symbolleiste. Mit einem Klick darauf kannst du deine Projektdatei `.cproject` öffnen. Während der Einrichtung wird nach der Standardkonfiguration gefragt, und ich wähle `Debug` aus.

## Build

Die Projektkonfiguration erfolgt mithilfe von CMake, das ich bereits installiert habe. Ebenso habe ich `ninja` als Alternative zu `make` sowie `arm-none-eabi-gcc` installiert. Alle diese Tools sind bereits in meinem Systempfad enthalten, daher sehen meine Konsolenausgaben wie folgt aus:

```shell
> cmake --version
cmake version 3.25.1

CMake suite maintained and supported by Kitware (kitware.com/cmake).
> ninja --version
1.10.2
> arm-none-eabi-gcc --version
arm-none-eabi-gcc.exe (Arm GNU Toolchain 12.2.MPACBTI-Rel1 (Build arm-12-mpacbti.34)) 12.2.1 20230214
Copyright (C) 2022 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

Das Erstellen der Binärdateien erfolgt einfach per Mausklick auf `F7`.

## Debugging

Mit `F5` startest du den Debugger. Im Gegensatz zu STM32CubeIDE startet der Debugger nicht in der `main`-Funktion, sondern im `Reset_Handler` der Datei `startup_stm32f446zetx.s`. Daher habe ich manuell einen Breakpoint in der `main`-Funktion gesetzt, zu der der Debugger nach erneutem Drücken von `F5` gelangt. Wenn du dieses Verhalten ändern möchtest, kannst du in der Datei `.vscode\launch.json` den Wert ``stopAtConnect`: true` auf ``stopAtConnect`: false` ändern.

![Debugansicht in Visual Studio Code](/assets/posts/STM32EntwicklungmitVisualStudioCode/debugging.jpg)
_Debugansicht in VS Code: Auf der linken Seite werden lokale Variablen, CPU-Register sowie Peripherie-Register angezeigt und können bearbeitet werden. Darüber hinaus gibt es eine Beschreibung der Register._

## Ein Blick hinter die Kulissen

Beim Konvertieren in ein VS Code-Projekt werden eine Vielzahl von Dateien erstellt. Im Verzeichnis `.vscode` findest du Konfigurationsdateien für VS Code. `tasks.json` enthält z.B. den Build-Task, und `launch.json` enthält die Konfiguration für den Bootloader.

Aber noch interessanter ist die Struktur der CMake-Dateien. Die Datei `CMakeLists.txt` enthält lediglich den Projektnamen, erstellt ein ausführbares Programm und fügt diesem die erforderlichen Eigenschaften hinzu:

```cmake
cmake_minimum_required(VERSION 3.20)

project("STM32FirstSteps" C CXX ASM)

include(cmake/st-project.cmake)

add_executable(${PROJECT_NAME})
add_st_target_properties(${PROJECT_NAME})
```

Die Datei `cmake\st-project.cmake` enthält die Funktion `add_st_target_properties`, die Kompilierungs- und Link-Definitionen, Optionen, Include-Verzeichnisse und eine Liste aller verwendeten Quelldateien enthält. Es wird leider keine seperate statische Bibliothek für die STM HAL erstellt.

Die Verwendung von [CMake Generator Expressions](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html) in den Optionen und Definitionen mag auf den ersten Blick verwirrend sein. Zum Beispiel wird `DEBUG` in jeder Debug-Konfiguration gesetzt, unabhängig von der Sprache. `STM32F446xx` und `USE_HAL_DRIVER` werden immer gesetzt, außer in Assemblerdateien. Diese Flags schaden jedoch auch in Assemblerdateien nicht.

```cmake
target_compile_definitions(
    ${TARGET_NAME} PRIVATE
    "$<$<AND:$<CONFIG:Debug>,$<COMPILE_LANGUAGE:ASM>>:DEBUG>"
    "$<$<AND:$<CONFIG:Debug>,$<COMPILE_LANGUAGE:C>>:DEBUG>"
    "$<$<AND:$<CONFIG:Debug>,$<COMPILE_LANGUAGE:C>>:USE_HAL_DRIVER>"
    "$<$<AND:$<CONFIG:Debug>,$<COMPILE_LANGUAGE:C>>:STM32F446xx>"
    "$<$<AND:$<CONFIG:Debug>,$<COMPILE_LANGUAGE:CXX>>:DEBUG>"
    "$<$<AND:$<CONFIG:Debug>,$<COMPILE_LANGUAGE:CXX>>:USE_HAL_DRIVER>"
    "$<$<AND:$<CONFIG:Debug>,$<COMPILE_LANGUAGE:CXX>>:STM32F446xx>"
    "$<$<AND:$<NOT:$<CONFIG:Debug>>,$<COMPILE_LANGUAGE:C>>:USE_HAL_DRIVER>"
    "$<$<AND:$<NOT:$<CONFIG:Debug>>,$<COMPILE_LANGUAGE:C>>:STM32F446xx>"
    "$<$<AND:$<NOT:$<CONFIG:Debug>>,$<COMPILE_LANGUAGE:CXX>>:USE_HAL_DRIVER>"
    "$<$<AND:$<NOT:$<CONFIG:Debug>>,$<COMPILE_LANGUAGE:CXX>>:STM32F446xx>"
)
```

In einem nachfolgendem [Artikel]({%post_url 2023-10-08-Cmake aufräumen und git submodule nutzen %}) habe ich die CMake-Dateien optimieren.