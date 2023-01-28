# Torrent
* На трекере хранится список файлов и информация об активных пользователях, у которых есть те или иные файлы (возможно не целиком).
* С помощью клиентского приложения можно просматривать список файлов на трекере, а также добавлять новые и выбирать файлы из списка для скачивания.
* Файлы условно разбиваются на последовательные блоки бинарных данных константного размера (например 10M). Последний блок может иметь меньший размер. Блоки нумеруются с нуля.

---

# Torrent
* Клиент при подключении отправляет на трекер список раздаваемых им файлов.
* При скачивании файла клиент получает у трекера информацию о клиентах, раздающих файл (сидах), и далее общается с ними напрямую.
* У отдельного сида можно узнать о том, какие полные части у него есть, а также скачать их.
* После скачивания отдельных блоков некоторого файла клиент становится сидом.

---
# Интерфейс клиента
* `upload [FILENAME]` -- клиент может загружать файл
* `list` -- клиент может узнать раздаваемые файлы
* `download [ID]` -- клиент может загрузить понравившийся ему файл

---
# Torrent-tracker
* Хранит мета-информацию о раздаваемых файлах:
    * идентификатор
    * активные клиенты (недавно был update), у которых есть этот файл целиком или некоторые его части
* Порт сервера: 8081
* Запросы:
    * `list` — список раздаваемых файлов
    * `upload` — публикация нового файла
    * `sources` — список клиентов, владеющих определенным файлов целиком или некоторыми его частями
    * `update` — загрузка клиентом данных о раздаваемых файлах
* **Важно:** перечисленные выше запросы являются _рекомендованными_, как и их формат, описываемый ниже. Вы можете менять список и семантику запросов на своё усмотрение (в случае таких изменений вам требуется где-то задокументировать какие запросы есть и для чего используются (можно в комментариях в `.proto`)).
    * Реализовывать (де)сериализацию запросов вы можете как вручную, так и с использованием любых библиотек и фреймворков

    
---
# List
* Формат запроса:

* Формат ответа:
    * (FileContent: id, name, size)*,
    * id — идентификатор файла
    * name — название файла
    * size — размер файла

---
# Upload
* Формат запроса:
    * filename, size, userInfo
    * filename — название файла
    * size — размер файла
    * UserInfo: ip, port -- информация про клиента
* Формат ответа:
    * FileContent: id, size, name — информация про файл

# Примечание
* Если клиент А и клиент Б решили опубликовать файл abc.txt, то это будут **разные** файлы, иными словами каждый запрос на публикацию файла возвращает **новый** id

---

# Sources

* Формат запроса:
    * <3: Byte> <id: Int>,
    * id — идентификатор файла
* Формат ответа:
    * idFile (UserInfo)*,
    * UserInfo: ip, port -- информация про клиента
    * id — id файла
---

# Update

* Формат запроса:
    * userInfo, portOfClientServer, (fileContent)*,
    * userInfo — порт клиента,
    * portOfClientServer — порт клиент-сервера
    * fileContent — информация про файл
* Формат ответа:
    * <status: Boolean>,
    * status — True, если информация успешно обновлена

# Примечание
* Клиент обязан исполнять данный запрос каждые 5 минут, иначе сервер считает, что клиент ушел с раздачи
---
# Torrent-client
* Порт клиента указывается при запуске и передается на трекер в рамках запроса update
* Каждый файл раздается по частям, размер части — константа на всё приложение
* Клиент хранит и раздает эти самые части
* Запросы:
    * stat — доступные для раздачи части определенного файла
    * get — скачивание части определенного файла

---

# Stat

* Формат запроса:
    * <1: Byte> <id: Int>,
    * id — идентификатор файла
* Формат ответа:
    * userInfo idFile (<part: Int>)*,
    * userInfo -- информация про клиента
    * idFile — id файла
    * part — номер части

# Примечание

* Часть считается доступной для раздачи, если она хранится на клиенте целиком
---
# Get
* Формат запроса:
    * <2: Byte> <id: Int> <part: Int>
    * id — идентификатор файла,
    * part — номер части
* Формат ответа:
    * <content: Bytes>,
    * content — содержимое части
    * fileContent -- информация про какой файл
    * partOfFile -- какая часть файла
---

# Требования:
* Gradle проект
* Консольные трекер и клиент, позволяющие исполнять указанные запросы
* Тесты
* Документация процесса сборки артефактов вашего приложения
    * В идеале, хотелось бы, чтобы этими артефактами были два shell-скрипта
      (один для запуска клиента, другой - для сервера)
    * Однако двух executable jar-файлов будет тоже достаточно
* Клиент должен сохранять информацию о раздаваемых файлах между перезапусками
* Трекер должен сохранять список раздаваемых файлов между перезапусками

# Запуск
* tracker.sh -- запуск трекера
* client.sh -- запуск клиента, внутри скрипта менять номер порта для многопользовательской версии

