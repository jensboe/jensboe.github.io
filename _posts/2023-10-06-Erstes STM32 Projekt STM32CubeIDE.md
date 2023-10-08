---
layout: post
title:  1. STM32 Projekt mit STM32CubeIDE
date:   2023-10-06
categories: STM32
tags: Software STM32 STM32CubeIDE
---

Ich verfüge unter anderem über ein NUCLEO-F446ZE.
Dieses werde ich in einer kleinen Serie zuerst mit dem STM32CubeIDE in Betrieb nehmen und ein kleines Blink-Projekt erstellen und debuggen.
Im weiterem Verlauf soll dann zuerst auf das STM32CubeIDE verzichtet werden und durch das Visual Studio Code eingesetzt werden.
Da ich grundsätzlich mit git, einem Version Control System arbeite, plane ich im weiteren Verlauf, die STM32 HAL und die CMSIS nicht als Kopie sondern als Submodul einzubinden.

Danach wird geprüft in wie weit man mit C++ statt mit C entwickeln kann und die STM32 HAL durch eine eigene C++-HAL ersetzt werden.

Aber fangen wir ersteinmal vorne an.

> Die Schritte in diesem Tutorial sind auch auf GitHub verfügbar. [STM32FirstSteps](https://github.com/jensboe/STM32FirstSteps)
{: .prompt-tip }

## Neues Projekt in STM32CubeIDE anlegen
Funktioniert sehr gradlinig Erklärung nur der Vollständigkeit halber.
Mit `New` -> `STM32 Project` ein neues Projekt anlegen und Namen eingeben.
Im `Target Selector` kann der verwendete Controller ausgewählt werden, es kann jedoch auch direkt das verwendete Board von STM ausgewählt werden.
Dies erspart die Konfiguration der Pins.

In den Targeted Languages habe ich `C++` angegeben, das Projekt wird ein `Executable` und als Typ habe ich `STM32Cube` ausgewählt.

![Im gezeigtem Target Selector kann aus MCU/MPU, Boards und Beispielen ein Target ausgewählt werden](/assets/posts/ErstesSTM32ProjektSTM32CubeIDE/target_selector.jpg)
_Target selection_

Auf der nächsten Seite wird das Framework Package ausgewählt und man kann einstellen wie die library files eingebunden werden.
Da ich sie später via git submodules eh anders einbinden will, wähle ich hier `Copy only necessary library files`.
ZUm Abschluss wird abgefragt ob man alle Peripherie im `default mode` initialisieren will.
Das will ich.

Im Anschluss öffnet sich das Projekt und die `Pinout & Configuration`.
Hier kann die Pin Belegung nachvollzogen und geändert werden.

![Pinout Configuration zeigt den Prozessor in einer Draufsicht und koloriert die Pins entsprechend ihrer Funktion.](/assets/posts/ErstesSTM32ProjektSTM32CubeIDE/pinout_configuration.jpg)
_Pinout & Configuration_

Neben `Pinout & Configuration` gibt es zahlreiche weitere Einstellungen.
Besonders praktisch finde ich `Clock Configuration`, da hier der clock tree des µC nachvollzogen und eingestellt werden kann.
Bei Änderungen erscheint im `Projekt Explorer` inter der *.ioc -Datei der Hinweis `[Code Generation is required]`.
Diese kann im Reiter `Project` oder per Linksklick auf die *.ioc-Datei gestartet werden.

## Debuggen
Das Debuggen wird mit dem Klick auf den Käfer Toolbar-Knopf gestartet.
![Debug Button in Form eines Käfers.](/assets/posts/ErstesSTM32ProjektSTM32CubeIDE/debugbutton.jpg)
_Debug button_

Der Build-Prozess startet und STM32CubeIDE will daraufhin eine neue Ansicht öffnen, die spezielle für das Debuggen optimiert wurde.
Ich stimme dem zu.
Daraufhin (wenn man das Board angeschlossen hat) wird die `main.c` Datei geöffnet und eine grüne Hervorhebung zeigt an, an welcher Stelle das Programm gestoppt wurde.

![Grüne Hervorhebung der 1. Instruktion an der der Debugger anhält.](/assets/posts/ErstesSTM32ProjektSTM32CubeIDE/debug_firststop.jpg)
_Debugger hält an erster Instruction der main function_

Mit `Step Into` (`F5`) , `Step Over` (`F6`), `Resume` (`F85`) und `Suspend` kann der Debugger gesteuert werden.
Mit `Terminate` (`STRG` + `F2`) kann das Debuggen beendet werden.

## Das erste eigene Programm schreiben

Das erste eigene Programm soll eine LED blinken lassen.
Daher schaue ich mir zuerst die `main`-Funktion in der `main.c`-Datei an.

```c
/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_USART3_UART_Init();
  MX_USB_OTG_FS_PCD_Init();
  /* USER CODE BEGIN 2 */

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
```
Die `main`-Funktion besteht aus Anweisungen zum Initialisieren des Boards und einer abschließenden endlosen `while`-Schleife.
Diese ist bei einem frisch erstelltem Projekt jedoch leer.
Funktionsaufrufe, die mit `HAL_` beginnen, rufen Funktionen aus der HAL-Bibliothek auf.`MX_` kennzeichnet Funktionen, die von der Codegenerierung erstellt wurden
Ein Überrest des STM32Cube**MX**?
`SystemClock_Config` wird ebenfalls von der Codegenerierung erzeugt, hat jedoch keinen Prefix.

Die Funktion ist von `USER CODE` Kommentare durchsetzt.
Code zwischen `BEGIN` und `END` bleibt bei einer erneuten Codegenerierung erhalten, Code außerhalb der Blöcke wird überschrieben.

Um das Ziel "Blinken einer LED" zu erreichen, füge ich deshalb folgendes in den `USER CODE` ein, so dass die while-Schleife wie folgt aussieht.
```c
   /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
    HAL_GPIO_TogglePin(LD1_GPIO_Port, LD1_Pin);
    HAL_Delay(500);
  }
  /* USER CODE END 3 */
```
Wird der Debugger nun gestartet, wird beim Durchsteppen die LD1 aktiviert bzw. deaktiviert.
Verlässt man den Step-Betrieb, blinkt die LD1 in einem 1Hz Takt.


## Weiterführende Links
* [From Zero to main(): Bare metal C](https://interrupt.memfault.com/blog/zero-to-main-1)<br>
Der Beginn der Zero to main() Serie.
In diesem Artikel wird beschreiben, wie ein µC nach dem einschalten in der `main`-Funktion landet und was dafür notwendig ist.
