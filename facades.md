git 4c3d4d4675101a3fa18154d1c85cf292a2292b46

---

# Фасады

- [Введение](#introduction)
- [Описание](#explanation)
- [Практическое использование](#practical-usage)
- [Создание фасадов](#creating-facades)
- [Фасады для тестирования](#mocking-facades)
- [Facade Class Reference](#facade-class-reference)

<a name="introduction"></a>
## Введение

Фасады предоставляют «статический» интерфейс (`Foo::bar()`) к классам, доступным в [IoC-контейнере](/docs/5.0/container). Laravel поставляется
со множеством фасадов и вы, вероятно, использовали их, даже не подозревая об этом.

Иногда вам может понадобиться создать собственные фасады для вашего приложения и пакетов (packages), поэтому давайте изучим идею, разработку
и использование этих классов.

> **Примечание:** перед погружением в фасады настоятельно рекомендуется как можно детальнее изучить [IoC-контейнер Laravel](/docs/5.0/container).

<a name="explanation"></a>
## Описание

В контексте приложения на Laravel, фасад - это класс, который предоставляет доступ к объекту в контейнере. Весь этот механизм реализован
в классе `Facade`. Фасады как Laravel, так и ваши собственные, наследуют этот базовый класс.

Ваш фасад должен определить единственный метод: `getFacadeAccessor`. Его задача - определить, что вы хотите получить из контейнера.
Класс `Facade` использует магический метод PHP `__callStatic()` для перенаправления вызовов методов с вашего фасаде на полученный объект.

Например, когда вы вызываете `Cache::get()`, Laravel получает объект `CacheManager` из IoC-контейнера и вызывает метод `get` этого класса.
Другими словами, фасады Laravel предоставляют удобный синтаксис для использования IoC-контейнера в качестве сервис-локатора (service locator).

<a name="practical-usage"></a>
## Практическое использование

В примере ниже делается обращение к механизму кэширования Laravel. На первый взгляд может показаться, что метод `get` принадлежит классу  `Cache`.

	$value = Cache::get('key');

Однако, если вы посмотрите в исходный код класса `Illuminate\Support\Facades\Cache`, то увидите, что он не содержит метода `get`:

	class Cache extends Facade {

		/**
		 * Получить зарегистрированное имя компонента.
		 *
		 * @return string
		 */
		protected static function getFacadeAccessor() { return 'cache'; }

	}

Класс Cache наследует класс `Facade` и определяет метод `getFacadeAccessor()`. Как вы помните, его задача - вернуть строковое имя (ключ)
привязки объекта в контейнере IoC.

Когда вы обращаетесь к любому статическому методу фасада `Cache`, Laravel получает объект `cache` из IoC и вызывает на нём требуемый метод
(в этом случае - `get`).

Таким образом, вызов `Cache::get` может быть записан так:

	$value = $app->make('cache')->get('key');

#### Испорт фасадов

Помните, если вы используете фасады в контроллерах с указанным пространством имён, вам нужно импортировать классы фасадов.
Все фасады находятся в глобальном простанстве имён.

	<?php namespace App\Http\Controllers;

	use Cache;

	class PhotosController extends Controller {

		/**
		 * Get all of the application photos.
		 *
		 * @return Response
		 */
		public function index()
		{
			$photos = Cache::get('photos');

			//
		}

	}
	
<a name="creating-facades"></a>
## Создание фасадов

Создать фасад довольно просто. Вам нужны только три вещи:

1. Биндинг (привязка) в IoC.
2. Класс-фасад.
3. Настройка для псевдонима фасада.

Посмотрим на следующий пример. Здесь определён класс `PaymentGateway\Payment`.

	namespace PaymentGateway;

	class Payment {

		public function process()
		{
			//
		}

	}

Нам нужно, чтобы этот класс извлекался из контейнера IoC, так что давайте добавим для него привязку (binding):

	App::bind('payment', function()
	{
		return new \PaymentGateway\Payment;
	});

Самое лучшее место для регистрации этой связки - новый [сервис-провайдер](/docs/5.0/container#service-providers) который мы назовём
`PaymentServiceProvider` и в котором мы создадим метод `register`, содержащий код выше. После этого вы можете настроить Laravel для загрузки
этого провайдера в файле `config/app.php`.

Дальше мы можем написать класс нашего фасада:

	use Illuminate\Support\Facades\Facade;

	class Payment extends Facade {

		protected static function getFacadeAccessor() { return 'payment'; }

	}

Наконец, по желанию можно добавить псевдоним (alias) для этого фасада в массив `aliases` файла настроек `config/app.php`- тогда мы сможем
вызывать метод `process` на классе `Payment`.

	Payment::process();

### Об автозагрузке псевдонимов

В некоторых случаях классы в массиве `aliases` не доступны из-за того, что [PHP не загружает неизвестные классы в подсказках типов](https://bugs.php.net/bug.php?id=39003).
Если `\ServiceWrapper\ApiTimeoutException` имеет псевдоним `ApiTimeoutException`, то блок `catch(ApiTimeoutException $e)`, помещённый в любое
пространство имён, кроме `\ServiceWrapper`, никогда не «поймает» это исключение, даже если оно было возбуждено внутри него.
Аналогичная проблема возникает в классах, которые содержат подсказки типов на неизвестные (неопределённые) классы.
Единственное решение - не использовать псевдонимы и вместо них в начале каждого файла писать `use` для ваших классов.

<a name="mocking-facades"></a>
## Фасады для тестирования

Юнит-тесты играют важную роль в том, почему фасады делают именно то, то они делают. На самом деле возможность тестирования - основная причина,
по которой фасады вообще существуют. Эта тема подробнее раскрыта в соответствующем разделе документации - [фасады-заглушки](/docs/testing#mocking-facades).

<a name="facade-class-reference"></a>
## Facade Class Reference

Ниже представлена таблица соответствий фасадов Laravel и классов, лежащих в их основе. 

Фасад  |  Класс  |  Имя в IoC 
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/5.0/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Console\Application](http://laravel.com/api/5.0/Illuminate/Console/Application.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/5.0/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel.com/api/5.0/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/5.0/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/5.0/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/5.0/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/5.0/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/5.0/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/5.0/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/5.0/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/5.0/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/5.0/Illuminate/Filesystem/Filesystem.html)  |  `files`
Form  |  [Illuminate\Html\FormBuilder](http://laravel.com/api/5.0/Illuminate/Html/FormBuilder.html)  |  `form`
Hash  |  [Illuminate\Hashing\HasherInterface](http://laravel.com/api/5.0/Illuminate/Hashing/HasherInterface.html)  |  `hash`
HTML  |  [Illuminate\Html\HtmlBuilder](http://laravel.com/api/5.0/Illuminate/Html/HtmlBuilder.html)  |  `html`
Input  |  [Illuminate\Http\Request](http://laravel.com/api/5.0/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/5.0/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/5.0/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/5.0/Illuminate/Mail/Mailer.html)  |  `mailer`
Paginator  |  [Illuminate\Pagination\Factory](http://laravel.com/api/5.0/Illuminate/Pagination/Factory.html)  |  `paginator`
Paginator (Instance)  |  [Illuminate\Pagination\Paginator](http://laravel.com/api/5.0/Illuminate/Pagination/Paginator.html)  |
Password  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laravel.com/api/5.0/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.reminder`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/5.0/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel.com/api/5.0/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/5.0/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/5.0/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/5.0/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/5.0/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Support\Facades\Response](http://laravel.com/api/5.0/Illuminate/Support/Facades/Response.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/5.0/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/5.0/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/5.0/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/5.0/Illuminate/Session/Store.html)  |
SSH  |  [Illuminate\Remote\RemoteManager](http://laravel.com/api/5.0/Illuminate/Remote/RemoteManager.html)  |  `remote`
SSH (Instance)  |  [Illuminate\Remote\Connection](http://laravel.com/api/5.0/Illuminate/Remote/Connection.html)  |
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/5.0/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/5.0/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/5.0/Illuminate/Validation/Validator.html)
View  |  [Illuminate\View\Factory](http://laravel.com/api/5.0/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/5.0/Illuminate/View/View.html)  |
