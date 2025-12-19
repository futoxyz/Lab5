# Лабораторная работа №5

### Поиск ошибок в [Лабораторной работе №2](https://github.com/futoxyz/Lab2)

# Ошибка 1 — завершение программы при создании архива

**Место:**

execute.py, метод execute: case "zip"/"tar"

**Симптом:**

Программа завершается с ошибкой при создании архива с недопустимыми символами на Windows.

**Как воспроизвести:**

При запуске на Windows: создать архив любого существующего каталога, включив в название один из следующих символов:
/, \, :, *, ?, ", <, >, |

**Отладка:**

Установлен breakpoint на строке *shutil.make_archive(line.fname, "zip", line.archive_dir)*. В отладчике видно, что переменная line.fname содержит недопустимые для файловой системы символы

При пошаговом выполнении видно, что исключение возникает, когда система пытается создать файл с таким именем.

**Причина:**

Отстуствие проверки названия файла для создания.

**Исправление:**

Добавлено:
```
for sym in ["/", "\\", ":", "*", "?", '"', "<", ">", "|"]:
    if sym in line.fname:
        data.log(BADNAME)
        return
```
**Проверка:**

При нахождении недопустимого символа программа возвращает ошибку пользователю.

**Доказательства:**
- [error1_debugger.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error1_debugger.png)
- [error1_after.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error1_after.png)


# Ошибка 2 — проблема чтения при использованни grep

**Место:** 

grep.py, метод grep

**Симптом:**

Программа завершается с ошибкой при использовании grep в некоторой директории.

**Как воспроизвести:**

Ввести команду grep с любыми условиями для каталога, где есть файл с символами, не входящие в стандартную кодировку Python.


**Отладка:**

Установлен breakpoint на цикле for.
Программа не может получить строки.

**Причина:**

Python не может прочитать файл.


**Исправление:**

Явное указание кодировки и исключение try except:
```
try:
    with open(file_dir, encoding='utf-8') as f:
    ...

except:
    return f"Could not search in {file_dir}\n"
```

**Проверка:**

Команда не выдаёт ошибку при любом содержании проверяемого файла.

**Доказательства:**
- [error2_debugger.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error2_debugger.png)
- [error2_after.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error2_after.png)

# Ошибка 3 — Ошибка отката копирования каталога

**Место:**

data.py, метод undo() класса Data.

**Симптом:**

При использовании undo после копирования каталога копия не всегда успешно удаляется на Windows.

**Как воспроизвести:**

На Windows: скопировать любой каталог в любой путь, затем прописать undo.

**Отладка:**

Установлен breakpoint на строке с удалением каталога. Ошибку возвращает операционная система.

**Причина:**

Windows (в зависимости от характеристик) ограничивает удаление каталога через код.

**Исправление:**

Переместить копию в .trash/
```
shutil.move(self.sc_dir, os.path.join(self.init_dir, ".trash"))
```
- sc_dir - путь к копии
- os.path.join(self.init_dir, ".trash") - путь к .trash

**Проверка:**

Копия не удаляется полностью, а идет в корзину. Исходная директория, где лежала копия каталога, освобождается.

**Доказательства:**
- [error3_debugger.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error3_debugger.png)
- [error3_after.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error3_after.png)

# Ошибка 4 — Копирование и удаление пути вместо простого перемещения при вызове rm
**Место:**

execute.py, метод execute()

**Симптом:**

Аналогично ошибке 3: код может вернуть ошибку удаления от Windows.

**Как воспроизвести:**

На Windows: удалить любой каталог.

**Отладка:**

breakpoint установлен на shutil.rmtree(), при выполнении Windows возвращает PermissionError.

**Причина:**

Вместо простого перемещения выполняется копирование и удаление, что не меняет результат и решает проблему.

**Исправление:**

Заменить на:
```
shutil.move(line.rm_dir, os.path.join(data.init_dir, ".trash"))
```

**Проверка:**

ОС не вызывает ошибки, логика кода осталась неизменной.

**Доказательства:**
- [error4_debugger.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error4_debugger.png)
- [error4_after.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error4_after.png)

# Ошибка 5 — Проблема чтения с помощью cat.
**Место:**

execute.py, метод execute(), case cat.

**Симптом:**

Программа завершается с ошибкой при использовании cat с файлом, содержащим определенные файлы.

**Как воспроизвести:**

Использовать cat на нечитаемом файле (такие, например, можно найти в папках .venv или .idea).

**Отладка:**

breakpoint стоит на выводе строк. Программа не может их получить.

**Причина:**

Python не может прочитать файл.

**Исправление:**

Указание кодировки и исключение try except:
```
with open(line.file_dir, encoding='utf-8') as f:
    try:
        ...
    except:
        data.log(READFAIL)

```

**Проверка:**

Программа дает ошибку вывода и не завершается.

**Доказательства:**
- [error5_debugger.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error5_debugger.png)
- [error5_after.png](https://github.com/futoxyz/Lab5/blob/main/screenshots/error5_after.png)