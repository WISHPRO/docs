git 11619ec101e66d1feabbc923fda58ce8b5daf763

---

# Контракты

- [Введение](#introduction)
- [Зачем нужны контракты?](#why-contracts)
- [Таблица основных контрактов](#contract-reference)
- [Использование контрактов](#how-to-use-contracts)

<a name="introduction"></a>
## Введение

Контракты в Laravel - это набор классов-интерфейсов, определяющий некий функционал ядра фреймворка. Например, контракт `Queue` определяет
методы работы с очередями, `Mailer` - методы для отправки мейлов.

Каждый контракт имеет свою реализацию (implementation) во фреймворке. Например, есть реализация `Queue` с различными драйверами
очередей и реализация `Mailer` с использованием [SwiftMailer](http://swiftmailer.org/).

Для удобства, контракты находятся в отдельном репозитории на гитхабе - https://github.com/illuminate/contracts. 

<a name="why-contracts"></a>
## Зачем нужны контракты?

Вы можете спросить - а для чего нужны контракты? Зачем вообще нужны интерфейсы? 

Ответ - слабая связность и упрощение кода.

### Слабая связность

Сначала давайте рассмотрим пример кода с сильной связностью.

	<?php namespace App\Orders;

	class Repository {

	    /**
	     * The cache.
	     */
	    protected $cache;

	    /**
	     * Create a new repository instance.
	     *
	     * @param  \Package\Cache\Memcached  $cache
	     * @return void
	     */
	    public function __construct(\SomePackage\Cache\Memcached $cache)
	    {
	        $this->cache = $cache;
	    }

	    /**
	     * Retrieve an Order by ID.
	     *
	     * @param  int  $id
	     * @return Order
	     */
	    public function find($id)
	    {
	        if ($this->cache->has($id))
	        {
	            //
	        }
	    }

	}

Код этого класса тесно связан с реализацией кэширования `\SomePackage\Cache\Memcached`. Мы зависим и от способа кэширования (`memcached`),
и от API данной библиотеки. Если мы хотим сменить кэширование с `memcached` на `redis`, нам придется вносить изменения в код класса `Repository`.

Чтобы такого не происходило, класс `Repository` не должен задумываться, кто именно предоставляет данные и как именно осуществляется запись.
Давайте изменим наш класс, чтобы отвязаться от конкретной реализации и сделать его более универсальным. Для этого добавим зависимость
от интерфейса кэширования.

	<?php namespace App\Orders;

	use Illuminate\Contracts\Cache\Repository as Cache;

	class Repository {

	    /**
	     * Create a new repository instance.
	     *
	     * @param  Cache  $cache
	     * @return void
	     */
	    public function __construct(Cache $cache)
	    {
	        $this->cache = $cache;
	    }

	}

Этот код не связан ни с одной внешней библиотекой, в том числе с ядром фреймворка! Контракт не содержит никакой конкретной реализации
кэширования, только интерфейс, и вы можете написать любую свою реализацию кэширования - используя внешние библиотеки или нет. Кроме того,
теперь вы можете в любой момент легко изменить способ кэширования, просто подав другую реализацию в конструктор класса при регистрации
его в [сервис-контейнере](/docs/master/container) в методе `register()` вашего [сервис-провайдера](/docs/master/provider).

### Упрощение кода

Когда все сервисы ядра фреймворка аккуратно определены в простых интерфейсах, очень легко определить, что именно делает тот или иной сервис.
**Фактически, контракты являются краткой документацией к API Laravel**. 

Кроме того, когда вы у себя в приложении внедряете в классы зависимости от простых интерфейсов, в вашем коде легче разобраться и его проще
поддерживать. Вместо того, чтобы заставлять коллег разбираться, какие методы вашего большого и сложного класса можно использовать извне,
вы адресуете их к простому и понятному интерфейсу.

<a name="contract-reference"></a>
## Таблица основных контрактов

Вот таблица соответствий контрактов Laravel 5 фасадам Laravel 4 :

Контракт  |  Фасад Laravel 4.x 
------------- | -------------
[Illuminate\Contracts\Auth\Пгфкв](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php) | Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php) | Password
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) |  &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="how-to-use-contracts"></a>
## Использование контрактов

Как получить в своем классе реализацию заданного контракта? Очень просто. Достаточно в аргументах конструктора явно указать в качестве
типа аргумента название соответствующего контракта - и при его создании там окажется реализация этого интерфейса. Это происходит за счет
того, что почти все классы Laravel (контроллеры, слушатели событий и очередей, роуты, фильтры роутов) регистрируются в сервис-контейнере
и в процессе создания экземпляра класса из контейнера в него встраиваются все определенные таким образом зависимости.
Фактически все происходит автоматически, вам не приходится задумываться об этом механизме.

Например, взглянем на слушателя событий:

	<?php namespace App\Events;

	use App\User;
	use Illuminate\Contracts\Queue\Queue;

	class NewUserRegistered {

		/**
		 * The queue implementation.
		 */
		protected $queue;

		/**
		 * Create a new event listener instance.
		 *
		 * @param  Queue  $queue
		 * @return void
		 */
		public function __construct(Queue $queue)
		{
			$this->queue = $queue;
		}

		/**
		 * Handle the event.
		 *
		 * @param  User  $user
		 * @return void
		 */
		public function fire(User $user)
		{
			// Queue an e-mail to the user...
		}

	}

Если вы хотите больше узнать о сервис-контейнере, прочтите [соответствующий раздел документации](/docs/master/container).