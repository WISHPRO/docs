git 53d6a6b5b4c47d274b714a13a03d9a796239b835

---

# Введение

- [C чего начать](#where-to-start)
- [Философия Laravel](#laravel-philosophy)

<a name="where-to-start"></a>
## С чего начать

Изучать новый незнакомый фреймворк - нелегкое занятие, но учить новое всегда интересно. Вот некоторые рекомендации, что начать читать в первую очередь:

- [Установка](/docs/master/installation) и [Настройка](/docs/master/configuration)
- [Routing](/docs/master/routing)
- [Requests & Input](/docs/master/requests)
- [Responses](/docs/master/responses)
- [Views](/docs/master/views)
- [Controllers](/docs/master/controllers)

После изучения этих документов вы будете иметь представление о том, как во фреймворке происходит обработка цикла запроса.

Затем можно почитать про [настройку соединения](/docs/database) и [построение запросов](/docs/master/queries) к базе данных, а так же про встроенный [Eloquent ORM](/docs/master/eloquent), облегчающий работу с БД. А для того, чтобы понять, как реализовать функционал регистрации и логина пользователей - главу [Аутентификация](/docs/master/authentication)

<a name="laravel-philosophy"></a>
## Философия Laravel

Laravel - фреймворк для построения веб-приложений с выразительным и элегантным синтаксисом. Мы считаем, что процесс разработки только тогда наиболее продуктивен, когда работа с фреймворком приносит радость и удовольствие. Счастливые разработчики пишут лучший код. Laravel - попытка сгладить все острые и неприятные моменты в работе php-разработчика. Он берет на себя ((аутентификацию)), ((роутинг)), работу с сессиями, кеширование и многое другое, что встречается в большинстве приложений, оставив вам только фокус на вашей задаче.

Laravel стремится сделать процесс разработки приятным для разработчика без ущерба для функциональности приложений. Для этого мы попытались объединить все самое лучшее из того, что мы видели в других фреймворках - Ruby On Rails, ASP.NET MVC и Синатра.

Laravel подходит для построения больших и надежных приложений. Превосходный ((IoC container)), встроенные миграции и интегрированная поддержка ((юнит-тестов)) дают вам мощные инструменты для того, чтобы сделать именно тот функционал, который вам нужен.