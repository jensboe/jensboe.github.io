---
layout: post
title:  "STM32 Entwicklung mit Visual Studio Code"
date:   2023-10-06 20:00
categories: STM32
---

STM32CubeIDE basiert auf Eclipse, die IDE hat schon über 20 Jahre auf dem Buckel und wirkte auf mich schon immer sehr klobig, allerdings immer noch besser als Keil
Visual Studio Code (VS Code) verwende ich schon in anderen Bereichen (unter anderem um diesen Blog zu füllen) und ist aktuell mein favorisierter Editor/IDE für alles.

Anfang des Jahres brachte STMicroelectronics die [STM32 VS Code Extension](https://marketplace.visualstudio.com/items?itemName=stmicroelectronics.stm32-vscode-extension) heraus. Mit ihr soll es sehr einfach sein.

> Die Schritte in diesem Tutorial sind auch in Github zu finden. <https://github.com/jensboe/STM32FirstSteps>
{: .prompt-tip }

# Das Projekt in Visual Studio Code öffnen

Nach der Installation der [STM32 VS Code Extension](https://marketplace.visualstudio.com/items?itemName=stmicroelectronics.stm32-vscode-extension) Erscheint in der linken Toolbar ein STM32 Schmetterlings-Symbol.
Nach einem Klick darauf kann mit `Import a local project` die `.cproject`-Datei geöffnet werden.
Während der Erstellung wird gefragt, welche Konfiguration als Standard ausgewählt werden soll. Ich entscheide mich für `Debug`.

# Build

Die Konfiguration des Projekts erfolgte mit `CMake`.Das habe ich bereits installiert.
Ebenso `ninja` als eine Alternative zum klassischem `make`. Ebenfalls installiert ist `arm-none-eabi-gcc`.
All dies ist bereits in meinem Pfad enthalten, so dass mein Ausgaben auf der Konsole wie folgt aussehen.

```shell
> cmake --version
cmake version 3.25.1

CMake suite maintained and supported by Kitware (kitware.com/cmake).
> ninja --version
1.10.2
> ninja --version
1.10.2

> arm-none-eabi-gcc --version
arm-none-eabi-gcc.exe (Arm GNU Toolchain 12.2.MPACBTI-Rel1 (Build arm-12-mpacbti.34)) 12.2.1 20230214
Copyright (C) 2022 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

Das Erstellen der Binaries erfolgt einfach per Knopfdruck auf `F7`.

# Debug

Mit `F5` wird der Debugger gestartet.
Anders als beim STM32CubeIDE hät der Debugger nicht in der `main`-Funktion, sondern im `Reset_Handler` der Datei `startup_stm32f446zetx.s`.
Aus diesem Grund habe ich mir einen manuellen Breakpoint in der `main`-Funktion gesetzt, in dieser landet der Debugger nach einem erneutem Drücken von `F5`.
Will man dieses Verhalten ändern, kann in der Datei `.vscode\launch.json` der Wert `"stopAtConnect": true` auf `"stopAtConnect": false` gesetzt werden.

![Debugansicht in Visual Studio Code](/assets/posts/STM32EntwicklungmitVisualStudioCode/debugging.jpg)
_Debugansicht in Visual Studio Code, auf der linken Seite werden die lokale Variablen, CPU-Register aber auch Peripherie-Register angzeigt und modifiziert werden. Zusätzlich ist auch eine Beschreibung der Register enthalten._


# Blick hinter die Kulissen

Bei der Umwandlung in ein VS Code Projekt werden eine vielzahl von Dateien erstellt.
`.vscode` enthält dabei Konfigurationsdateien für VS Code, `tasks.json` enthält z.B. den Build-Task, und`launch.json` die Konfiguration des Bootloaders.

Deutlich interessanter ist der Aufbau der Cmake Dateien.
`CMakeLists.txt` enthält nur den Projektnamen, erzeugt eine executable und fügt dieser st_target_properties hinzu.

```cmake
cmake_minimum_required(VERSION 3.20)

project("STM32FirstSteps" C CXX ASM)

include(cmake/st-project.cmake)

add_executable(${PROJECT_NAME})
add_st_target_properties(${PROJECT_NAME})
```

`cmake\st-project.cmake` enthält die Funktion `add_st_target_properties`.
Diese enthält compile und link definitions und options, include directories und eine Auflistung aller verwendeten source files.
Eine statische Bibliothek für z.B. die STM HAL wird nicht erstellt.

Bei den optionen und definitions werden  [cmake-generator-expressions](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html) verwendet.
Warum diese verwendet werden, erschließt sich mir nicht.
`DEBUG` wird z.B. in jeder Debug Konfiguration gesetzt, ganz unabhängig von der Sprache.
`STM32F446xx` und `USE_HAL_DRIVER` immer, außer es handelt sich im eine  Assembler Datei.
Diese Flags schaden bei Assembler sicherlich auch nicht.

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
In einen späterem Artikel werde ich die Cmake Dateien aufräumen.