# zlib Example

Author: Yuriy Astrov  

Это одна из популярных библиотек, которая часто используется для работы с ZIP архивами, GZIP (.gz) архивами и сжатия интернет трафика. Впрочем, часто для работы с ZIP архивами используют библиотеки-обёртки (т.е. библиотеки, которые сами используют zlib).

Официальная страница: [http://zlib.net/](http://zlib.net/).  
Документацию и примеры так же можно найти:

*  [Usage Example](http://www.zlib.net/zlib_how.html)  
*  [Руководство по библиотеке zlib 1.1.4](http://zlib.net.ru/)  

## Сборка 

### В OS Windows, компилятор MinGW

Исходники библиотеки находятся в директории C:\zlib-1.2.11.  
В примере используется компилятор MinGW из комплекта IDE Code::Blocks, находящийся в директории C:\codeblocks-16.01mingw-nosetup\MinGW\bin.  

Для сборки под OS Windows удобно использовать BAT файл:

    setlocal
    set PATH=C:\codeblocks-16.01mingw-nosetup\MinGW\bin;%PATH%
    cd C:\zlib-1.2.11
    mingw32-make -f win32/Makefile.gcc
    endlocal
 
Он временно (на сеанс) прописывает путь к компилятору в переменную PATH, заходит в папку с библиотекой и вызывает mingw32-make.

Конечно, для запуска, в папку с программой нужно скопировать zlib1.dll.

### Установка в OS Linux

В ОС Linux для установки луше воспользоваться менеджером пакетов, например apt-get для Debian/Ubuntu дистрибутивов.

### Пример сжатия строки

Неудобство доставляет то, что zlib создаёт новые типы данных (на самом деле псевдонимы), причины выбора которых не всегда очевидны.

Для сжатия (архивирования) строки src и записи результата в buffer:

    #define RESULT_MAX_LENGTH 1000

    const char *src = "Hello world";                        /* Исходная строка */
    const uLong srcLen = (uLong)(strlen(src)*sizeof(char)); /* Размер строки в байтах */
    char   buffer[RESULT_MAX_LENGTH];                       /* Результат сжатия */
    uLongf resultLen = RESULT_MAX_LENGTH*sizeof(char);      /* Размер результа */

    /* Архивируем (сжимаем) строку */
    compress((Bytef*)buffer, &resultLen, (const Bytef*)src, srcLen);

По документации buffer должен быть больше исходной строки на 0.1% и ещё 12 байт.

Чтобы разархивировать buffer и результат записать в output:

    char output[RESULT_MAX_LENGTH];  /* результат разархивирования*/
    uLongf resultLen2 = RESULT_MAX_LENGTH*sizeof(char);
    uncompress((Bytef*)output, &resultLen2, (const Bytef*)buffer, resultLen);

Полный пример:

    /* Zlib compress string example.
     * 
     * Compile (zlib версии 1.2.11 находится в C:/zlib-1.2.11):
     * gcc -std=c99 -LC:/zlib-1.2.11 -IC:/zlib-1.2.11 -lzlib1 main.c -o test
     *  */
    #include <stdio.h>
    #include <stdlib.h>
    #include <strings.h>
    #include "zlib.h"
     
    /* Будет использоваться размер буфера данных с запасом,
     * 1000 должно хватить. Всё-таки это всего лишь
     * пример - можно немного и упростить. */
    #define RESULT_MAX_LENGTH 1000
     
    int main(int argc, char **argv)
    {
        const char *src = "Hello world"; /* Исходная строка */
        char buffer[RESULT_MAX_LENGTH];  /* Результат архивирования (сжатия) */
        /* Будем сжимать строку src и результат записывать в buffer,
         * подготавливаем размер буфера результата и очищаем buffer.
         * По документации buffer должен быть
         * больше исходной строки на 0.1% и ещё 12 байт, но поскольку у нас
         * пример, берём с запасом.*/
        uLongf resultLen = RESULT_MAX_LENGTH*sizeof(char);
        memset(buffer, 0, RESULT_MAX_LENGTH*sizeof(char));
        const uLong srcLen = (uLong)(strlen(src)*sizeof(char));
     
        printf("String length: %d", strlen(src));
        printf("String length in bytes: %d", strlen(src)*sizeof(char));
        printf("String:\n%s\n", src);
        printf("Compress start\n");
        /* Архивируем (сжимаем) строку */
        const int result = compress((Bytef*)buffer, &resultLen, (const Bytef*)src, srcLen);
        /* Проверяем результат вызова */
        switch(result) {
            case Z_OK:
                printf("Compressed Ok!\n");
                printf("Result length: %d\n", resultLen);
                printf("Compressed string:\n%s\n", buffer);
                break;
            case Z_MEM_ERROR:
                printf("Need more memory!\n");
                return -1;
                break;
            case Z_BUF_ERROR:
                printf("Need more bytes in destination buffer!\n");
                return -1;
                break;
            default: break;
        }
        printf("Uncompress start\n");
        char output[RESULT_MAX_LENGTH];  /* результат разархивирования*/
        /* Будем разархивировать buffer и результат записывать в output. */   
        uLongf resultLen2 = RESULT_MAX_LENGTH*sizeof(char);
        memset(output, 0, RESULT_MAX_LENGTH*sizeof(char));
        const int result2 = uncompress((Bytef*)output, &resultLen2,
                                        (const Bytef*)buffer, resultLen);
        switch(result2) {
            case Z_OK:
                printf("Uncompressed Ok!\n");
                printf("Result length: %d\n", resultLen2);
                printf("Uncompressed string:\n%s\n", output);
                break;
            case Z_MEM_ERROR:
                printf("Need more memory!\n");
                return -1;
                break;
            case Z_BUF_ERROR:
                printf("Need more bytes in destination buffer!\n");
                return -1;
                break;
            case Z_DATA_ERROR:
                printf("Invalid data!\n");
                return -1;
                break;
            default: break;
        }
        return 0;
    }

Cборка с библиотекой версии 1.2.11 в OS Windows:

    gcc -std=c99 -LC:/zlib-1.2.11 -IC:/zlib-1.2.11 -lzlib1 main.c -o test

В этом примере размер сжатой строки оказывается больше, чем у оригинальной, но это всего лишь показывает, что конкретная строка плохо поддаётся сжатию. Строка, в которой много одинаковых букв (и соответственно байтов) сжималась бы лучше.

## Cжатие больших объёмов данных

Для сжатия больших объёмов данных (например файлов), без загрузки их в память компьютера, нужно использовать структуру z_stream, описанную в файле zlib.h (в OS Linux и Unix: /usr/include/zlib.h).
