---
layout: post
title:  "Настройка Sublime Text 3, SW4 и STM32CubeMX для разработки STM32 под Windows 10"
date:   2017-11-30 16:42:14 +0300
categories: stm32 sublime-text-3 SW4STM32
---

Данная статья рассказывает о том, по шагам, как настроить связку [Sublime text 3](https://www.sublimetext.com/3), [AC6 System Workbench for STM32](http://www.openstm32.org/) и [STM32CubeMX](http://www.st.com/en/development-tools/stm32cubemx.html) в качестве среды разработки для [STM32](http://www.st.com/en/microcontrollers/stm32-32-bit-arm-cortex-mcus.html) под MS Windows x64. 

> Аналогичная инструкция по настройке среды в Linux лежит [тут](/stm32/sublime-text-3/sw4stm32/2017/11/29/st3_SW4_and_STM32CubeMX_for_STM32_under_Linux.html).

> Инструкция дана для абсолютно чистой машины на базе **MS Windows 10 x64**, в
> вашем случае могут быть уже установлены какие-то компоненты. В этой
> статье и в последующих буду приводить примеры для отладочной платы
> [STM32F3DISCOVERY](http://www.st.com/en/evaluation-tools/stm32f3discovery.html).
> Она основана на МК STM32F303VCT6 c 256-Кбайт Flash и 48-КБайт RAM в
> корпусе LQFP100. Вы можете адаптировать настройки под ваш МК очень
> легко благодаря STM32CubeMX.

<!--more-->
Что в итоге у меня получилось:
-  Sublime Text 3 с автодополнениями и подсветкой всех функций, включая HAL и остальные библиотеки проекта;
- AC6 System Workbench for STM32 с кастомизированными перспективами и прочими плюшками;
- Конечно же использую контроль версий в Git.

## Предварительно устанавливаем необходимый софт
### Создадим директорию для хранения библиотек и проектов
Я предпочитаю хранить библиотеки на всех своих машинах по одному и тому же пути. Т.к. для компиляторов в общем случае нужны абсолютные пути (тот же CubeMX генерирует абсолютные пути к библиотекам), я создаю отдельную директорию в корне второго раздела диска. У меня это везде **D:\Libs**. 

Аналогично и для проектов я тоже предпочитаю создавать директорию на отдельном, несистемном разделе. В моём случае это **D:\workspace**.

### Устанавливаем Oracle JRE
Скачиваем [дистрибутив](https://www.java.com/ru/download/) с сайта Oracle. Ставим со всеми настройками по-умолчанию.

## Устанавливаем STM32CubeMX
Действуем согласно [инструкции самого ST](http://www.st.com/content/ccc/resource/technical/document/user_manual/10/c5/1a/43/3a/70/43/7d/DM00104712.pdf/files/DM00104712.pdf/jcr:content/translations/en.DM00104712.pdf).

Скачиваем [дистрибутив](http://www.st.com/en/development-tools/stm32cubemx.html#getsoftware-scroll) с сайта ST (потребуется регистрация), разархивируем и запустим установщик. Пока ставим со всеми настройками по-умолчанию.

Запускаем STM32CubeMX, идём в настройки **Help->Updater Settings** и меняем путь для хранения библиотек на **D:\Libs\STM32Cube\Repository**.

![Скрин](/assets/img/stm32-windows-ide/CubeMX-Setup-1.png)

Установим библиотеку для STM32F3. Открываем **Help->Install New Libriaries**, ставим галку  *Firmware Package for Family STM32F3*, жмём **Install Now**

![Скрин](/assets/img/stm32-windows-ide/CubeMX-Setup-2_F3.png)

## Устанавливаем Sublime Text 3
Вообще, SW4 вполне самодостаточная IDE. Но я люблю кодить именно в ST3, а компиляция и дебаг в SW4. Инструкция по установке ST3 для любых дистрибутивов лежит [тут](https://www.sublimetext.com/docs/3/linux_repositories.html).  

[Скачиваем](https://www.sublimetext.com/3), устанавливаем, никаких чудес

Для работы некоторых плагинов, надо установить несколько компонентов.

- [Распространяемый пакет Microsoft Visual C++ 2010 (x64)](https://www.microsoft.com/ru-ru/download/confirmation.aspx?id=14632)
- [CppCheck](http://cppcheck.sourceforge.net/)
- [Clang](http://releases.llvm.org/download.html#svn), качаем версию **Clang for Windows** 

При установке Clang надо выбрать галку Add LLVM to the System PATH:
![Скрин](/assets/img/stm32-windows-ide/llvm-install-1.png)

В процессе установки выдаст ошибку при попытке интегрировать в MS VisualStudio. Просто закройте окно.

### Настройка Sublime Text 3
Для начала установим Package Control.
Запускаем ST3 и жмём **CTRL+`**, в командную строку вставляем код и жмём ENTER:

```python
import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

![](/assets/img/stm32-linux-ide/Sublime-install-1.png)

Для полноценной работы нам понадобятся такие пакеты:
- ARM Assembly (Подсветка Assembler в коде)
- C Improved (Подсветка C)
- CMakeEditor (Подсветка Cmake)
- DocBlokr (Форматирование комментариев)
- EasyClangComplete (автодополнение функций)
- Hex to Int preview (показывает значение HEX)
- SublimeAStyleFormatter (форматирует код нажатием CTRL+SHIFT+A)
- Sublimelinter
- Sublimelinter-contrib-cmakelint (Подсветка cmake)
- Sublimelinter-cppcheck (проверяет код на ошибки с помощью cppcheck)
- Sublimelinter-annotations (Подсвечивает FIXME, NOTE, TODO и т.д.)

Устанавливаем нужные пакеты из меню **Preferences -> Package Control -> Install Package**.

### Настраиваем пакеты ST3
#### EasyClangComplete
Настраиваем *clang* на *C99* и указываем директории с базовыми библиотеками вроде *StdLib*.

Открываем **Preferences -> EasyClangComplete -> Settings** и вставляем в правую половину:

```json
{
    "common_flags" : [
        "-IC:/Program Files/LLVM/include/clang-c",
      ],
    "c_flags" : [ "-std=c99" ],
    "clang_binary" : "clang",
    "verbose" : false,
    "use_libclang_caching": true,
}
```

#### Sublimelinter
- Линтер срабатывает только при загрузке/сохранении
- Аннотации FIXME обводит как ошибку, остальные как предупреждение
- CppCheck проверяет только правила Warning, Style, Performance и Portability, а также указываем стандарты C99 и C++11.
- Не показывать ошибки отдельным окном при сохранении

Открываем **Preferences -> Sublimelinter -> Settings** и вставляем в правую половину:

```json
{
    "user": {
        "debug": false,
        "delay": 0.25,
        "error_color": "D02000",
        "gutter_theme": "Packages/SublimeLinter/gutter-themes/Default/Default.gutter-theme",
        "gutter_theme_excludes": [],
        "lint_mode": "load/save",
        "linters": {
            "annotations": {
                "@disable": false,
                "args": [],
                "errors": [
                    "FIXME"
                ],
                "excludes": [],
                "warnings": [
                    "NOTE",
                    "README",
                    "TODO",
                    "@todo",
                    "XXX",
                    "WIP"
                ]
            },
            "clang": {
                "@disable": false,
                "args": [],
                "excludes": [],
                "extra_flags": "",
                "include_dirs": []
            },
            "cmakelint": {
                "@disable": false,
                "args": [],
                "excludes": []
            },
            "cppcheck": {
                "@disable": false,
                "args": [],
                "enable": [
                    "warning",
                    "style",
                    "performance",
                    "portability"
                ],
                "excludes": [],
                "std": [
                    "c99",
                    "c++11"
                ]
            },
        },
        "mark_style": "outline",
        "no_column_highlights_line": false,
        "passive_warnings": false,
        "paths": {
            "linux": [],
            "osx": [],
            "windows": ["C:/Program Files/Cppcheck"]
        },
        "python_paths": {
            "linux": [],
            "osx": [],
            "windows": []
        },
        "rc_search_limit": 3,
        "shell_timeout": 10,
        "show_errors_on_save": false,
        "show_marks_in_minimap": true,
        "tooltip_fontsize": "1rem",
        "tooltip_theme": "Packages/SublimeLinter/tooltip-themes/Default/Default.tooltip-theme",
        "tooltip_theme_excludes": [],
        "tooltips": false,
        "warning_color": "DDB700",
        "wrap_find": true
    }
}

```

#### SublimeAStyleFormatter
Форматирование настроено на мой вкус. Подробное описание всех пунктов есть в **Preferences -> SublimeAStyleFormatter -> Settings - Default**


Открываем **Preferences -> SublimeAStyleFormatter -> Settings - User** и вставляем:
```json
{
    "options_default": {
        "style": "google",
        "indent": "spaces",
        "indent-spaces": 4,
        "indent-col1-comments": true,
        "indent-switches": true,
        "indent-cases": true,
        "break-blocks": "default",
        "attach-namespaces": true,
        "break-blocks": "default",
        "add-brackets": true,
        "lineend": "linux"
    }
}
```

### Устанавливаем AC6 System Workbench for STM32 (SW4STM32)
Полностью процесс установки описан на сайте [OpenSTM32.org](http://www.openstm32.org/Installing%2BSystem%2BWorkbench%2Bfor%2BSTM32%2Bwith%2Binstaller). Регистрируемся и скачиваем [инсталятор](http://www.ac6-tools.com/downloads/SW4STM32/install_sw4stm32_win_64bits-latest.exe). В общем-то ставим по-умолчанию, заморочек нет.

#### Настройка SW4
Запускаем SW4, создаём рабочую область по пути **D:\workspace** 
![Скрин](/assets/img/stm32-windows-ide/sw4-setup-1.png)

При первом запуске будет установлен [ARM Toolchain](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm). После запуска кликаем на **Workbench** в правом верхнем углу.

Сначала установим удобную тему. Открываем Help->Install New Software, жмём **Add..** и вводим:
```
Eclipse Color Theme
http://eclipse-color-theme.github.io/update/
```
Жмём ОК, выбираем *Eclipse Color Theme* и жмём **Next >** и дальше всё по накатанной.
![Скрин](/assets/img/stm32-windows-ide/sw4-setup-2.png)

Включим тему: **Window -> Preferences ->Appearance -> Color Theme**. Я люблю Monokai, а вы можете позже подобрать, какую вам нравится.

Ну вот и пришло время запустить наш первый проект и проверить среду разработки в работе.

# Проверка настроенной среды и небольшие доводки
Повторюсь, я буду приводить примеры для отладочной платы [STM32F3DISCOVERY](http://www.st.com/en/evaluation-tools/stm32f3discovery.html). Делаю всё максимально просто, нам ведь просто надо проверить настройки среды разработки.

Запустим STM32CubeMX, выберем в главном окне New Project. Откроем вкладку Board Selector. Выбираем нашу плату:
- Type of board: Discovery
- MCU Series: STM32F3
- Из списка ниже выбираем STM32F3DISCOVERY

![Скрин](/assets/img/stm32-windows-ide/CubeMX-new-project-1.png)

#### Сделаем минимальную настройку МК
В левом меню включим:
- FreeRTOS: поставить галку Enabled
- SYS: Trace Asynchronous Sw
- Timebase Source: TIM17 (на текущем этапе можно выбрать любой)

![Скрин](/assets/img/stm32-linux-ide/CubeMX-Create-STM32F3Disco-2.png)

Переходим во вкладку Clock Configuration:
- В поле HCLK вводим 64МГц (на встроенном осциляторе максимальная частота)

#### Настроим сам проект
Открываем **Project->Settings** из верхнего меню.
- Указываем имя проекта в поле *Project Name*: **STM32Discovery-SW4-Test**
- Указываем путь в поле *Project Location*: **D:\workspace**
- Выбираем Toolchain: **SW4STM32**

![Скрин](/assets/img/stm32-windows-ide/CubeMX-new-project-2.png)

Откроем вкладку **Code Generator** и включим **"Add necessary libriary files as reference in the toolchain project configuration file"**

![Скрин](/assets/img/stm32-windows-ide/CubeMX-new-project-3.png)

Жмём **ОК** и теперь мы готовы создать проект. Жмите *Generate Source Code* в верхнем меню:

![](/assets/img/stm32-linux-ide/CubeMX-Compile-Button.png)

### Проверим Sublime Text 3

 - Добавить проект в ST3: **Project->Add folder to Project...** и
   выбираем папку нашего нового проекта.  
 - Создать новый файл: **Правый
   щелчок на имени проекта слева -> New File**

В открывшемся окне вставьте актуальные пути к библиотекам HAL:
```bash
-ID:\Libs\STM32Cube\Repository\STM32Cube_FW_F3_V1.9.0\Drivers\CMSIS\Device\STTM32F3xx\Include
-ID:\Libs\STM32Cube\Repository\STM32Cube_FW_F3_V1.9.0\Drivers\CMSIS\Include
-ID:\Libs\STM32Cube\Repository\STM32Cube_FW_F3_V1.9.0\Middlewares\Third_Party\FreeRTOS\Source\CMSIS_RTOS
-ID:\Libs\STM32Cube\Repository\STM32Cube_FW_F3_V1.9.0\Drivers\STM32F3xx_HAL_Driver\Inc
-ID:\Libs\STM32Cube\Repository\STM32Cube_FW_F3_V1.9.0\Middlewares\Third_Party\FreeRTOS\Source\include
```
Теперь надо сбросить кэш Cmake: **CTRL-SHIFT-P -> EasyClangComplete: Clean current cmake cache**

Откроем **Src/main.c**. Наведём курсор на какую-нибудь функцию и порадуемся всплывающим окошкам с её описанием. Подробнее о работе EasyClangComplete можно посмотреть [тут](https://github.com/niosus/EasyClangComplete).

Попробуем отформатировать код: нажмём CTRL+ALT+F и радуемся, как всё поменялось. Если предпочитаете другой стиль, нет проблем, настройки в **Preferences -> SublimeAStyleFormatter**.

### Откомпилируем проект и посмотрим debug
Запускаем SW4 и импортируем проект:
- **File -> Import... -> General -> Existing projects into workspace**
    - Select root directory: *D:\workspace\STM32Discovery-SW4-Test*
    - **НЕ** ставим галку Copy projects into workspace

![Скрин](/assets/img/stm32-windows-ide/sw4-project-import.png)
![Скрин](/assets/img/stm32-windows-ide/sw4-project-import-2.png)

Жмём Finish.

Выбираем проект в **Project explorer**. Компилируем **Project -> Build Project**. Если всё было сделано без ошибок ранее, проект скомпилируется за несколько секунд. И в консоли в нижней части будет примерно так:
```
Finished building target: STM32Discovery-SW4-Test.elf
 
make --no-print-directory post-build
Generating binary and Printing size information:
arm-none-eabi-objcopy -O binary "STM32Discovery-SW4-Test.elf" "STM32Discovery-SW4-Test.bin"
arm-none-eabi-size "STM32Discovery-SW4-Test.elf"
   text	   data	    bss	    dec	    hex	filename
   8724	     16	   5040	  13780	   35d4	STM32Discovery-SW4-Test.elf
 

16:12:35 Build Finished (took 18s.983ms)
```
> Кстати, под Linux тот же проект компилится за 6 секунд. Пишите, если знаете, почему.

Подключим нашу плату к компьютеру и попробуем загрузить прошивку: **Run -> Run as Ac6 STM32 C/C++ Application**. Если всё удачно, в конце вывода консоли будет:

```
wrote 10240 bytes from file Debug/STM32Discovery-SW4-Test.elf in 0.603749s (16.563 KiB/s)
** Programming Finished **
** Verify Started **
STM32F303VCTx.cpu: target state: halted
target halted due to breakpoint, current mode: Thread 
xPSR: 0x61000000 pc: 0x2000002e msp: 0x2000a000
verified 8748 bytes in 0.246880s (34.604 KiB/s)
** Verified OK **
** Resetting Target **
```
Ну и самое вкусное. Запустим дебагер. **Run -> Debug** (или просто жмите F11). SW4 загрузит прошивку и предложит открыть отдельную перспективу для дебагера. Советую согласиться.

Изначально наша программа будет остановлена на ```int main(void) {}```, это брейкпонт по-умолчанию. Запустим программу нажав F8, чтобы инициализировались все настройки МК, потом остановим, нажав кнопку *Suspend* в верхней панели. Давайте попробуем зажечь светодиоды. В правой верхней части перспективы откроем вкладку **I/O Registers**, развернём GPIO и правый щелчок на **GPIOE -> ODR -> Activate**

![](/assets/img/stm32-windows-ide/eclipse-debug-io-registers-1.png)

Теперь посмотрим в STM32CubeMX в нашем проекте, что светодиоды сидят на ногах PE8 - PE15:
![](/assets/img/stm32-linux-ide/STMF3DiscoveryLedPins.png)

В столбце HEX Value в строках c **GPIOE -> ODR -> ODR15** по **GPIOE -> ODR -> ODR8** выставим **1** и радуемся магии, как загораются светодиоды на плате. Ставим **0** - гаснут.

![](/assets/img/stm32-windows-ide/eclipse-debug-io-registers-2.png)

К сожалению, такая магия возможно только в остановленном состоянии, в отличие от того же Keil uVision, реалтайма тут нет. ~~(грустный смайлик)~~

Осталось только настроить Git.

## Установка  Git
Скачиваем [дистрибутив](https://git-scm.com/download/win) с сайта git-scm.com.
Устанавливаем со всеми опциями по-умолчанию, кроме одной: советую в качестве редактора включить Nano. Но если вам хочется, можете оставить Vim или же подключить Notepad++, если он установлен.

![Скрин](/assets/img/stm32-windows-ide/git-install-1.png)

### Настройка Git
Проводником заходим в папку нашего проекта, правый щелчок мышью **-> Git Bash Here**:

![Скрин](/assets/img/stm32-windows-ide/git-setup-1.png)

В появившемся окне инициализируем Git, включая ввод персональных данных:
```bash
git init
git config --global user.email "b**@bulki.me"
git config --global user.name "Nikita Bulaev"
```
> Не забудьте подставить свои данные!

Создадим .gitignore в корне проекта и добавим в него:
```
/Debug/

.metadata
bin/
tmp/
*.tmp
*.bak
*.swp
*~.nib
local.properties
.settings/
.loadpath
.recommenders

# External tool builders
.externalToolBuilders/

# Locally stored "Eclipse launch configurations"
*.launch

# CDT-specific (C/C++ Development Tooling)
.cproject

# sbteclipse plugin
.target

# Tern plugin
.tern-project

# TeXlipse plugin
.texlipse

# STS (Spring Tool Suite)
.springBeans

# Code Recommenders
.recommenders/

# Scala IDE specific (Scala & Java development for Eclipse)
.cache-main
.scala_dependencies
.worksheet
```
Ну и сделаем коммит:

```bash
git add *
git commit -a -m "Initial commit"
```


В общем-то дальнейшие вещи выходят за рамки статьи о настройке среды разработки. Это и полноценное описание дебагинга, и всякие горячие клавиши ST3. Об этом я бы хотел поговорить в другой раз. Надеюсь, данная статья будет полезна, хотя ныне настройка среды стала гораздо проще, чем ещё год назад. Удачи!

