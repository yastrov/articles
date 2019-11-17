TITLE: Установка wxWidgets-3.1.3 в ОС Windows и подключение к IDE Code::Blocks
AUTHOR: Yuriy Astrov
TAGS: wxWidgets;Windows;IDE Code::Blocks
DATE: 17.11.2019

# Установка wxWidgets-3.1.3 в ОС Windows и подключение к IDE Code::Blocks  

## Предисловие  

Есть много советов как скомпилировать wxWidgets из исходников, но, к сожалению, очень мало сказано о том, как подключить к IDE Code::Blocks готовую сборку.

Этот текст написать для всех, в т.ч. тех, кто впервые настраивает Code::Blocks и подключает к нему компилятор.  

Перед установкой полезно прочитать README [https://github.com/wxWidgets/wxWidgets/releases/tag/v3.1.3](https://github.com/wxWidgets/wxWidgets/releases/tag/v3.1.3), в котором сказано о том, какие файлы wxWidgets потребуются, какие компиляторы поддерживаются.  

Внимание!: если в имени директории или пути к ней, куда установлены компилятор или wxWidgets будут пробелы, возможны ошибки!  

## Программы  
IDE Code::Blocks [http://www.codeblocks.org/downloads/binaries](http://www.codeblocks.org/downloads/binaries)  
MinGW x64 [https://sourceforge.net/projects/mingw-w64/](https://sourceforge.net/projects/mingw-w64/)  
Взял `mingw-i686-8.1.0-release-win32-sjlj` (Обязательно с обработкой ошибок SJLJ, так сказано в README к релизу wxWidgest!). Все доступные сборки перечислены на странице [https://sourceforge.net/projects/mingw-w64/files/](https://sourceforge.net/projects/mingw-w64/files/)  
wxWidgets-3.1.3 [https://github.com/wxWidgets/wxWidgets/releases](https://github.com/wxWidgets/wxWidgets/releases)  
wxFormBuilder [https://github.com/wxFormBuilder/wxFormBuilder/releases](https://github.com/wxFormBuilder/wxFormBuilder/releases)  

## Дополнительные материалы
WxSmith tutorials [http://wiki.codeblocks.org/index.php/WxSmith_tutorials](http://wiki.codeblocks.org/index.php/WxSmith_tutorials)  

## Установка и настройка компилятора MinGW  
Компилятор распаковал в директорию `C:\PORTABLE_INSTALLED\` и переименовал из mingw в `mingw-i686-8.1.0-release-win32-sjlj-rt_v6`  
`C:\PORTABLE_INSTALLED\mingw-i686-8.1.0-release-win32-sjlj-rt_v6`  
(обязательно без пробела, иначе возможны ошибки!)  
Можно выбрать 64-х битную версию компилятора.

В IDE Code::Blocks в настройках (верхнее меню) `"Settings" -> "Compiler..."`  
Убедился, что `"Current compiler"` выбран GNU GCC и скопировал его настройки, назвав `mingw-i686-8.1.0-release-win32-sjlj-rt_v6`.  
Убедился, что теперь выбран новый компилятор.
На вкладке `"Compiler settings"` ниже установил стандарт языка С++11 (вкладка `"Compiler flag" -> "General"`, вторая строчка с `-std=c++11`) (согласно рекомендации README wxWidgets)  
На вкладке `"Search directories"`, появившейся вкладке `"Compiler"`, добавил:  
`C:\ProgramFiles\wxWidgets-3.1.3\include`  
В `C:\ProgramFiles\wxWidgets-3.1.3` будет позже wxWidgets.  

На вкладке `"Toolchain executables"`:  
Установил текущую директорию компилятора: `C:\PORTABLE_INSTALLED\mingw-i686-8.1.0-release-win32-sjlj-rt_v6`  
И на вкладке ниже, `"Program Files"`, добавил префикс `i686-w64-` к первым четырем утилитам:  
*  "C compiler":              `mingw32-gcc.exe` -> `i686-w64-mingw32-gcc.exe`  
*  "C++ compiler":            `mingw32-g++.exe` -> `i686-w64-mingw32-g++.exe`  
*  "Linker for dynamic libs": `mingw32-g++.exe` -> `i686-w64-mingw32-g++.exe`  
*  "Linker for static libs":  `ar.exe` -> `i686-w64-mingw32-gcc-ar.exe`  

В соответствии с именами в директории компилятора: `C:\PORTABLE_INSTALLED\mingw-i686-8.1.0-release-win32-sjlj-rt_v6\bin`  

## Установка и настройка wxWidgets-3.1.3  
Поставил пакет `wxMSW-3.1.3-Setup.exe` в `C:\ProgramFiles\` (обязательно без пробела, иначе возможны ошибки!)  

Скачал и распаковал файлы для MinGW 8.1.0:  
`wxMSW-3.1.3_gcc810_Dev.7z`  
`wxMSW-3.1.3_gcc810_ReleaseDLL.7z`  
Получились директории:  
`C:\ProgramFiles\wxWidgets-3.1.3\include`  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\gcc810_dll`  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\gcc810_dll\mswu`  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\gcc810_dll\mswud`  

Если вы выбрали 64-х битный компилятор, то нужно выбирать и 64-х битные DLL:
`wxMSW-3.1.3_gcc810_x64_Dev.7z`
`wxMSW-3.1.3_gcc810_x64_ReleaseDLL.7z`

## Создание первого проекта wxWidgets

Выбрал в меню `"Project" -> "New"`, и в открытом окне выбрать `wxWidgets project`.  
В "Мастере создания проекта":  
Выбрал `wxWidgets-3.1.x`  
Выбрал название проекта и его размещение  
На одном из следующих шагов будет предложено выбрать редактор графического интерфейса GUI. Можно оставить None и создавать GUI в программе, можно выбрать конкретный редактор.  
И тип окна приложения: на основе диалога ("Dialog Based") или фрейма ("Frame Based"). Я выбрал фрейм.  

Примечание: Если выбрать wxSmith, то можно будет редактировать GUI с помощью встроенного плагина Code::Blocks. Если выбрать wxFormBuilder, то его потребуется установить.

Затем будет предложено выбрать размещение wxWidgets. Я указал: `C:\ProgramFiles\wxWidgets-3.1.3`.  

Если это первый проект wxWidgets, будет предложено установить глобавльные переменные.  
Я указал только "base": `C:\ProgramFiles\wxWidgets-3.1.3`  
Указание "include", "lib" могут потребоваться, если Вы скачивали wxWidgets отдельными архивами, а не пакетом установки.  
Позднее глобальные переменные wx можно отредактирвать в пункте меню `"Settings" -> "Global Variables"`.  

На следующем шаге нужно убедиться, что выбран компилятор для wxWidgets, в моем случае `mingw-i686-8.1.0-release-win32-sjlj`.  

При выборе конфигурации нужно выбрать:  
`"Use wxWidgets DLL"`  
`"Enable Unicode"`  
Остальные настройки на этой странице оставил выключенными.  

Возможно будет предупреждение об ошибке, однако моя программа была собрана.

На следующем шаге будет предложено выбрать модули wxWidgets для работы с текстом, XML, и другие. Можно пропустить.  

Теперь нужно настроить созданный проект.  

### В настройках проекта

В меню `"Project" -> "Build options..."` теперь нужно настроить проект для сборки.  
В левом окошке выделил имя проекта (так же есть настройки для "Debug" и "Release").  
По аналогии с тем, как делал раньше, включил стандарт С++11, необходимый для wxWidgets-3.1.3.   
Во вкладке `"Search Directories" -> "Compiler"` добавил:  
`C:\ProgramFiles\wxWidgets-3.1.3\include`  

Теперь выбрал `"Debug"`:  
Во вкладке `"Search Directories" -> "Compiler"` добавил:  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\mswud`  
Поскольку пользуюсь Unicode версией и Debug библиотеками.  
Во вкладке `"Search Directories" -> "Linker"` добавил:  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\gcc810_dll\mswud`  

В `"Release"`:  
Здесь почти то же самое, но без буквы "d" в названии директории:  
Во вкладке `"Search Directories" -> "Compiler"` добавил:  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\mswu`  
Поскольку пользуюсь Unicode версией и Debug библиотеками.  
Во вкладке `"Search Directories" -> "Linker"` добавил:  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\gcc810_dll\mswu`  

Теперь, если все сделано правильно, программа будет собрана.  

## Если при сборке появилась ошибка "wx/setup.h: No such file or directory"  

Это значит, что вероятно компилятор не может найти путь к:  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\mswud` для Debug  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\mswu` для Release  

Ошибка упомянута в документации IDE Code::Blocks [http://wiki.codeblocks.org/index.php/WxWindowsQuickRef](http://wiki.codeblocks.org/index.php/WxWindowsQuickRef)  

## Если при сборке появилась ошибка "cannot find -lwxmsw31u"

Это значит, что компоновщик (Linker) не может найти DLL, т.е. возможно пропущены или указаны некорректно:  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\gcc810_dll\mswud` для Debug  
`C:\ProgramFiles\wxWidgets-3.1.3\lib\gcc810_dll\mswu` для Release  

Ошибка упомянута в документации IDE Code::Blocks [http://wiki.codeblocks.org/index.php/WxWindowsQuickRef](http://wiki.codeblocks.org/index.php/WxWindowsQuickRef)  

## Добавить информацию о авторе и описание программы  

Информацию о авторе и описание программы можно добавить в файле ресурсов `"wx.rc"`, который будет создан вместе с проектом.  

Описание в документации IDE Code::Blocks: [http://wiki.codeblocks.org/index.php/FAQ-Compiling_(general)#Q:_Microsoft_calls_MSVCRT.DLL_a_.22Known_DLL..22_How_do_I_know_if_I_can.2Fshould_use_it.3F](http://wiki.codeblocks.org/index.php/FAQ-Compiling_(general)#Q:_Microsoft_calls_MSVCRT.DLL_a_.22Known_DLL..22_How_do_I_know_if_I_can.2Fshould_use_it.3F)  

Документация MSDN:
[https://docs.microsoft.com/en-us/windows/win32/menurc/versioninfo-resource](https://docs.microsoft.com/en-us/windows/win32/menurc/versioninfo-resource)  
[https://docs.microsoft.com/en-us/windows/win32/menurc/varfileinfo-block](https://docs.microsoft.com/en-us/windows/win32/menurc/varfileinfo-block)  

Я добавил описание перед последним `#endif`.  

Cтоит обратить внимание на числа, указанные в `VALUE "Translation"` и описании BLOCK. Они описывают язык, на котором написана информация, и кодировку файла ресурсов.

В Windows IDE Code::Blocks по умолчанию использует кодировку Windows-1251, поэтому для описания программы на русском будет
    BLOCK "041904E3"  
    VALUE "Translation",  0x419, 1251  
Поскольку 1251 = 0x04E3. Подробнее можно почитать в MSDN.

Возможно указание двух блоков подряд для разных языков, тогда в `VALUE "Translation"` после запятой добавляется вторая пара чисел.
