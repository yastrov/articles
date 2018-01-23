# Пример работы с QuaZIP

Author: Yuriy Astrov  

QuaZIP - это библиотека для Qt Framework, поддерживающая работу с ZIP архивами. Для работы требуется библиотека zlib.

*  QuaZIP [https://sourceforge.net/projects/quazip/](https://sourceforge.net/projects/quazip/).  
*  Документация [http://quazip.sourceforge.net/](http://quazip.sourceforge.net/).  
*  Официальная страничка zlib: [http://zlib.net/](http://zlib.net/).  

## Сборка

### Сборка под OS Windows с использованием MinGW (из комплекта Qt Framework)

Пусть Qt Framework установлен в директорию C:\Qt (в примере используется версия 5.9), библиотека zlib в C:\zlib-1.2.11, QuaZIP в C:\quazip-0.7.3.  

Для сборки библиотек удобно использовать BAT скрипты.  

BAT файл make_zlib.bat:

    setlocal
    set PATH=C:\Qt\Tools\mingw530_32\bin;%PATH%
    cd C:\zlib-1.2.11
    mingw32-make -f win32/Makefile.gcc
    endlocal

BAT файл для сборки QuaZIP make_quazip.bat:

    setlocal
    set PATH=C:\Qt\Tools\mingw530_32\bin;C:\Qt\5.9\mingw53_32\bin;%PATH%
    cd C:\quazip-0.7.3
    qmake "CONFIG+=release" "INCLUDEPATH+=C:\zlib-1.2.11" "LIBS+=-LC:\zlib-1.2.11 -lz"
    mingw32-make
    qmake "CONFIG+=debug" "INCLUDEPATH+=C:\zlib-1.2.11" "LIBS+=-LC:\zlib-1.2.11 -lz"
    mingw32-make
    endlocal

Конечно, для запуска, в папку с программой нужно скопировать zlib1.dll и quazip.dll.

### Сборка под OS Unix/Linux

О сборке QuaZIP можно прочитать на странице [http://quazip.sourceforge.net/](http://quazip.sourceforge.net/).

## Подключение к проекту Qt Framework

Фрагмент Qt файла проекта .pro:

    win32-g++: {
        INCLUDEPATH += C:/zlib-1.2.11
        INCLUDEPATH += C:/quazip-0.7.3
        LIBS += -LC:/zlib-1.2.11 -lz
        LIBS += -LC:/quazip-0.7.3/quazip/release -lquazip
    }

    win32-msvc: {
        LIBS += C:/zlib-1.2.11/zlib.lib
        LIBS += C:/quazip-0.7.3/quazip/release/quazip.lib
    }

    unix: {
        CONFIG += link_pkgconfig
        PKGCONFIG += zlib

        LIBS += -L/usr/lib -lquazip5
        INCLUDEPATH += /usr/include/quazip5
    }

## Пример открытия архива

Пример открытия архива:

    QuaZip qzip(filename);
    if(qzip.open(QuaZip::mdUnzip)) {
        ;
        qzip.close();
    }

## Работа с каждым файлом, находящимся в архиве

Перечислить все файлы в архиве и их свойства можно с помощью цикла:

    for(bool more=qzip.goToFirstFile(); more;
                    more=qzip.goToNextFile()) {
        ;
    }

Пример получения свойств и открытие каждого файла в архиве:

    QuaZipFile file(&qzip);
    QuaZipFileInfo64 zipInfo;
    for(bool more=qzip.goToFirstFile(); more;
                    more=qzip.goToNextFile()) {
        if (!qzip.getCurrentFileInfo(&zipInfo)) {
            /* Не удалось получить информацию о файле в архиве. */
            break;
        }
        if(file.open(QIODevice::ReadOnly)) {
            ; /* Здесь можно работать с file, как с обычным QIODevice. */
        }
        file.close();
    }

Полный пример:

    /* ... */

    #ifdef Q_OS_WIN
        #include <quazip/quazip.h>
        #include <quazip/quazipfile.h>
        #include <quazip/quazipfileinfo.h>
    #else
        #include <quazip.h>
        #include <quazipfile.h>
        #include <quazipfileinfo.h>
    #endif
    #ifdef MYPREFIX_DEBUG
        #include <QDebug>
    #endif

    #DEFINE bufferSize 4096

    void Foo(const QString &filename)
    {
        char buffer[bufferSize];
        QFileInfo fInfo(filename);
        if(fInfo.isFile() && fInfo.isReadable()) {
            QuaZip qzip(filename);
            if(qzip.open(QuaZip::mdUnzip)) {
                QuaZipFile file(&qzip);
                QuaZipFileInfo64 zipInfo;

                for(bool more=qzip.goToFirstFile(); more;
                                more=qzip.goToNextFile()) {
                    if(qzip.getZipError() != UNZ_OK) {
                        #ifdef MYPREFIX_DEBUG
                        qDebug() << "ZIP error!";
                        #endif
                        break;
                    }
                    if (!qzip.getCurrentFileInfo(&zipInfo)) {
                        #ifdef MYPREFIX_DEBUG
                        qDebug() << "ZIP error!";
                        #endif
                        break;
                    }
                    if(file.open(QIODevice::ReadOnly)) {
                        const QString fname1 = file.getActualFileName();
                        const QString fname2 = zipInfo.name;
                        /* Действия с file, как с обычным QIODevice.
                           Например:
                         */
                        while(!file.atEnd()) {
                            const qint64 readLen = file.read(buffer, bufferSize);
                            if (readLen <= 0)
                                break;
                            QByteArray data(buffer, readLen);
                        }
                        file.close();
                    } else {
                        if(file.getZipError() != UNZ_OK) {
                            #ifdef MYPREFIX_DEBUG
                            qDebug() << "ZIP error in file: " << zipInfo.name;
                            #endif
                            break;
                        }
                    }
                }
                qzip.close();
            }
            if(qzip.getZipError() != UNZ_OK) {
                #ifdef MYPREFIX_DEBUG
                qDebug() << "ZIP error!";
                #endif
            }
        }
    }

## Проверка ZIP архива (корректности CRC32 для каждого файла)

Для каждого файла, находящегося в архиве, записана контрольная сумма CRC32. С её помощью можно проверить, что распакованный файл не содержит ошибок.

Самый важный фрагмент (подсчёт суммы для файла и сравнение с находящимся в архиве значением):

    QuaCrc32 crc32Obj;
    QuaZipFileInfo64 zipInfo;
    qint64 readLen = 0;
    qzip.getCurrentFileInfo(&zipInfo)

    if(file.open(QIODevice::ReadOnly)) {
        while(!file.atEnd()) {
            readLen = file.read(buffer, bufferSize);
            if (readLen <= 0)
                break;
            crc32Obj.update(QByteArray(buffer, readLen));
        }
        file.close();
        if(crc32Obj.value() != zipInfo.crc) {
            /* Файл в архиве повреждён. */
        }
    }

Фрагмент ZipChecker.h:

    /* ... */
    #ifdef Q_OS_WIN
        #include <quazip/quazip.h>
        #include <quazip/quazipfile.h>
        #include <quazip/quazipfileinfo.h>
        #include <quazip/quacrc32.h>
    #else
        #include <quazip.h>
        #include <quazipfile.h>
        #include <quazipfileinfo.h>
        #include <quacrc32.h>
    #endif
    #ifdef MYPREFIX_DEBUG
        #include <QDebug>
    #endif

    class ZipChecker {
        /* ... */
    private:
        const static qint64 bufferSize=4096;
        char buffer[bufferSize];
    };

Фрагмент ZipChecker.cpp:

    bool ZipChecker::isCorrectFile(const QString &fileName)
    {
        if(fileName.isEmpty() || fileName.isNull())
            return false;
        QFileInfo fInfo(fileName);
        if(fInfo.isFile() && fInfo.isReadable())
        {
            QuaZip qzip(fileName);
            bool isDamagedZip = false;
            if(qzip.open(QuaZip::mdUnzip))
            {
                QuaZipFile file(&qzip);
                QuaZipFileInfo64 zipInfo;
                qint64 readLen = 0;
                QuaCrc32 crc32Obj;

                for(bool more=qzip.goToFirstFile(); more;
                    more=qzip.goToNextFile()) {
                    crc32Obj.reset();
                    if(qzip.getZipError() != UNZ_OK) {
                        isDamagedZip = true;
                        break;
                    }
                    if (!qzip.getCurrentFileInfo(&zipInfo)) {
                        isDamagedZip = true;
                        break;
                    }
                    /* QuaZipFile наследован от QIODevice. */
                    if(file.open(QIODevice::ReadOnly)) {
    #ifdef MYPREFIX_DEBUG
                        qDebug() << "innerFile: " << file.getActualFileName();
    #endif
                        while(!file.atEnd()) {
                            readLen = file.read(buffer, bufferSize);
                            if (readLen <= 0)
                                break;
                            crc32Obj.update(QByteArray(buffer, readLen));
                        }
                        file.close();
                        if(crc32Obj.value() != zipInfo.crc) {
                            isDamagedZip = true;
                            break;
                        }
                    } else {
                        if(qzip.getZipError() != UNZ_OK) {
                            isDamagedZip = true;
                            break;
                        }
                    }
                }
            }
            if(qzip.isOpen())
                qzip.close();
            if(qzip.getZipError() != UNZ_OK) {
                isDamagedZip = true;
            }

            if(isDamagedZip) {
                return false;
            }
        }
        return true;
    }


