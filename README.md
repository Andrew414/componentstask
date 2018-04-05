# Многокомпонентное приложение
### Лабораторная работа #3
[[презентация]](https://www.dropbox.com/s/4gxy1ekjm16n81o/Task%203.pptx?dl=0) [[репозиторий]](https://github.com/Andrew414/componentstask)

## Условие
Написать приложение, которое позволяет осуществлять выбор файла и его модификацию с помощью двух встроенных компонентов. 

Приложение может иметь консольный интерфейс, GUI или Web-интерфейс, оно может быть реализовано в виде бинарных (`.exe`/`.dll`/`.so`) файлов или в виде скриптов на любом языке программирования. Приложение должно явно проверять, что оно **не** запущено с правами администратора или root-правами и выходить в противном случае.

В случае, если есть возможность использовать первый компонент, приложение должно использовать его. Если у первого компонента не хватает прав, приложение должно использовать второй компонент.

Приложение должно корректно обрабатывать ситуацию, если один из компонентов отсутствует.

Подобная архитектура приложения, разделенного на компоненты, каждый из которых работает с теми минимальными правами, без которых оно в принципе не может работать, называется **принципом наименьших привилегий**. Такая архитектура позволяет обезопасить компьютер от атак, так как даже в случае их совершения ущерб не распространится дальше ограничений, наложенных на приложение.

#### Компоненты приложения
Первый компонент должен быть реализован в отдельном исполняемом файле (`.dll`, `.exe`, `.jar`, `.so`, `.py`, `.sh`, …) Он должен предоставлять *интерфейс*, по которому главное приложение может его вызвать. В качестве интерфейса могут выступать функции, экспортируемые из файла или интерфейс командной строки, иными словами, первый компонент может быть запущен в контексте главного приложения или в отдельном процессе. Под экспортируемым функциями в случае `.dll`, написанной на `C/C++`, подразумеваются функции, объявленные `__declspec(dllexport)` и загружаемые через `LoadLibrary` и `GetProcAddress`, а в случае скриптовых языков - описанные в отдельных файлах подпрограммы, которые могут быть вызваны из главного приложения с помощью различных `import`-директив.

Второй компонент также должен быть реализован в отдельном исполняемом файле (`.dll`, `.exe`, `.jar`, `.so`, `.py`, `.sh`, …). Этот компонент компонент должен **запускаться с правами администратора** и выполнять ту же самую работу, которую выполняет первый компонент. Запрос прав администратора может выполнятся как внутри самого второго компонента, так и главным приложением при запуске этого компонента.

Оба компонента могут лежать в одном файле, но не в том же, в котором лежит главное приложение. Например, файл `components.dll` может экспортировать две функции - `ModifyFile(...)` и `ModifyFileAsAdmin(...)`, либо файл `component.dll` может экспортировать одну функцию - `ModifyFile`, но главное приложение будет иметь возможность запустить эту функции обычным способом и с правами администратора.

#### Пример приложения
* GUI, позволяющий выбрать файл в файловой системе.
* Компоненты, принимающие имя файла и создающие *zip-архив* с этим файлом по тому же пути с таким же именем (но с расширением, измененным на zip).
  * В случае, если пользователь выбирает файл с рабочего стола, к которому есть доступ у пользователя, с правами которого работает приложение, должен вызываться **первый** компонент.
  * В случае, если пользователь выбирает файл из каталога C:\Program Files\, к которому есть права только у администраторов, должен запускаться **второй** компонент с правами администратора.

#### Примеры компонентов
* Архивация файла
* Удаление файла
* Изменение содержимого файла
  * Для текстовых файлов - шифрование через xor, для остальных - игнорирование
  * Для графических файлов - приведение к черно-белому виду или другой фильтр, для остальных - игнорирование
* Подсчет хеша файла и запись его рядом с файлом
* И др.

### Дополнительные требования
Программа должна быть написана с использованием общепринятого [**стандарта**](https://ru.wikipedia.org/wiki/Стандарт_оформления_кода). Если для выбранного языка есть один такой стандарт, используйте его, иначе выберите любой для своего языка программирования и укажите в комментариях к Pull Request, какой именно стандарт выбран.

### Дополнительные задания
Четыре задачи:
* Графический интерфейс для главного приложения
  * Графический интерфейс также должен быть написан с использованием принципа наименьших привилегий - не запускаться с правами администратора и т.д.
* Несколько разных пар компонентов
  * Примеры компонентов можно найти выше в условии (например, архивация, удаление и хеширование файлов)
  * Кажая пара компонентов должна быть независима от других
  * В каждой паре компонентов первый должен запускаться без прав администратора, а второй - с правами администратора
* Вызов первого компонента происходит без создания отдельного процесса
  * Должны использоваться экспортируемые из компонента функции
* Загрузка всех (пар) компонентов происходит динамически
  * Количество компонентов и пути к ним не должны быть прописаны в коде главного приложения
  * Приложение должно знать правило, по которому производится поиск компонентов
  * Например, поиск происходит в подпапке `\components\`, компонентами считаются все `.dll`-файлы, которые экспортируют три функции - `GetDesription`, `ProcessFile`, `ProcessFileWithAdminRights`
  * Приложение выполняет поиск всех файлов `.dll` в указанной подпапке, и проверяет, экспортируют ли они три указанные функции
  * Если да - добавляет компонент в общий список, после чего дает выбрать имя файла и операцию (компонент), которую можно с ним произвести

### Репозиторий
[Репозиторий](https://github.com/Andrew414/componentstask) содержит главную страницу [`README.md`](https://github.com/Andrew414/componentstask/blob/master/README.md) и одиннадцать персональных каталогов. Необходимо сделать копию (fork) репозитория и вносить все изменения только в свой персональный каталог. Все эти каталоги содержат пустой файл `.gitignore`, куда нужно вписать все временные файлы из лабораторной работы. Остальные файлы должны быть файлами кода (в т.ч. проекта).

### Сдача лабораторной работы
Можно выбрать любой язык программирования, IDE и фреймворки/библиотеки. Если для работы нужны какие-то нестандартные конфигурации или ПО, предоставьте инструкцию по настройке среды.

Сдача через **Pull Request** из вашего клонированного репозитория. Pull Request должен содержать код, файлы проекта и информационные файлы. Результаты сборок и временные файлы должны быть исключены из коммитов.

### Оценка
Максимум за лабораторную работу - **10 баллов**.
- **3 балла** за главную задачу
- **3 балла**, если работа сделана вовремя
- **1 балл** за вызов первого компонента *без создания процесса*
- **1 балл** за реализацию *нескольких* компонентов
- **1 балл** за реализацию *динамической загрузки* компонентов
- **1 балл** за реализацию *графического интерфейса*

### Сроки
На выполнение лабораторной дается 4 недели (15 марта - 12 апреля). Лабораторная работа считается принятой, когда Pull Request переходит в состояние "approved". Дни, в течение которых лабораторная была на проверке, не учитываются.
![ ](https://i.snag.gy/lPOzf7.jpg)

Например, если вы сдаете лабораторную через две недели после выдачи (29 марта), но комментарии даны только через неделю (5 апреля), у вас все еще будет две недели на устранение недочетов, все дни после коммита до получения ревью не считаются.

### Желаю удачи!
