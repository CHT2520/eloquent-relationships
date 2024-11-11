# Eloquent Relationships

This practical explore the use of Eloquent relationships, specifically a one-to-many relationship which is implemented in Eloquent using _Has Many_ and _Belongs To_.

The starting point for this practical is the **Intro to Laravel** practical we did a couple of weeks ago.

-   You'll probably need to change the _DocumentRoot_ settings in your _httpd.conf_ file to set _film-app_ example as the web server root e.g.

    ```
    DocumentRoot "/xampp/htdocs/film-app/public"
    <Directory "/xampp/htdocs/film-app/public">
    ```

-   If you don't have your own copy of this work you can get a copy from https://github.com/CHT2520/intro-to-laravel-code.

We will add an additional database table to this example to store information about the age rating certificate for films (U, PG, 15 etc.) and then modify the create and read functionality to make use of this table.

## Setting Up The Database Tables

### Migrations

First add a _certificates_ database table to the project. 
- Using Artisan enter the following

```
php artisan make:migration create_certificates_table --create=certificates
```

-   Modify this migration file and edit the `up()` method to specify the following schema for the table

```php
public function up(): void
    {
        Schema::create('certificates', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description');
            $table->string('filename');
            $table->timestamps();
        });
    }
```

There is a one-to-many relationship between certificate and film, so we need to add a foreign key to the _films_ table. 
- Again using Artisan enter the following command

```
php artisan make:migration add_certificate_id_to_films_table --table=films
```

-   In this file edit the `up()` and `down()` methods to add and remove a _certificate_id_ column on the _films_ table.

```php
public function up(): void
{
    Schema::table('films', function (Blueprint $table) {
        $table->unsignedBigInteger('certificate_id');
        $table->foreign('certificate_id')->references('id')->on('certificates');
    });
}

public function down(): void
{
    Schema::table('films', function (Blueprint $table) {
        $table->dropForeign('certificate_id');
        $table->dropColumn('certificate_id');
    });
}
```

-   Using Artisan run

```
php artisan migrate:refresh
```

-   Using phpmyadmin check the tables have been set up.

### Seeding

-   Using Artisan enter the following command:

```
php artisan make:seeder CertificateSeeder
```

-   Open the newly generated seeder file and add some seeds i.e.

```php
namespace Database\Seeders;

use Illuminate\Support\Facades\DB;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

class CertificateSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        DB::table('certificates')->insert(['name' => 'U', 'description' => 'Suitable for all', 'filename' => "u.png"]);
        DB::table('certificates')->insert(['name' => 'PG', 'description' => 'Parental guidance', 'filename' => "pg.png"]);
        DB::table('certificates')->insert(['name' => '12a', 'description' => 'Suitable for twelve years and over', 'filename' => "12a.png"]);
        DB::table('certificates')->insert(['name' => '15', 'description' => 'Suitable for only for fifteen years and over', 'filename' => "15.png"]);
        DB::table('certificates')->insert(['name' => '18', 'description' => 'Suitable for only for eighteen years and over', 'filename' => "18.png"]);
    }
}
```

-   Next modify the existing _FilmSeeder.php_ file. Change the `run()` method so that we specify the _certificate_id_ for each film.

```php
public function run()
{
    DB::table('films')->insert(['title' => 'Jaws', 'year' => '1975', 'duration' => 124, 'certificate_id' => 4]);
    DB::table('films')->insert(['title' => 'Winter\'s Bone', 'year' => '2010', 'duration' => 100, 'certificate_id' => 4]);
    DB::table('films')->insert(['title' => 'Do The Right Thing', 'year' => '1989', 'duration' => 120, 'certificate_id' => 4]);
    DB::table('films')->insert(['title' => 'The Incredibles', 'year' => '2004', 'duration' => 15, 'certificate_id' => 1]);
    DB::table('films')->insert(['title' => 'The Godfather', 'year' => '1972', 'duration' => 177, 'certificate_id' => 5]);
    DB::table('films')->insert(['title' => 'Spirited Away', 'year' => '2001', 'duration' => 124, 'certificate_id' => 2]);
    DB::table('films')->insert(['title' => 'Moonlight', 'year' => '2016', 'duration' => 111, 'certificate_id' => 4]);
}
```

