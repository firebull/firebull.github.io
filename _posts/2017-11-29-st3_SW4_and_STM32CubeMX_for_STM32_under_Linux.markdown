---
layout: post
title:  "Настройка Sublime Text 3, SW4 и STM32CubeMX для разработки STM32 под Linux"
date:   2017-11-29 17:32:10 +0300
categories: stm32 sublime-text-3 SW4STM32
---


Подобных статей достаточно много на просторах интернета, но хотелось бы написать актуальную вариацию. Лично я долгое время мучался в связке: Ubuntu - основная система, разработка под STM32 в виртуальной машине Windows 7. Но однажды меня это очень утомило и я таки решил потратить несколько дней на поиск решения и вылизывание полноценной среды под Linux Ubuntu. Забегу вперёд и скажу, что идеала я так и не добился, не удалось сделать realtime debug, как в Keil. В остальном всё очень пристойно.

> Аналогичная инструкция по настройке среды в MS Windows 10 лежит [тут](/stm32/sublime-text-3/sw4stm32/2017/11/30/st3_SW4_and_STM32CubeMX_for_STM32_under_windows.html).

<!--more-->

> Эту статью я сначала опубликовал на [Хабре](https://habrahabr.ru/post/343456/)

Что в итоге у меня получилось:
-  Sublime Text 3 с автодополнениями и подсветкой всех функций, включая HAL и остальные библиотеки проекта;
- AC6 System Workbench for STM32 с кастомизированными перспективами и прочими плюшками;
- Конечно же использую контроль версий в Git.

Шаги, которые необходимо проделать:
- Добавить необходимые PPA в APT;
- Установить нужные библиотеки;
- Установить программы;
- Внести правки в конфигурационные файлы;
- Доделать всякие плюшки в виде красивых иконок, поиска из меню и т.д.

Инструкция дана для абсолютно чистой машины на базе Ubuntu 16.04, в вашем случае могут быть уже установлены какие-то компоненты. В этой статье и в последующих буду приводить примеры для отладочной платы [STM32F3DISCOVERY](http://www.st.com/en/evaluation-tools/stm32f3discovery.html). Она основана на МК STM32F303VCT6 c 256-Кбайт Flash и 48-КБайт RAM в корпусе LQFP100. Вы можете адаптировать настройки под ваш МК очень легко благодаря STM32CubeMX.

> Всё указано для пользователя с именем **bulkin**, не забудьте менять его при настройке.

## Предварительно устанавливаем необходимый софт
```bash
sudo apt install git clang cmake python-pip
sudo pip install --upgrade pip
```
#### Создадим нужные директории для установки программ и хранения библиотек
Я предпочитаю устанавливать программы не из PPA в отдельную папку в домашней директории **~/Programs**. Во-первых, устанавливаю с правами локального пользователя. Во-вторых, папка с именем на английском, т.к. некоторые программы не любят кириллицу в пути.
Библиотеки предпочитаю хранить в **/opt/libs**.
<spoiler title="Почему в /opt?">
Да, */opt* - это OPTIONAL APPLICATIONS. Но у меня в */opt* монтируется отдельный раздел с btrfs со сжатием. На всех своих машинах я настроил пути к своим библиотекам в */opt/libs*. Морочиться и делать все по канонам (использовать */usr/local/lib* и */usr/local/share*) не вижу смысла, легко запутаться.
</spoiler>

```bash
mkdir ~/Programs
sudo mkdir /opt/libs
sudo chown bulkin:bulkin /opt/libs
```

#### Для STM32CubeMX и Eclipse нам понадобятся:
- Т.к. STM32CubeMX 32-битное приложение, необходимы необходимы соответсвующие библиотеки, если у вас 64-битная ОС.
-  Java Run Time Environment (будем использовать OpenJRE 8).

### Устанавливаем 32-битные библиотеки
```bash
sudo apt install lib32ncurses5
```

### Устанавливаем Open JDK 8 JRE
```bash
sudo apt install default-jre
```

### Устанавливаем STM32CubeMX
Действуем согласно [инструкции самого ST](http://www.st.com/content/ccc/resource/technical/document/user_manual/10/c5/1a/43/3a/70/43/7d/DM00104712.pdf/files/DM00104712.pdf/jcr:content/translations/en.DM00104712.pdf).

Скачиваем [дистрибутив](http://www.st.com/en/development-tools/stm32cubemx.html#getsoftware-scroll) с сайта ST (потребуется регистрация), разархивируем и запустим файл с расширением *.linux*. Если установщик не запускается, скорее всего не установлены 32-битные библиотеки. Путь установки меняем на **/home/bulkin/Programs/STM32CubeMX**, устанавливаем.

![](/assets/img/stm32-linux-ide/CubeMX-install-1.png)

#### Создадим красивый ярлык и добавим поиск в Dash
```bash
touch ~/.local/share/applications/CubeMX.desktop
chmod ug+x ~/.local/share/applications/CubeMX.desktop
nano ~/.local/share/applications/CubeMX.desktop
```
Вставим следующее содержимое:
```
[Desktop Entry]
Version=1.0
Name=CubeMX
Comment=STM32 CubeMX
Exec=/home/bulkin/Programs/STM32CubeMX/STM32CubeMX
Icon=/home/bulkin/Programs/STM32CubeMX/help/STM32CubeMX.ico
Terminal=false
Type=Application
Categories=Utility;Application;
```
Теперь в dash иконка появится в разделе Приложения, поиск в dash будет работать после перелогинивания.

Запускаем STM32CubeMX, идём в настройки **Help->Updater Settings** и меняем путь для хранения библиотек на */opt/libs/STM32Cube/Repository/*

![](/assets/img/stm32-linux-ide/CubeMX-install-2.png)

Установим библиотеку для STM32F3. Открываем **Help->Install New Libriaries**, ставим галку  *Firmware Package for Family STM32F3*, жмём **Install Now**

![](/assets/img/stm32-linux-ide/CubeMX-install-3.png)

## Устанавливаем Sublime Text 3
Вообще, SW4 вполне самодостаточная IDE. Но я люблю кодить именно в ST3, а компиляция и дебаг в SW4. Инструкция по установке ST3 для любых дистрибутивов лежит [тут](https://www.sublimetext.com/docs/3/linux_repositories.html).

```bash
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
sudo apt install apt-transport-https
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
sudo apt update
sudo apt install sublime-text cppcheck python-configparser
sudo pip install --upgrade cmake
sudo pip install cmakelint
sudo pip install cubemx2cmake
```
**После этого стоит перелогиниться.**

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
        "-I/usr/include",
        "-I/usr/lib/clang/$clang_version/include",
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
            "windows": []
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
Полностью процесс установки описан на сайте [OpenSTM32.org](http://www.openstm32.org/Installing%2BSystem%2BWorkbench%2Bfor%2BSTM32%2Bwith%2Binstaller).

Регистрируемся на [www.openstm32.org](http://www.openstm32.org) и скачиваем [инсталятор](http://www.ac6-tools.com/downloads/SW4STM32/install_sw4stm32_linux_64bits-latest.run).

Переходим в папку загрузки и вводим
```bash
bash install_sw4stm32_linux_64bits-latest.run
```
Если у вас установлен **gksudo** и вы дал, установщик запустится в графическом режиме. У меня не установлен, потому всё в терминале.

Несколько раз вводим **1** в качестве согласия со всякой ерундой. Указываем путь установки:
```
/home/bulkin/Programs/SystemWorkbench
```
Снова соглашаемся со всем, как девственник в свой первый раз, и ждём окончания установки.

#### Добавим поиск SW4 в Dash
```bash
chmod +x "/home/bulkin/Рабочий стол/sw4stm32_shortcut.desktop"
cp "/home/bulkin/Рабочий стол/sw4stm32_shortcut.desktop" ~/.local/share/applications/sw4stm32_shortcut.desktop
```
#### Настройка SW4
Запускаем SW4, соглашаемся с созданием рабочей области, кликаем на *workbench*. При первом запуске будет установлен ARM Toolchain.

Сначала установим удобную тему. Открываем Help->Install New Software, жмём **Add..** и вводим:
```
Eclipse Color Theme
http://eclipse-color-theme.github.io/update/
```
Жмём ОК, выбираем *Eclipse Color Theme* и жмём **Next >** и дальше всё по накатанной.

![](/assets/img/stm32-linux-ide/Eclipse-install-1.png)

Включим тему: **Window -> Preferences ->Appearance -> Color Theme**. Я люблю Monokai, а вы можете позже подобрать, какую вам нравится.

Ну вот и пришло время запустить наш первый проект и проверить среду разработки в работе.

# Проверка настроенной среды и небольшие доводки
Повторюсь, я буду приводить примеры для отладочной платы [STM32F3DISCOVERY](http://www.st.com/en/evaluation-tools/stm32f3discovery.html). Делаю всё максимально просто, нам ведь просто надо проверить настройки среды разработки.

Запустим STM32CubeMX, выберем в главном окне New Project. Откроем вкладку Board Selector. Выбираем нашу плату:
- Type of board: Discovery
- MCU Series: STM32F3
- Из списка ниже выбираем STM32F3DISCOVERY

![](/assets/img/stm32-linux-ide/CubeMX-Create-STM32F3Disco-1.png)

И два щелчка на нашей плате.

#### Сделаем минимальную настройку МК
В левом меню включим:
- FreeRTOS: поставить галку Enabled
- SYS: Trace Asynchronous Sw
- Timebase Source: TIM17 (на текущем этапе можно выбрать любой)

![](/assets/img/stm32-linux-ide/CubeMX-Create-STM32F3Disco-2.png)

Переходим во вкладку Clock Configuration:
- В поле HCLK вводим 64МГц (на встроенном осциляторе максимальная частота)

#### Настроим сам проект
Открываем **Project->Settings** из верхнего меню.
- Указываем имя проекта в поле *Project Name*: STM32Discovery-SW4-Test
- Указываем путь в поле *Project Location*: /home/bulkin/workspace
- Выбираем Toolchain: SW4STM32

![](/assets/img/stm32-linux-ide/CubeMX-Create-STM32F3Disco-3.png)

Откроем вкладку **Code Generator** и включим "Add necessary libriary files as reference in the toolchain project configuration file"

![](/assets/img/stm32-linux-ide/CubeMX-Create-STM32F3Disco-4.png)

Жмём **ОК** и теперь мы готовы создать проект. Жмите Generate Source Code в верхнем меню:
![](/assets/img/stm32-linux-ide/CubeMX-Compile-Button.png)

#### Проверим Sublime Text 3
Для начала надо создать CMakeLists.txt. Для этого открываем консоль в корне нашего проекта и вводим:
```bash
cubemx2cmake
```
Из нашего **STM32Discovery-SW4-Test.ioc** будут созданы необходимые для компиляции из командной строки файлы. Но нас интересует только **CMakeLists.txt.template**. Переименуем его в **CMakeLists.txt**.

![](/assets/img/stm32-linux-ide/ST3-cubemx2cmake.png)

**Project->Add folder to Project...** и выбираем папку нашего нового проекта.
Для начала надо добавить в *CMakeLists.txt* недостающие пути к библиотекам. Это нужно для корректной работы *EasyClangComplete*. Слева щёлкаем на *CMakeLists.txt* и вносим изменения:
Над строкой ```set(USER_INCLUDE Inc)``` добавляем:
```
set(STM32CUBEREPO /opt/libs/STM32Cube/Repository/STM32Cube_FW_F3_V1.9.0)
```
Все последующие **set** вплоть до **file** меняем на:
```
set(USER_INCLUDE Inc)
set(CMSIS_DEVICE_INCLUDE ${STM32CUBEREPO}/Drivers/CMSIS/Device/ST/STM32F3xx/Include)
set(CMSIS_INCLUDE ${STM32CUBEREPO}/Drivers/CMSIS/Include)
set(CMSIS_RTOS_INCLUDE ${STM32CUBEREPO}/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS)
set(HAL_INCLUDE ${STM32CUBEREPO}/Drivers/STM32F3xx_HAL_Driver/Inc)
set(FREERTOS_INCLUDE ${STM32CUBEREPO}/Middlewares/Third_Party/FreeRTOS/Source/include)
```
Ну и в раздел **include_directories** в самый конец добавить ``` ${FREERTOS_INCLUDE} ${CMSIS_DEVICE_INCLUDE}```. В итоге:
```
include_directories(${USER_INCLUDE} ${CMSIS_DEVICE_INCLUDE} ${CMSIS_INCLUDE} ${HAL_INCLUDE} ${CMSIS_RTOS_INCLUDE} ${FREERTOS_INCLUDE})
```

```cmake
set(MCU_FAMILY STM32F3xx)

cmake_minimum_required(VERSION 3.6)

project(STM32Discovery-SW4-Test C ASM)

add_definitions(-DSTM32F303xC)
add_definitions(-DUSE_HAL_LIBRARY)

set(STM32CUBEREPO /opt/libs/STM32Cube/Repository/STM32Cube_FW_F3_V1.9.0)

set(USER_INCLUDE Inc)
set(CMSIS_DEVICE_INCLUDE ${STM32CUBEREPO}/Drivers/CMSIS/Device/ST/STM32F3xx/Include)
set(CMSIS_INCLUDE ${STM32CUBEREPO}/Drivers/CMSIS/Include)
set(CMSIS_RTOS_INCLUDE ${STM32CUBEREPO}/Middlewares/Third_Party/FreeRTOS/Source/CMSIS_RTOS)
set(HAL_INCLUDE ${STM32CUBEREPO}/Drivers/STM32F3xx_HAL_Driver/Inc)
set(FREERTOS_INCLUDE ${STM32CUBEREPO}/Middlewares/Third_Party/FreeRTOS/Source/include)
set(RVDS_INCLUDE ${STM32CUBEREPO}/Middlewares/Third_Party/FreeRTOS/Source/portable/RVDS/ARM_CM4F)


file(GLOB_RECURSE USER_INCLUDE_F ${USER_INCLUDE}/*.h)
file(GLOB_RECURSE CMSIS_DEVICE_INCLUDE_F ${CMSIS_DEVICE_INCLUDE}/*.h)
file(GLOB_RECURSE CMSIS_INCLUDE_F ${CMSIS_INCLUDE}/*.h)
file(GLOB_RECURSE HAL_INCLUDE_F ${HAL_INCLUDE}/*.h)

file(GLOB_RECURSE USER_SOURCES Src/*.c)
file(GLOB_RECURSE HAL_SOURCES ${STM32CUBEREPO}/Drivers/STM32F3xx_HAL_DRIVER/Src/*.c)
file(GLOB_RECURSE CMSIS_SYSTEM ${STM32CUBEREPO}/Drivers/CMSIS/Device/ST/STM32F3xx/Source/Templates/system_STM32F3xx.c)
file(GLOB_RECURSE CMSIS_STARTUP ${STM32CUBEREPO}/Drivers/CMSIS/Device/ST/STM32F3xx/Source/Templates/gcc/startup_STM32F303xC.s)

set(SOURCE_FILES ${USER_SOURCES} ${HAL_SOURCES} ${CMSIS_SYSTEM} ${CMSIS_STARTUP} STM32F303VCTx_FLASH.ld
                 ${USER_INCLUDE_F} ${CMSIS_DEVICE_INCLUDE_F} ${CMSIS_INCLUDE_F} ${HAL_INCLUDE_F})

include_directories(${USER_INCLUDE} ${CMSIS_DEVICE_INCLUDE} ${CMSIS_INCLUDE} ${HAL_INCLUDE} ${CMSIS_RTOS_INCLUDE} ${FREERTOS_INCLUDE} ${RVDS_INCLUDE})

add_executable(${PROJECT_NAME}.elf ${SOURCE_FILES})

set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-Map=${PROJECT_SOURCE_DIR}/build/${PROJECT_NAME}.map")
set(HEX_FILE ${PROJECT_SOURCE_DIR}/build/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_SOURCE_DIR}/build/${PROJECT_NAME}.bin)

add_custom_command(TARGET ${PROJECT_NAME}.elf PRE_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory ${PROJECT_SOURCE_DIR}/build
        COMMENT "Creating build directory")
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
        COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
        COMMENT "Building ${HEX_FILE}
Building ${BIN_FILE}")

add_custom_target(flash
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
        COMMAND openocd -f openocd_flash.cfg
        COMMENT "Flashing the target processor..."
        DEPENDS ${PROJECT_NAME}.elf)
add_custom_command(TARGET flash POST_BUILD COMMENT "Flashing finished!")
```

> Стоит объяснить, зачем нужны эти танцы с бубном. Дело в том, что при создании проекта мы выбрали опцию "добавлять библиотеки в качестве ссылок в тулчейне". А вот **cubemx2cmake** указывает относительный путь к библиотекам *HAL*, а также не добавляет пару путей к библиотекам *CMSIS* и *FreeRTOS*. Вероятно, это будет исправлено в будущих версиях, но пока так.

Теперь надо сбросить кэш Cmake: **CTRL-SHIFT-P -> EasyClangComplete: Clean current cmake cache**

Откроем Src/main.c Наведём курсор на какую-нибудь функцию и порадуемся всплывающим окошкам с её описанием. Подробнее о работе EasyClangComplete можно посмотреть [тут](https://github.com/niosus/EasyClangComplete).

Попробуем отформатировать код: нажмём CTRL+ALT+F и радуемся, как всё поменялось. Если предпочитаете другой стиль, нет проблем, настройки в **Preferences -> SublimeAStyleFormatter**.

### Откомпилируем проект и посмотрим debug
Запускаем SW4 и импортируем проект:
- File -> Import... -> General -> Existing projects into workspace
    - Select root directory: /home/bulkin/workspace/STM32Discovery-SW4-Test
    - НЕ ставим галку Copy projects into workspace

![](/assets/img/stm32-linux-ide/eclipse-import-project.png)

Жмём Finish.

Выбираем проект в **Project explorer**. Компилируем **Project -> Build Project**. Если всё было сделано без ошибок ранее, проект скомпилируется за несколько секунд. И в консоли в нижней части будет примерно так:
```
Generating binary and Printing size information:
arm-none-eabi-objcopy -O binary "STM32Discovery-SW4-Test.elf" "STM32Discovery-SW4-Test.bin"
arm-none-eabi-size "STM32Discovery-SW4-Test.elf"
   text	   data	    bss	    dec	    hex	filename
   8724	     16	   5040	  13780	   35d4	STM32Discovery-SW4-Test.elf


16:42:41 Build Finished (took 6s.173ms)
```
Подключим нашу плату к компьютеру и попробуем загрузить прошивку: **Run -> Run as Ac6 STM32 C/C++ Application**. Если всё удачно, в конце вывода консоли будет:

```
wrote 10240 bytes from file Debug/STM32Discovery-SW4-Test.elf in 0.513113s (19.489 KiB/s)
** Programming Finished **
** Verify Started **
STM32F303VCTx.cpu: target state: halted
target halted due to breakpoint, current mode: Thread
xPSR: 0x61000000 pc: 0x2000002e msp: 0x2000a000
verified 8748 bytes in 0.104648s (81.635 KiB/s)
** Verified OK **
** Resetting Target **
```
Ну и самое вкусное. Запустим дебагер. **Run -> Debug** (или просто жмите F11). SW4 загрузит прошивку и предложит открыть отдельную перспективу для дебагера. Советую согласиться.

Изначально наша программа будет остановлена на ```int main(void) {}```, это брейкпонт по-умолчанию. Запустим программу нажав F8, чтобы инициализировались все настройки МК, потом остановим, нажав кнопку *Suspend* в верхней панели. Давайте попробуем зажечь светодиоды. В правой верхней части перспективы откроем вкладку **I/O Registers**, развернём GPIO и правый щелчок на **GPIOE -> ODR -> Activate**

![](/assets/img/stm32-linux-ide/eclipse-debug-io-registers-1.png)

Теперь посмотрим в STM32CubeMX в нашем проекте, что светодиоды сидят на ногах PE8 - PE15:
![](/assets/img/stm32-linux-ide/STMF3DiscoveryLedPins.png)

В столбце HEX Value в строках c **GPIOE -> ODR -> ODR15** по **GPIOE -> ODR -> ODR8** выставим **1** и радуемся магии, как загораются светодиоды на плате. Ставим **0** - гаснут.

![](/assets/img/stm32-linux-ide/eclipse-debug-io-registers-2.png)

К сожалению, такая магия возможно только в остановленном состоянии, в отличие от того же Keil uVision, реалтайма тут нет. ~~(грустный смайлик)~~

Осталось только настроить Git.

## Настройка Git
Заходим через терминал в директорию с нашим проектом и инициализируем Git, включая ввод персональных данных:
```bash
cd ~/workspace/STM32Discovery-SW4-Test
git init
git config --global user.email "b**@bulki.me"
git config --global user.name "Nikita Bulaev"
```
Создадим .gitignore и добавим в него:
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
