# Laravel-Backpack-admin-panel
This guide will help you create your laravel backpack admin panel 
% Laravel Backpack/base %https://laravel-backpack.readme.io/docs/install-on-laravel-54

> composer require backpack/base

Add provider to config\app.php at the bottom in providers

> Backpack\Base\BaseServiceProvider::class,

now run these commands

> php artisan vendor:publish --provider="Backpack\Base\BaseServiceProvider"
> php artisan vendor:publish --provider="Prologue\Alerts\AlertsServiceProvider"
> php artisan migrate

Now open Providers/AppServiceProvider and paste this at the top

> use Illuminate\Support\Facades\Schema;

and in boot function

> Schema::defaultStringLength(191);

add this is user model at top

> use Backpack\Base\app\Notifications\ResetPasswordNotification as ResetPasswordNotification;

and add this method

public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }

now run 

> composer require backpack/generators --dev
> composer require laracasts/generators --dev

open composer.jason file and add this require-dev

> "laracasts/generators": "dev-master as 1.1.4",

now run 

> composer update

now making crud application

> composer require backpack/crud

and add this in config\app in providers at the bottom

> Backpack\CRUD\CrudServiceProvider::class,

now run 

> php artisan elfinder:publish 

resolving elfinder issues

> composer require barryvdh/laravel-elfinder

goto config\app and add this in providers

> Barryvdh\Elfinder\ElfinderServiceProvider::class

run again to check 

> php artisan elfinder:publish
		
> php artisan vendor:publish --provider="Backpack\CRUD\CrudServiceProvider" --tag="public" 
> php artisan vendor:publish --provider="Backpack\CRUD\CrudServiceProvider" --tag="lang" 
> php artisan vendor:publish --provider="Backpack\CRUD\CrudServiceProvider" --tag="config" 
> php artisan vendor:publish --provider="Backpack\CRUD\CrudServiceProvider" --tag="elfinder"

go to config\filesystems

and add this 

'uploads' => [
            'driver' => 'local',
            'root' => public_path('uploads'),
        ],

and add folder "uploads" in public folder

goto resources\views\vendor\backpack\base\inc\sidebar.blade.php 
and add this in <ul>

> <li><a href="{{ url(config('backpack.base.route_prefix', 'admin') . '/elfinder') }}"><i class="fa fa-files-o"></i> <span>File manager</span></a></li>

... optional .... &for language&

> composer require backpack/langfilemanager

add this in config\app in providers at the bottom

> Backpack\LangFileManager\LangFileManagerServiceProvider::class,

> php artisan migrate --path=vendor/backpack/langfilemanager/src/database/migrations
> php artisan db:seed --class="Backpack\LangFileManager\database\seeds\LanguageTableSeeder"
> php artisan vendor:publish --provider="Backpack\LangFileManager\LangFileManagerServiceProvider" --tag="config"
> php artisan vendor:publish --provider="Backpack\LangFileManager\LangFileManagerServiceProvider" --tag="lang" 

paste these links in sidebar.blade.php

<li><a href="{{ url(config('backpack.base.route_prefix', 'admin') . '/language') }}"><i class="fa fa-flag-o"></i> <span>Languages</span></a></li>
<li><a href="{{ url(config('backpack.base.route_prefix', 'admin') . '/language/texts') }}"><i class="fa fa-language"></i> <span>Language Files</span></a></li>

now run  &for backup&

> composer require backpack/backupmanager

add these in config\app in providers at bottom

> Spatie\Backup\BackupServiceProvider::class,
> Backpack\BackupManager\BackupManagerServiceProvider::class,

now run 

> php artisan vendor:publish --provider="Backpack\BackupManager\BackupManagerServiceProvider"

add this in config/filesystems.php

'backups' => [
            'driver' => 'local',
            'root'   => storage_path('backups'), // that's where your backups are stored by default: storage/backups
			

        ],

now we are going to customer crud using laravel crud generator
make sure you have already downloaded the generators described above 

goto providers\AppServiceProvider and paste this 
--it's for the localdevelopment--

public function register()
{
	if ($this->app->environment() == 'local') {
		$this->app->register('Laracasts\Generators\GeneratorsServiceProvider');
	}
}

now run this

> php artisan make:migration:schema create_customers_table --model=0

add your attributes in customers migration and run migration

now run 

> php artisan backpack:crud customer

creating routes for admin

> Route::group(['prefix'=>'admin','middleware'=>'admin'], function(){
    CRUD::resource('customer','Admin\CustomerCrudController');
    });
	
now we are making &page manager&

> composer require backpack/pagemanager

add these in config\app in providers

> Cviebrock\EloquentSluggable\ServiceProvider::class, 
> Backpack\PageManager\PageManagerServiceProvider::class,

now run 

> php artisan vendor:publish --provider="Backpack\PageManager\PageManagerServiceProvider"

> php artisan migrate

and add this link 

<li><a href="{{ url(config('backpack.base.route_prefix').'/page') }}"><i class="fa fa-file-o"></i> <span>Pages</span></a></li>


...view created pages...

> Route::get('{page}/{subs?}', ['uses' => 'PageController@index'])
    ->where(['page' => '^((?!admin).)*$', 'subs' => '.*']);
	
create a page controller

> php artisan make:controller PageController


use Illuminate\Http\Request;
use Backpack\PageManager\app\Models\Page;
use App\Http\Controllers\Controller;

class PageController extends Controller
{
    public function index($slug)
    {
        $page = Page::findBySlug($slug);

        if (!$page)
        {
            abort(404, 'Please go back to our <a href="'.url('').'">homepage</a>.');
        }

        $this->data['title'] = $page->title;
        $this->data['page'] = $page->withFakes();

        return view('pages.'.$page->template, $this->data);
    }
}

create a "pages" folder inside views folder
