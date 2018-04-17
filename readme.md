<p align="center"><img src="https://laravel.com/assets/img/components/logo-laravel.svg"></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/d/total.svg" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/v/stable.svg" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://poser.pugx.org/laravel/framework/license.svg" alt="License"></a>
</p>

## License

The Laravel framework is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).

## Installing Laravel

	composer create-project --prefer-dist laravel/laravel amir-project

### After Clone
	1. Update .env, set Database name
	2. php artisan key:generate

		installed: 3,4,7,8,9

	    for languages
	    ****Code****

		2.1
		Route::get('language/{locale}', function ($locale ='en'){
		    session()->put('locale', $locale);
		    return back();
		});
		
		2.2
		php artisan make:middleware SetLocale

		2.3
		app/Http/Middleware/SetLocale.php
		<?php 
			namespace App\Http\Middleware;
			use Closure, App, Session;

			class SetLocale {

			    public function handle($request, Closure $next)
			    {
			        if(!session('locale'))
			        {
			            session()->put('locale', \Config::get('app.locale'));
			        }
			        app()->setLocale(session('locale'));
			        return $next($request);
			    }
			}

		2.4
		In the protected $middlewareGroups array of the app/Http/Kernel.php file.
		    protected $middlewareGroups = [
		        'web' => [
		        	...
					\App\Http\Middleware\SetLocale::class,

		using
		{{ App::getLocale()}}
		{{ trans('general.title') }}

		for Helper
	    ****Code****

			append composer.json in the "autoload": {
	        "files": ["app/Helpers/register.php"]
			
			app/Helpers/register.php
			<?php
	        	include_once 'TextHelpers.php';

	        app/Helpers/TextHelpers.php
	        <?php ...

			run composer dump-autoload

	        using
			persian_normalizer($text);
			$news_record->slug = persian_slug($text);
			$news_record->slug = turkish_slug($text);
			
			Requests/
			protected function getValidatorInstance() {
				/*
					Automatic generate slug
				*/
			    $this->merge(['slug'=> turkish_slug($this->input('title'))]);
			    return parent::getValidatorInstance();
			}

### init Nodejs
		
		... from within our project folder
		$ npm init (Optional)
		$ npm install gulp -g

		* install gulp plugin
		$ npm install in the gulpfile.js

		Remove node_modules
		$ npm install rimraf -g $ rimraf node_modules

		* install bower
		Delete bower_components in your root folder
		Create a .bowerrc file in the root
		In the file write this code {"directory" : "resources/vendor"}
		Run a bower install
		$ npm install -g bower

### Remove Public from url
		Reaname /server.php to index.php
		Move public/.htaccess to /.htaccess
		Move public/quickadmin to resources
		Find  "url('quickadmin/" where: \vendor Replace "url('resources/quickadmin/"
		php artisan vendor:publish --force

## INSTALL PLUGINS
	Note **: don't composer update
	cd amir-project

#### 1. quickadmin
	** composer require laraveldaily/quickadmin
		insert `Laraveldaily\Quickadmin\QuickadminServiceProvider::class,` to your `\config\app.php` providers.

		php artisan quickadmin:install
		insert App/Http/Kernel.php in the $routeMiddleware: 'role' => \Laraveldaily\Quickadmin\Middleware\HasPermissions::class,

#### 2. filemanager
	**  composer require unisharp/laravel-filemanager:~1.8

        insert config/app.php in the $providers: Unisharp\Laravelfilemanager\LaravelFilemanagerServiceProvider::class,

        set asset path vendor index.blade.php and LfmHelpers.php
        php artisan vendor:publish --tag=lfm_config
		php artisan vendor:publish --tag=lfm_public
		php artisan route:clear
		php artisan config:clear
        *****Code******
    	In blade:

    	<script src="{!!url('/resources/vendors/ckeditor')!!}/ckeditor.js"></script>
		<script>

		    CKEDITOR.replace( 'ckeditorxxx', {
		        language: '{{ App::getLocale() }}',
		        filebrowserImageBrowseUrl: '{{url('/')}}/laravel-filemanager?type=Images',
		        filebrowserImageUploadUrl: '{{url('/')}}/laravel-filemanager/upload?type=Images&_token={{csrf_token()}}',
		        filebrowserBrowseUrl: '{{url('/')}}/laravel-filemanager?type=Files',
		        filebrowserUploadUrl: '{{url('/')}}/laravel-filemanager/upload?type=Files&_token={{csrf_token()}}'
		    });	
		</script>
		{!! Form::textarea('content', old('content',$page->content), ['class'=>'form-control', 'id'=>'ckeditorxxx']) !!}
		***********

#### 3,4. Image/ImageCache
	composer require intervention/image
    composer require intervention/imagecache
		insert config/app.php in the $providers: Intervention\Image\ImageServiceProvider::class,
		insert config/app.php in the alias: 'Image' => Intervention\Image\Facades\Image::class,
		php artisan vendor:publish --provider="Intervention\Image\ImageServiceProviderLaravel5"
		note: the requested PHP extension fileinfo
		*******Code*******
		// for size image manegment
		// for daynamic images in admin panel
		Route::get('/photo/{size}/{service}/{name}', function ( $size, $service, $name ){
			return imageResizeCache(config('app.admin').'/public/uploads/',$size, $service, $name);
		});
		// for static images in local
		Route::get('/fix/{size}/{service}/{name}', function ( $size, $service, $name ){
			return imageResizeCache('/resources/vendor/',$size, $service, $name);
		});

		function imageResizeCache ($path = null, $size = null, $service = null, $name = null) {
			if (!is_null($path) && !is_null($size) && !is_null($service) && !is_null($name)) {
				
				$size        = explode('x', $size);
				$cache_image = Image::cache(function ($image) use ($path, $size, $service, $name) {
					if ( 'null' == $size[0] || 'null' == $size[1]) {
				    	return $image->make(url( $path . $service .'/'. $name))
				    		     	->resize($size[0], $size[1], function ($constraint) {
																    $constraint->aspectRatio();
																});
					} else {
						return $image->make(url( $path . $service .'/'. $name))
				    		     	->resize($size[0], $size[1]);
					}
					
				}, env('CACHE_PHOTO_MINUTE',10)); // default cache for 10 minutes

				return Response::make($cache_image, 200, ['Content-Type' =>'image']);
			} else {
				abort(404);
			}
		}

		// for test <img src="{{ url('/photo/100x100/somephoto.jpg') }}">
		**************

#### 5. unique validator
		[Documentation](https://github.com/felixkiss/uniquewith-validator)
	composer require felixkiss/uniquewith-validator
		insert config/app.php in the $providers: Felixkiss\UniqueWithValidator\ServiceProvider::class,

#### 6. recaptcha
    composer require greggilbert/recaptcha:dev-master
        //info: https://github.com/greggilbert/recaptcha
        insert config/app.php in the $providers: Greggilbert\Recaptcha\RecaptchaServiceProvider::class,
        insert config/app.php in the alias     : 'Recaptcha' => Greggilbert\Recaptcha\Facades\Recaptcha::class,
        php artisan vendor:publish --provider="Greggilbert\Recaptcha\RecaptchaServiceProvider"
        /config/recaptcha.php, enter your reCAPTCHA public and private keys.
        resources/lang/[lang]/validation.php: "recaptcha" => 'The :attribute field is not correct.',

#### 7. debuger
	composer require barryvdh/laravel-debugbar --dev
		insert config/app.php in the $providers: Barryvdh\Debugbar\ServiceProvider::class,
		insert config/app.php in the alias	   : 'Debugbar' => Barryvdh\Debugbar\Facade::class,
		php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"

#### 8. Collective
	composer require laravelcollective/html:5.2.*
		insert config/app.php in the $providers: Collective\Html\HtmlServiceProvider::class,
		insert config/app.php in the alias	   : 'Form' => Collective\Html\FormFacade::class,
      											 'Html' => Collective\Html\HtmlFacade::class,

#### 9. firebase
    composer require mpociot/laravel-firebase-sync
		This package requires you to add the following section to your config/services.php file:
		'firebase' => [
		    'api_key' => 'API_KEY', // Only used for JS integration
		    'auth_domain' => 'AUTH_DOMAIN', // Only used for JS integration
		    'database_url' => 'https://your-database-at.firebaseio.com',
		    'secret' => 'DATABASE_SECRET',
		    'storage_bucket' => 'STORAGE_BUCKET', // Only used for JS integration
		]

#### 10. filter
    composer require tucker-eric/eloquentfilter
		insert config/app.php in the $providers:EloquentFilter\ServiceProvider::class,
		php artisan vendor:publish --provider="EloquentFilter\ServiceProvider"
		...

## Backup DataBase
	mysqldump --opt -u root dbname > E:\xampp\htdocs\projectname\database\dbname.dump
	mysql -u root -p***** dbname < /var/www/html/projectname/database/dbname.dump

    mysqldump --opt -uroot -p***** dbname > /var/www/html/projectname/database/dbname-21102016.dump
	mysql -u root dbname < E:\xampp\htdocs\projectname\database\dbname.dump

## Clean && Cache
	php artisan cache:clear && php artisan cache:table && php artisan route:clear && php artisan route:cache && php artisan view:clear && php artisan config:clear && php artisan config:cache && php artisan debugbar:clear && php artisan clear-compiled && composer dump-autoload

## Note 
    Create Class ViewComposerServiceProvider in App\Providers\ViewComposerServiceProvider.php

	AppServiceProvider@boot:
       view()->composer('*', 'App\Providers\ViewComposerServiceProvider');
    
    public function compose(View $view)
    {
    // $setting = \Cache::remember('setting', env('CACHE_EXPIRE'), function ()  use($lang){
        $setting = SettingsTranslate::with('settings')->where('lang', $lang)->first();
        // return $setting;
    // });


        $view->with(compact('setting'));
    }
## Firebase Commands
	$ npm install -g firebase-tools
	$ firebase login --interactive
	$ firebase init
	$ firebase deploy
	$ firebase use <alias_or_project_id>

	$npm ls -g --depth=0

	$ npm install request
