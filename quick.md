# Laravel: Inicio rápido

- [Instalación](#instalacion)
- [Enrutado](#routing)
- [Creando una vista](#creating-a-view)
- [Creando una Migración](#creating-a-migration)
- [Eloquent ORM](#eloquent-orm)
- [Mostrando la informacion](#displaying-data)

<a name="installation"></a>
## Instalación

### Usando el instalador de Laravel

Primero, descarga el [archivo de instalación PHAR de laravel](http://laravel.com/laravel.phar). Por conveniencia, cambie el nombre del archivo a `laravel` y muevalo a la carpeta `/usr/local/bin`. Una vez instalado, el comando `laravel new` creará una instalación limpia de Laravel en el directorio especificado. Por ejemplo, `laravel new blog` crearía el directorio llamado `blog` que contendra una instalación limpia de Laravel con todas las dependencias instaladas. Este método de instalación es mas rápido que el instalar usando Composer.

### Usando Composer

El framework Laravel utiliza [Composer](http://getcomposer.org) para su instalación y la gestión de dependencias. Si no lo tiene instalado, puede empezar por [instalar Composer es su maquina](http://getcomposer.org/doc/00-intro.md).

Ahora puedes instalar Laravel utilizado el siguiente comando en tu consola:

	composer create-project laravel/laravel nombre-de-proyecto --prefer-dist

Este comando descargará e instalará una copia limpia de Laravel en la carpeta `nombre-de-proyecto` en directorio que se encuentre en el momento.

Si prefiere, otra alternativa es descargar manualmente una copia del [repositorio de Laravel en Github](https://github.com/laravel/laravel/archive/master.zip). Lo siguiente es ejecutar el comando `composer install` en el directorio raíz del proyecto creado manualmente con anterioridad. Este comando dscargará e instalará las dependencia del framework.

### Permisos

Despúes de instalar Laravel, s probablemente se necesite darle permisos de escritura al navegador para el directorio `app/storage`. Para mas detalles de configuración ver la docmunetación de [Instalación](/page/installation).

### Serving Laravel

Typically, you may use a web server such as Apache or Nginx to serve your Laravel applications. If you are on PHP 5.4+ and would like to use PHP's built-in development server, you may use the `serve` Artisan command:

	php artisan serve

<a name="directories"></a>
### Directory Structure

After installing the framework, take a glance around the project to familiarize yourself with the directory structure. The `app` directory contains folders such as `views`, `controllers`, and `models`. Most of your application's code will reside somewhere in this directory. You may also wish to explore the `app/config` directory and the configuration options that are available to you.

<a name="routing"></a>
## Routing

To get started, let's create our first route. In Laravel, the simplest route is a route to a Closure. Pop open the `app/routes.php` file and add the following route to the bottom of the file:

	Route::get('users', function()
	{
		return 'Users!';
	});

Now, if you hit the `/users` route in your web browser, you should see `Users!` displayed as the response. Great! You've just created your first route.

Routes can also be attached to controller classes. For example:

	Route::get('users', 'UserController@getIndex');

This route informs the framework that requests to the `/users` route should call the `getIndex` method on the `UserController` class. For more information on controller routing, check out the [controller documentation](/page/controllers).

<a name="creating-a-view"></a>
## Creating A View

Next, we'll create a simple view to display our user data. Views live in the `app/views` directory and contain the HTML of your application. We're going to place two new views in this directory: `layout.blade.php` and `users.blade.php`. First, let's create our `layout.blade.php` file:

	<html>
		<body>
			<h1>Laravel Quickstart</h1>

			@yield('content')
		</body>
	</html>

Next, we'll create our `users.blade.php` view:

	@extends('layout')

	@section('content')
		Users!
	@stop

Some of this syntax probably looks quite strange to you. That's because we're using Laravel's templating system: Blade. Blade is very fast, because it is simply a handful of regular expressions that are run against your templates to compile them to pure PHP. Blade provides powerful functionality like template inheritance, as well as some syntax sugar on typical PHP control structures such as `if` and `for`. Check out the [Blade documentation](/page/templates) for more details.

Now that we have our views, let's return it from our `/users` route. Instead of returning `Users!` from the route, return the view instead:

	Route::get('users', function()
	{
		return View::make('users');
	});

Wonderful! Now you have setup a simple view that extends a layout. Next, let's start working on our database layer.

<a name="creating-a-migration"></a>
## Creating A Migration

To create a table to hold our data, we'll use the Laravel migration system. Migrations let you expressively define modifications to your database, and easily share them with the rest of your team.

First, let's configure a database connection. You may configure all of your database connections from the `app/config/database.php` file. By default, Laravel is configured to use MySQL, and you will need to supply connection credentials within the database configuration file. If you wish, you may change the `driver` option to `sqlite` and it will use the SQLite database included in the `app/database` directory.

Next, to create the migration, we'll use the [Artisan CLI](/page/artisan). From the root of your project, run the following from your terminal:

	php artisan migrate:make create_users_table

Next, find the generated migration file in the `app/database/migrations` folder. This file contains a class with two methods: `up` and `down`. In the `up` method, you should make the desired changes to your database tables, and in the `down` method you simply reverse them.

Let's define a migration that looks like this:

	public function up()
	{
		Schema::create('users', function($table)
		{
			$table->increments('id');
			$table->string('email')->unique();
			$table->string('name');
			$table->timestamps();
		});
	}

	public function down()
	{
		Schema::drop('users');
	}

Next, we can run our migrations from our terminal using the `migrate` command. Simply execute this command from the root of your project:

	php artisan migrate

If you wish to rollback a migration, you may issue the `migrate:rollback` command. Now that we have a database table, let's start pulling some data!

<a name="eloquent-orm"></a>
## Eloquent ORM

Laravel ships with a superb ORM: Eloquent. If you have used the Ruby on Rails framework, you will find Eloquent familiar, as it follows the ActiveRecord ORM style of database interaction.

First, let's define a model. An Eloquent model can be used to query an associated database table, as well as represent a given row within that table. Don't worry, it will all make sense soon! Models are typically stored in the `app/models` directory. Let's define a `User.php` model in that directory like so:

	class User extends Eloquent {}

Note that we do not have to tell Eloquent which table to use. Eloquent has a variety of conventions, one of which is to use the plural form of the model name as the model's database table. Convenient!

Using your preferred database administration tool, insert a few rows into your `users` table, and we'll use Eloquent to retrieve them and pass them to our view.

Now let's modify our `/users` route to look like this:

	Route::get('users', function()
	{
		$users = User::all();

		return View::make('users')->with('users', $users);
	});

Let's walk through this route. First, the `all` method on the `User` model will retrieve all of the rows in the `users` table. Next, we're passing these records to the view via the `with` method. The `with` method accepts a key and a value, and is used to make a piece of data available to a view.

Awesome. Now we're ready to display the users in our view!

<a name="displaying-data"></a>
## Displaying Data

Now that we have made the `users` available to our view. We can display them like so:

	@extends('layout')

	@section('content')
		@foreach($users as $user)
			<p>{{ $user->name }}</p>
		@endforeach
	@stop

You may be wondering where to find our `echo` statements. When using Blade, you may echo data by surrounding it with double curly braces. It's a cinch. Now, you should be able to hit the `/users` route and see the names of your users displayed in the response.

This is just the beginning. In this tutorial, you've seen the very basics of Laravel, but there are so many more exciting things to learn. Keep reading through the documentation and dig deeper into the powerful features available to you in [Eloquent](/page/eloquent) and [Blade](/page/templates). Or, maybe you're more interested in [Queues](/page/queues) and [Unit Testing](/page/testing). Then again, maybe you want to flex your architecture muscles with the [IoC Container](/page/ioc). The choice is yours!