-   Add the CertificateSeeder to the `run()` method of _DatabaseSeeder_

```php
public function run(): void
{
    $this->call([
        CertificateSeeder::class,
        FilmSeeder::class
    ]);
}
```

> Note that we want to seed the _certificates_ tables first so we have _certificate_id_ values to use as foreign keys in the _films_ table.

-   Run the seeding

```
php artisan db:seed
```

-   Check phpmyadmin to make sure the tables have been populated

> Remember if you get in a mess with setting up the database tables you can always run `php artisan migrate:refresh --seed` to re-run the migrations and seeding.

## Setting Up The Eloquent Relationship

First create a _Certificate_ model.

-   Using Artisan enter the following command:

```
php artisan make:model Certificate
```

Next, we need to tell Eloquent about the relationship between _Certificate_ and _Film_. Modify _Certificate.php_ to specify a _hasMany_ relationship with _Film_.

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Certificate extends Model
{
    public function films(): HasMany
    {
        return $this->hasMany(Film::class);
    }
}

```

-   Modify _Film.php_ to specify a _belongsTo_ relationship with _Certificate_.

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Film extends Model
{
    public function certificate(): BelongsTo
    {
        return $this->belongsTo(Certificate::class);
    }
}
```

> Pay careful attention to the naming conventions we have used. A film belongs to a single certificate so we use a singular `certificate()` for the method name. A certificate has many films, so we use the plural `films()` as the method name.

- See https://laravel.com/docs/11.x/eloquent-relationships for more detail on how relationships work in Eloquent. 
### Testing This Works

An nice way to test this works without having use any views is using `artisan tinker`. Using Artisan enter the following command:

```
php artisan tinker
```

This allows us to enter PHP code where we can interact with our app.

-   Try entering the following:

```
$film = App\Models\Film::find(3)
```

And then

```
$film->certificate->name
```

You should be able to retrieve the certificate of the third film.

> Note we use the object-oriented arrow syntax `$film->certificate->name` to access properties of the certificate object.

-   Now test the Certificate model

```
$certificate = App\Models\Certificate::find(4)
```

Followed by

```
$certificate->films
```

And you should see the collection of films that belong to this certificate.

-   Enter _ctrl+c_ to exit out of tinker.

## Using The Relationship

Now we are confident the relationship works we can use it in the app.

### Viewing Film Certificates

-   Modify the index view (_index.blade.php_) to show certificate information for each film

```html
<x-layout title="List the films">
    <h1>Here's a list of films</h1>
    @foreach ($films as $film)
    <p>
        <a href="/films/{{$film->id}}">
            {{$film->title}} ({{$film->certificate->name}})
        </a>
    </p>
    @endforeach
</x-layout>
```
-   In a browser test this works.

### Adding a New Film

On the create view, when the user adds a new film they will need to choose the certificate for the new film, therefore we need to pass a collection of all the certificates to this view.

-   Modify the _create()_ method of _FilmController_ so that a complete list of all the certificates is passed to the create view (you will also need to import the Certificate class at the top of the file (like we did for the Film class on line 5)).

```php
function create()
{
    $certificates = Certificate::all();
    return view('films.create', ['certificates' => $certificates]);
}
```

-   Add the following _div_ element inside the form in _create.blade.php_. This will output a group of radio buttons where the user can select a certificate.

