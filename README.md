# Разработка информационной системы «Реестры» для оплаты карт закрытых вызовов скорой помощи Челябинской области

Скорой помощи в Челябинской области необходима система для передачи карт закрытых вызовов в ТФОМС для последующей оплаты.


 - СМП - Скорая медицинская помощь
 - ТФОМС - Территориальный фонд обязательного медицинского страхования
 - ПК "АДИС" - Программный комплекс автоматизации станций скорой медицинской помощи

* Пользователи: десятки операторов СМП
* Требования:
  * Интеграция с системой ПК "АДИС",
  * Доступ к просмотру закрытых карт вызовов,
  * Дать возможность редактировать карты вызова,
  * Показывать ошибки валидации в картах,
  * Дать возможность фильтровать карты по определенным критериям,
  * Возможность идентифицировать пациента,
  * Формировать реестр,
  * Возможность загружать ответ от фонда в систему.
* Дополнительный контекст:
  * В конце месяца начинается тестовая приёмка реестров, фонд валидирует пакет реестров, возвращает записи с ошибками, если таковые есть,
  * В начале каждого месяца сдается итоговый реестр,
  * Пакет реестров - zip архив со сводными XMl документами с информацией о картах закрытых вызовов.