```html
<div>
    <fieldset>
        <legend>Select a certificate for your film:</legend>
        @foreach ($certificates as $certificate)
        <label for="{{$certificate->name}}">
            <input
                type="radio"
                name="certificate_id"
                id="{{$certificate->name}}"
                value="{{$certificate->id}}"
            />
            {{$certificate->name}}
        </label>
        @endforeach
    </fieldset>
</div>
```
-   Visit http://localhost/films/create and make sure the user can select a certificate for the new film.
- Use the browser tools to inspect the page and view the HTML that has been generated. 

### Saving the certificate

-   This is fairly easy, edit the `store()` method in _FilmController_ to add a certificate id for the new film i.e.

```php
function store(Request $request)
{
    $film = new Film();
    $film->title = $request->title;
    $film->year = $request->year;
    $film->duration = $request->duration;
    $film->certificate_id = $request->certificate_id;
    $film->save();
    return redirect('/films');
}
```

-   In the browser, try adding a new film. This should all work.

## Implementing Functionality for the _certificates_ Table

One additional feature we could add is viewing information about the different certificates

-   Using Artisan make a new _CertificateController_

```
php artisan make:controller CertificateController
```

-   Add an `index()` method to this controller

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class CertificateController extends Controller
{
    function index(){
        dd("Certificate Controller working");
    }
}
```

> The `dd()`` command simply displays a message and the stops further code from executing. It's a quick way to check we have set up the routing properly.

In _web.php_ add a route for the `index()` method of the _CertificateController_

```php
Route::get('/certificates', [CertificateController::class, 'index']);
```

-   Test this route works i.e. http://localhost/certificates. You should get the message from the `dd()` function.

-   Now modify the _CertificateController_ class to use Eloquent to get hold of all the certificates and pass them to a view.

```php
namespace App\Http\Controllers;

use App\Models\Certificate;
use Illuminate\Http\Request;

class CertificateController extends Controller
{
    function index()
    {
        $certificates = Certificate::all();
        return view('certificates.index', ['certificates' => $certificates]);
    }
}
```

-   Add a new _certificates_ folder inside the _resources/views_ folder. Create a new _index.blade.php_ file inside the _certificates_ folder and paste in the following code:

```html
<x-layout title="List the certificates">
    <h1>Certificates</h1>
    @foreach ($certificates as $certificate)
    <h2>{{$certificate->name}} Certificate</h2>
    <p>{{$certificate->description}}</p>
    <ul>
        @if($certificate->films->isEmpty())
        <li>No films classified under this certificate</li>
        @else @foreach ($certificate->films as $film)
        <li>{{$film->title}}</li>
        @endforeach @endif
    </ul>
    @endforeach
</x-layout>
```

> Note that for each certificate object we are able access a collection of films that this certificate 'has'.

-   Open _layout.blade.php_ and modify the navigation list to provide a link to the certificates page.

```html
<ul>
    <li><a href="/films">Home</a></li>
    <li><a href="/films/create">Add new film</a></li>
    <li><a href="/films/about">About</a></li>
    <li><a href="/certificates">Certificates</a></li>
</ul>
```

-   Test this works.

## Test Your Understanding

-   If you look at the migration and seeding for the certificates table you will see we specified a filename for each certificate. It would be nice if we could use this to display images for the certificates.
    -   Have a look online e.g. https://en.wikipedia.org/wiki/British_Board_of_Film_Classification#Current_certificates and download copies of images for each certificate.
    -   Create a new folder called _images_ in the _public_ folder.
        -   Save your certificate images in this folder.
    -   Make the filenames in the database match the names of the files you have downloaded.
    -   Use the `asset` helper function in your views to display the certificates images - see https://laravel.com/docs/11.x/helpers#method-asset

## There's More

This is a fairly brief introduction to relationships in Eloquent. Here are some more topics to investigate:

-   Implementing many-to-many relationships https://laravel.com/docs/11.x/eloquent-relationships#many-to-many
-   Query Scopes - https://laravel.com/docs/11.x/eloquent#query-scopes
-   Eager loading and the "N + 1" query problem - https://laravel.com/docs/11.x/eloquent-relationships#eager-loading
