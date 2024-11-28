(Utilizando duas entidades no projeto)

- Pokemon: name, type, power, image e coach_id
- Coach: name, image

1 - Criação do projeto:

`composer create-project laravel/laravel laravel-pokemon-crud "11.*" --prefer-dist`

`cd laravel-pokemon-crud`

`cp .env.example .env`

`php artisan key:generate`

`code .`

2 - Criando a model com a migration e controller:

`php artisan make:model Pokemon -mc` 

3 - Colocando as variáveis na (database/migrations/***create-pokemon_table.php):

```jsx
	$table->id();
  $table->string('name', 200);
  $table->string('type', 100);
  $table->integer('power');
  $table->timestamps();
```

4 - Depois de atualizar qualquer coisa nas migrations:

`php artisan migrate:fresh`

5 - Botar os campos preenchíveis na (app/Models/Pokemon.php):

```jsx
protected $fillable = [
        'name',
        'type',
        'power',
    ];
```

6 - Definir as rotas em (routes/web.php):

```jsx
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\PokemonController;

Route::get('pokemon', [PokemonController::class, 'index']);
Route::get('pokemon/create', [PokemonController::class, 'create']);
Route::post('pokemon', [PokemonController::class, 'store']);
Route::get('pokemon/{id}/edit', [PokemonController::class, 'edit']);
Route::put('pokemon/{id}', [PokemonController::class, 'update']);
Route::delete('pokemon/{id}', [PokemonController::class, 'destroy']);
```

7 - Configurar a Controller (app/Http/Controllers/PokemonController.php):

```jsx
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;
use App\Models\Coach;
use App\Models\Pokemon;

class PokemonController extends Controller
{
    public function index()
    {
        $pokemon = Pokemon::all();
        return view('pokemon.index', compact('pokemon'));
    }

    public function create()
    {
        return view('pokemon.create');
    }

    public function store(Request $request)
    {
        Pokemon::create($request->all());
        return redirect('pokemon')->with('success', 'Pokemon created successfully.');
    }

    public function edit($id)
    {
        $pokemon = Pokemon::findOrFail($id);
        return view('pokemon.edit', compact('pokemon'));
    }

    public function update(Request $request, $id)
    {
        $pokemon = Pokemon::findOrFail($id);
        $pokemon->update($request->all());
        return redirect('pokemon')->with('success', 'Pokemon updated successfully.');
    }

    public function destroy($id)
    {
        $pokemon = Pokemon::findOrFail($id);
        $pokemon->delete();
        return redirect('pokemon')->with('success', 'Pokemon deleted successfully.');
    }
}
```

8 - Adicionar o campo de image em (database/migrations/***create_pokemon_table.php):

`$table->*text*('image')***;`*** 

9 - Na model, adicionar o fillable de image:

`'image',` 

10 - Criar link simbólico do armazenamento local:

`php artisan storage:link` 

11 - Modificar as functions store e update na Controller

```jsx
public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'type' => 'required',
            'power' => 'required',
            'image' => 'required|image|mimes:jpeg,png,jpg,gif,webp|max:2048',
        ]);
        $imageName = time().'.'.$request->image->extension();
        $request->image->move(public_path('images'), $imageName);

        $pokemon = new Pokemon();
        $pokemon->name = $request->name;
        $pokemon->type = $request->type;
        $pokemon->power = $request->power;
        $pokemon->image = 'images/'.$imageName;
        $pokemon->save();

        return redirect('pokemon')->with('success', 'Pokemon created successfully.');
    }
```

```jsx
public function update(Request $request, $id)
    {
        $pokemon = Pokemon::findOrFail($id);
        
        if(!is_null($request->image)) {
            $imageName = time().'.'.$request->image->extension();
            $request->image->move(public_path('images'), $imageName);
            $pokemon->image = 'images/'.$imageName;
        }

        $pokemon->name = $request->name;
        $pokemon->type = $request->type;
        $pokemon->power = $request->power;
        $pokemon->save();

        return redirect('pokemon')->with('success', 'Pokemon updated successfully.');
    }
```

12 - Criar a outra entidade (Coach)

`php artisan make:model Coach -mc`

13 - Editar a model Coach e Pokemon

(app/Models/Coach.php):

```jsx
class Coach extends Model
{
    protected $fillable = [
       'image',
       'name' 
    ];
}
```

(app/Models/Pokemon.php):

```jsx
class Pokemon extends Model
{
    protected $fillable = [
        'name',
        'type',
        'power',
        'image',
        'coach_id'
    ];
}
```

14 - Alterar as migrations (database/migrations/***create_pokemon_table.php) e (***create_coaches_table.php):

Pokemon

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('pokemon', function (Blueprint $table) {
            $table->id();
            $table->string('name', 100);
            $table->string('type', 100);
            $table->integer('power');
            $table->text('image');
            $table->foreignId('coach_id')->references('id')->on('coaches');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('pokemon');
    }
};

```

Coach

```jsx
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('coaches', function (Blueprint $table) {
            $table->id();
            $table->string('name', 200);
            $table->text('image');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('coaches');
    }
};

```

15 - Adicionar as funções na Model do Coach e do Pokemon:

Coach:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Coach extends Model
{
    protected $fillable = [
        'image',
        'name',
    ];

    public function pokemon()
    {
        return $this->hasMany(Pokemon::class);
    }
}
```

Pokemon:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Pokemon extends Model
{
    protected $fillable = [
        'name',
        'type',
        'power',
        'image',
        'coach_id'
    ];

    public function coach()
    {
        return $this->belongsTo(Coach::class);
    }
}

```

16 - Adicionar as rotas

```php
<?php

use App\Http\Controllers\ProfileController;
use App\Http\Controllers\CoachController;
use App\Http\Controllers\PokemonController;
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});

Route::get('pokemon', [PokemonController::class, 'index'])->middleware(['auth', 'verified'])->name('index-pokemon');
Route::get('pokemon/create', [PokemonController::class, 'create'])->middleware(['auth', 'verified'])->name('create-pokemon');
Route::post('pokemon', [PokemonController::class, 'store'])->middleware(['auth', 'verified'])->name('store-pokemon');
Route::get('pokemon/{id}/edit', [PokemonController::class, 'edit'])->middleware(['auth', 'verified'])->name('edit-pokemon');
Route::put('pokemon/{id}', [PokemonController::class, 'update'])->middleware(['auth', 'verified'])->name('update-pokemon');
Route::delete('pokemon/{id}', [PokemonController::class, 'destroy'])->middleware(['auth', 'verified'])->name('destroy-pokemon');

Route::get('coaches', [CoachController::class, 'index'])->middleware(['auth', 'verified'])->name('index-coach');
Route::get('coaches/create', [CoachController::class, 'create'])->middleware(['auth', 'verified'])->name('create-coach');
Route::post('coaches', [CoachController::class, 'store'])->middleware(['auth', 'verified'])->name('store-coach');
Route::get('coaches/{id}/edit', [CoachController::class, 'edit'])->middleware(['auth', 'verified'])->name('edit-coach');
Route::put('coaches/{id}', [CoachController::class, 'update'])->middleware(['auth', 'verified'])->name('update-coach');
Route::delete('coaches/{id}', [CoachController::class, 'destroy'])->middleware(['auth', 'verified'])->name('destroy-coach');
```

17 - CoachController:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Coach;

class CoachController extends Controller
{
    public function index()
    {
        $coaches = Coach::all();
        return view('coaches.index', compact('coaches'));
    }

    public function create()
    {
        return view('coaches.create');
    }

    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'image' => 'required|image|mimes:jpeg,png,jpg,gif,webp',
        ]);
        $imageName = time().'.'.$request->image->extension();
        $request->image->move(public_path('images'), $imageName);

        $coach = new Coach();
        $coach->name = $request->name;
        $coach->image = 'images/'.$imageName;
        $coach->save();

        return redirect('coaches')->with('success', 'Coach created successfully.');
    }

    public function edit($id)
    {
        $coach = Coach::findOrFail($id);
        return view('coaches.edit', compact('coach'));
    }

    public function update(Request $request, $id)
    {
        $coach = Coach::findOrFail($id);
        
        if(!is_null($request->image)) {
            $imageName = time().'.'.$request->image->extension();
            $request->image->move(public_path('images'), $imageName);
            $coach->image = 'images/'.$imageName;
        }

        $coach->name = $request->name;
        $coach->save();

        return redirect('coaches')->with('success', 'Coach updated successfully.');
    }

    public function destroy($id)
    {
        $coach = Coach::findOrFail($id);
        $coach->delete();
        return redirect('coaches')->with('success', 'Coach deleted successfully.');
    }
}

```

18 - Editando função create e edit na PokemonController:

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;
use App\Models\Coach;
use App\Models\Pokemon;

class PokemonController extends Controller
{
    public function index()
    {
        $pokemon = Pokemon::all();
        return view('pokemon.index', compact('pokemon'));
    }

    public function create()
    {
        Gate::authorize('create', Pokemon::class);
        $coaches = Coach::all();
        return view('pokemon.create', compact('coaches'));
    }

    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'type' => 'required',
            'power' => 'required',
            'image' => 'required|image|mimes:jpeg,png,jpg,gif,webp|max:2048',
        ]);
        $imageName = time().'.'.$request->image->extension();
        $request->image->move(public_path('images'), $imageName);

        $pokemon = new Pokemon();
        $pokemon->name = $request->name;
        $pokemon->type = $request->type;
        $pokemon->power = $request->power;
        $pokemon->coach_id = $request->coach_id;
        $pokemon->image = 'images/'.$imageName;
        $pokemon->save();

        return redirect('pokemon')->with('success', 'Pokemon created successfully.');
    }

    public function edit($id)
    {
        Gate::authorize('edit', Pokemon::class);
        $pokemon = Pokemon::findOrFail($id);
        $coaches = Coach::all();
        return view('pokemon.edit', compact(['pokemon', 'coaches']));
    }

    public function update(Request $request, $id)
    {
        $pokemon = Pokemon::findOrFail($id);
        
        if(!is_null($request->image)) {
            $imageName = time().'.'.$request->image->extension();
            $request->image->move(public_path('images'), $imageName);
            $pokemon->image = 'images/'.$imageName;
        }

        $pokemon->name = $request->name;
        $pokemon->type = $request->type;
        $pokemon->power = $request->power;
        $pokemon->coach_id = $request->coach_id;
        $pokemon->save();

        return redirect('pokemon')->with('success', 'Pokemon updated successfully.');
    }

    public function destroy($id)
    {
        Gate::authorize('destroy', Pokemon::class);
        $pokemon = Pokemon::findOrFail($id);
        $pokemon->delete();
        return redirect('pokemon')->with('success', 'Pokemon deleted successfully.');
    }
}
```

19 - Autenticação e autorização, rodar no terminal do Bash

`composer require laravel/breeze --dev` 

`php artisan breeze:install` 

- blade
- yes
- 1

`php artisan migrate`

20 -  Para poder usar a parte de autenticação em português, utilizamos os comandos:

`php artisan lang:publish`

`composer require lucascudo/laravel-pt-br-localization --dev` 

`php artisan vendor:publish --tag=laravel-pt-br-localization`

no .env alterar:

```jsx
APP_LOCALE=pt_BR
APP_FALLBACK_LOCALE=pt_BR
APP_FAKER_LOCALE=pt_BR
```

21 - Dentro de app, criar a pasta Policies e dentro, o PokemonPolicy.php:

```jsx
<?php

namespace App\Policies;

use App\Models\Pokemon;
use App\Models\User;

class PokemonPolicy 
{
    public function create(?User $user)
    {
        return !is_null($user);
    }

    public function edit(?User $user)
    {
        return !is_null($user);
    }

    public function destroy(?User $user)
    {
        return !is_null($user);
    }
}
```

22 - Agora em (app/Providers/AppServiceProvider.php):

```jsx
<?php

namespace App\Providers;

use App\Models\Pokemon;
use App\Policies\PokemonPolicy;
use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Gate;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        //
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Gate::policy(Pokemon::class, PokemonPolicy::class);
    }
}
```

23 - Arrumar a PokemonController:

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;
use App\Models\Coach;
use App\Models\Pokemon;

class PokemonController extends Controller
{
    public function index()
    {
        $pokemon = Pokemon::all();
        return view('pokemon.index', compact('pokemon'));
    }

    public function create()
    {
        Gate::authorize('create', Pokemon::class);
        $coaches = Coach::all();
        return view('pokemon.create', compact('coaches'));
    }

    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required',
            'type' => 'required',
            'power' => 'required',
            'image' => 'required|image|mimes:jpeg,png,jpg,gif,webp|max:2048',
        ]);
        $imageName = time().'.'.$request->image->extension();
        $request->image->move(public_path('images'), $imageName);

        $pokemon = new Pokemon();
        $pokemon->name = $request->name;
        $pokemon->type = $request->type;
        $pokemon->power = $request->power;
        $pokemon->coach_id = $request->coach_id;
        $pokemon->image = 'images/'.$imageName;
        $pokemon->save();

        return redirect('pokemon')->with('success', 'Pokemon created successfully.');
    }

    public function edit($id)
    {
        Gate::authorize('edit', Pokemon::class);
        $pokemon = Pokemon::findOrFail($id);
        $coaches = Coach::all();
        return view('pokemon.edit', compact(['pokemon', 'coaches']));
    }

    public function update(Request $request, $id)
    {
        $pokemon = Pokemon::findOrFail($id);
        
        if(!is_null($request->image)) {
            $imageName = time().'.'.$request->image->extension();
            $request->image->move(public_path('images'), $imageName);
            $pokemon->image = 'images/'.$imageName;
        }

        $pokemon->name = $request->name;
        $pokemon->type = $request->type;
        $pokemon->power = $request->power;
        $pokemon->coach_id = $request->coach_id;
        $pokemon->save();

        return redirect('pokemon')->with('success', 'Pokemon updated successfully.');
    }

    public function destroy($id)
    {
        Gate::authorize('destroy', Pokemon::class);
        $pokemon = Pokemon::findOrFail($id);
        $pokemon->delete();
        return redirect('pokemon')->with('success', 'Pokemon deleted successfully.');
    }
}
```

24 - Arrumar rotas:

```
<?php

use App\Http\Controllers\ProfileController;
use App\Http\Controllers\CoachController;
use App\Http\Controllers\PokemonController;
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});

Route::get('pokemon', [PokemonController::class, 'index'])->middleware(['auth', 'verified'])->name('index-pokemon');
Route::get('pokemon/create', [PokemonController::class, 'create'])->middleware(['auth', 'verified'])->name('create-pokemon');
Route::post('pokemon', [PokemonController::class, 'store'])->middleware(['auth', 'verified'])->name('store-pokemon');
Route::get('pokemon/{id}/edit', [PokemonController::class, 'edit'])->middleware(['auth', 'verified'])->name('edit-pokemon');
Route::put('pokemon/{id}', [PokemonController::class, 'update'])->middleware(['auth', 'verified'])->name('update-pokemon');
Route::delete('pokemon/{id}', [PokemonController::class, 'destroy'])->middleware(['auth', 'verified'])->name('destroy-pokemon');

Route::get('coaches', [CoachController::class, 'index'])->middleware(['auth', 'verified'])->name('index-coach');
Route::get('coaches/create', [CoachController::class, 'create'])->middleware(['auth', 'verified'])->name('create-coach');
Route::post('coaches', [CoachController::class, 'store'])->middleware(['auth', 'verified'])->name('store-coach');
Route::get('coaches/{id}/edit', [CoachController::class, 'edit'])->middleware(['auth', 'verified'])->name('edit-coach');
Route::put('coaches/{id}', [CoachController::class, 'update'])->middleware(['auth', 'verified'])->name('update-coach');
Route::delete('coaches/{id}', [CoachController::class, 'destroy'])->middleware(['auth', 'verified'])->name('destroy-coach');

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard');

Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});

require __DIR__.'/auth.php';
```

25 - Agora nas views, vamos criar as pastas e adicionar os modelos prontos:

- layouts/base.blade.php:

```jsx
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title')</title>

    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap" rel="stylesheet" />

    <!-- Styles / Scripts -->
    @if (file_exists(public_path('build/manifest.json')) || file_exists(public_path('hot')))
        @vite(['resources/css/app.css', 'resources/js/app.js'])
    @else
        <style>
            /* ! tailwindcss v3.4.1 | MIT License | https://tailwindcss.com */*,::after,::before{box-sizing:border-box;border-width:0;border-style:solid;border-color:#e5e7eb}::after,::before{--tw-content:''}:host,html{line-height:1.5;-webkit-text-size-adjust:100%;-moz-tab-size:4;tab-size:4;font-family:Figtree, ui-sans-serif, system-ui, sans-serif, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji;font-feature-settings:normal;font-variation-settings:normal;-webkit-tap-highlight-color:transparent}body{margin:0;line-height:inherit}hr{height:0;color:inherit;border-top-width:1px}abbr:where([title]){-webkit-text-decoration:underline dotted;text-decoration:underline dotted}h1,h2,h3,h4,h5,h6{font-size:inherit;font-weight:inherit}a{color:inherit;text-decoration:inherit}b,strong{font-weight:bolder}code,kbd,pre,samp{font-family:ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;font-feature-settings:normal;font-variation-settings:normal;font-size:1em}small{font-size:80%}sub,sup{font-size:75%;line-height:0;position:relative;vertical-align:baseline}sub{bottom:-.25em}sup{top:-.5em}table{text-indent:0;border-color:inherit;border-collapse:collapse}button,input,optgroup,select,textarea{font-family:inherit;font-feature-settings:inherit;font-variation-settings:inherit;font-size:100%;font-weight:inherit;line-height:inherit;color:inherit;margin:0;padding:0}button,select{text-transform:none}[type=button],[type=reset],[type=submit],button{-webkit-appearance:button;background-color:transparent;background-image:none}:-moz-focusring{outline:auto}:-moz-ui-invalid{box-shadow:none}progress{vertical-align:baseline}::-webkit-inner-spin-button,::-webkit-outer-spin-button{height:auto}[type=search]{-webkit-appearance:textfield;outline-offset:-2px}::-webkit-search-decoration{-webkit-appearance:none}::-webkit-file-upload-button{-webkit-appearance:button;font:inherit}summary{display:list-item}blockquote,dd,dl,figure,h1,h2,h3,h4,h5,h6,hr,p,pre{margin:0}fieldset{margin:0;padding:0}legend{padding:0}menu,ol,ul{list-style:none;margin:0;padding:0}dialog{padding:0}textarea{resize:vertical}input::placeholder,textarea::placeholder{opacity:1;color:#9ca3af}[role=button],button{cursor:pointer}:disabled{cursor:default}audio,canvas,embed,iframe,img,object,svg,video{display:block;vertical-align:middle}img,video{max-width:100%;height:auto}[hidden]{display:none}*, ::before, ::after{--tw-border-spacing-x:0;--tw-border-spacing-y:0;--tw-translate-x:0;--tw-translate-y:0;--tw-rotate:0;--tw-skew-x:0;--tw-skew-y:0;--tw-scale-x:1;--tw-scale-y:1;--tw-pan-x: ;--tw-pan-y: ;--tw-pinch-zoom: ;--tw-scroll-snap-strictness:proximity;--tw-gradient-from-position: ;--tw-gradient-via-position: ;--tw-gradient-to-position: ;--tw-ordinal: ;--tw-slashed-zero: ;--tw-numeric-figure: ;--tw-numeric-spacing: ;--tw-numeric-fraction: ;--tw-ring-inset: ;--tw-ring-offset-width:0px;--tw-ring-offset-color:#fff;--tw-ring-color:rgb(59 130 246 / 0.5);--tw-ring-offset-shadow:0 0 #0000;--tw-ring-shadow:0 0 #0000;--tw-shadow:0 0 #0000;--tw-shadow-colored:0 0 #0000;--tw-blur: ;--tw-brightness: ;--tw-contrast: ;--tw-grayscale: ;--tw-hue-rotate: ;--tw-invert: ;--tw-saturate: ;--tw-sepia: ;--tw-drop-shadow: ;--tw-backdrop-blur: ;--tw-backdrop-brightness: ;--tw-backdrop-contrast: ;--tw-backdrop-grayscale: ;--tw-backdrop-hue-rotate: ;--tw-backdrop-invert: ;--tw-backdrop-opacity: ;--tw-backdrop-saturate: ;--tw-backdrop-sepia: }::backdrop{--tw-border-spacing-x:0;--tw-border-spacing-y:0;--tw-translate-x:0;--tw-translate-y:0;--tw-rotate:0;--tw-skew-x:0;--tw-skew-y:0;--tw-scale-x:1;--tw-scale-y:1;--tw-pan-x: ;--tw-pan-y: ;--tw-pinch-zoom: ;--tw-scroll-snap-strictness:proximity;--tw-gradient-from-position: ;--tw-gradient-via-position: ;--tw-gradient-to-position: ;--tw-ordinal: ;--tw-slashed-zero: ;--tw-numeric-figure: ;--tw-numeric-spacing: ;--tw-numeric-fraction: ;--tw-ring-inset: ;--tw-ring-offset-width:0px;--tw-ring-offset-color:#fff;--tw-ring-color:rgb(59 130 246 / 0.5);--tw-ring-offset-shadow:0 0 #0000;--tw-ring-shadow:0 0 #0000;--tw-shadow:0 0 #0000;--tw-shadow-colored:0 0 #0000;--tw-blur: ;--tw-brightness: ;--tw-contrast: ;--tw-grayscale: ;--tw-hue-rotate: ;--tw-invert: ;--tw-saturate: ;--tw-sepia: ;--tw-drop-shadow: ;--tw-backdrop-blur: ;--tw-backdrop-brightness: ;--tw-backdrop-contrast: ;--tw-backdrop-grayscale: ;--tw-backdrop-hue-rotate: ;--tw-backdrop-invert: ;--tw-backdrop-opacity: ;--tw-backdrop-saturate: ;--tw-backdrop-sepia: }.absolute{position:absolute}.relative{position:relative}.-left-20{left:-5rem}.top-0{top:0px}.-bottom-16{bottom:-4rem}.-left-16{left:-4rem}.-mx-3{margin-left:-0.75rem;margin-right:-0.75rem}.mt-4{margin-top:1rem}.mt-6{margin-top:1.5rem}.flex{display:flex}.grid{display:grid}.hidden{display:none}.aspect-video{aspect-ratio:16 / 9}.size-12{width:3rem;height:3rem}.size-5{width:1.25rem;height:1.25rem}.size-6{width:1.5rem;height:1.5rem}.h-12{height:3rem}.h-40{height:10rem}.h-full{height:100%}.min-h-screen{min-height:100vh}.w-full{width:100%}.w-\[calc\(100\%\+8rem\)\]{width:calc(100% + 8rem)}.w-auto{width:auto}.max-w-\[877px\]{max-width:877px}.max-w-2xl{max-width:42rem}.flex-1{flex:1 1 0%}.shrink-0{flex-shrink:0}.grid-cols-2{grid-template-columns:repeat(2, minmax(0, 1fr))}.flex-col{flex-direction:column}.items-start{align-items:flex-start}.items-center{align-items:center}.items-stretch{align-items:stretch}.justify-end{justify-content:flex-end}.justify-center{justify-content:center}.gap-2{gap:0.5rem}.gap-4{gap:1rem}.gap-6{gap:1.5rem}.self-center{align-self:center}.overflow-hidden{overflow:hidden}.rounded-\[10px\]{border-radius:10px}.rounded-full{border-radius:9999px}.rounded-lg{border-radius:0.5rem}.rounded-md{border-radius:0.375rem}.rounded-sm{border-radius:0.125rem}.bg-\[\#FF2D20\]\/10{background-color:rgb(255 45 32 / 0.1)}.bg-white{--tw-bg-opacity:1;background-color:rgb(255 255 255 / var(--tw-bg-opacity))}.bg-gradient-to-b{background-image:linear-gradient(to bottom, var(--tw-gradient-stops))}.from-transparent{--tw-gradient-from:transparent var(--tw-gradient-from-position);--tw-gradient-to:rgb(0 0 0 / 0) var(--tw-gradient-to-position);--tw-gradient-stops:var(--tw-gradient-from), var(--tw-gradient-to)}.via-white{--tw-gradient-to:rgb(255 255 255 / 0)  var(--tw-gradient-to-position);--tw-gradient-stops:var(--tw-gradient-from), #fff var(--tw-gradient-via-position), var(--tw-gradient-to)}.to-white{--tw-gradient-to:#fff var(--tw-gradient-to-position)}.stroke-\[\#FF2D20\]{stroke:#FF2D20}.object-cover{object-fit:cover}.object-top{object-position:top}.p-6{padding:1.5rem}.px-6{padding-left:1.5rem;padding-right:1.5rem}.py-10{padding-top:2.5rem;padding-bottom:2.5rem}.px-3{padding-left:0.75rem;padding-right:0.75rem}.py-16{padding-top:4rem;padding-bottom:4rem}.py-2{padding-top:0.5rem;padding-bottom:0.5rem}.pt-3{padding-top:0.75rem}.text-center{text-align:center}.font-sans{font-family:Figtree, ui-sans-serif, system-ui, sans-serif, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji}.text-sm{font-size:0.875rem;line-height:1.25rem}.text-sm\/relaxed{font-size:0.875rem;line-height:1.625}.text-xl{font-size:1.25rem;line-height:1.75rem}.font-semibold{font-weight:600}.text-black{--tw-text-opacity:1;color:rgb(0 0 0 / var(--tw-text-opacity))}.text-white{--tw-text-opacity:1;color:rgb(255 255 255 / var(--tw-text-opacity))}.underline{-webkit-text-decoration-line:underline;text-decoration-line:underline}.antialiased{-webkit-font-smoothing:antialiased;-moz-osx-font-smoothing:grayscale}.shadow-\[0px_14px_34px_0px_rgba\(0\2c 0\2c 0\2c 0\.08\)\]{--tw-shadow:0px 14px 34px 0px rgba(0,0,0,0.08);--tw-shadow-colored:0px 14px 34px 0px var(--tw-shadow-color);box-shadow:var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow)}.ring-1{--tw-ring-offset-shadow:var(--tw-ring-inset) 0 0 0 var(--tw-ring-offset-width) var(--tw-ring-offset-color);--tw-ring-shadow:var(--tw-ring-inset) 0 0 0 calc(1px + var(--tw-ring-offset-width)) var(--tw-ring-color);box-shadow:var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow, 0 0 #0000)}.ring-transparent{--tw-ring-color:transparent}.ring-white\/\[0\.05\]{--tw-ring-color:rgb(255 255 255 / 0.05)}.drop-shadow-\[0px_4px_34px_rgba\(0\2c 0\2c 0\2c 0\.06\)\]{--tw-drop-shadow:drop-shadow(0px 4px 34px rgba(0,0,0,0.06));filter:var(--tw-blur) var(--tw-brightness) var(--tw-contrast) var(--tw-grayscale) var(--tw-hue-rotate) var(--tw-invert) var(--tw-saturate) var(--tw-sepia) var(--tw-drop-shadow)}.drop-shadow-\[0px_4px_34px_rgba\(0\2c 0\2c 0\2c 0\.25\)\]{--tw-drop-shadow:drop-shadow(0px 4px 34px rgba(0,0,0,0.25));filter:var(--tw-blur) var(--tw-brightness) var(--tw-contrast) var(--tw-grayscale) var(--tw-hue-rotate) var(--tw-invert) var(--tw-saturate) var(--tw-sepia) var(--tw-drop-shadow)}.transition{transition-property:color, background-color, border-color, fill, stroke, opacity, box-shadow, transform, filter, -webkit-text-decoration-color, -webkit-backdrop-filter;transition-property:color, background-color, border-color, text-decoration-color, fill, stroke, opacity, box-shadow, transform, filter, backdrop-filter;transition-property:color, background-color, border-color, text-decoration-color, fill, stroke, opacity, box-shadow, transform, filter, backdrop-filter, -webkit-text-decoration-color, -webkit-backdrop-filter;transition-timing-function:cubic-bezier(0.4, 0, 0.2, 1);transition-duration:150ms}.duration-300{transition-duration:300ms}.selection\:bg-\[\#FF2D20\] *::selection{--tw-bg-opacity:1;background-color:rgb(255 45 32 / var(--tw-bg-opacity))}.selection\:text-white *::selection{--tw-text-opacity:1;color:rgb(255 255 255 / var(--tw-text-opacity))}.selection\:bg-\[\#FF2D20\]::selection{--tw-bg-opacity:1;background-color:rgb(255 45 32 / var(--tw-bg-opacity))}.selection\:text-white::selection{--tw-text-opacity:1;color:rgb(255 255 255 / var(--tw-text-opacity))}.hover\:text-black:hover{--tw-text-opacity:1;color:rgb(0 0 0 / var(--tw-text-opacity))}.hover\:text-black\/70:hover{color:rgb(0 0 0 / 0.7)}.hover\:ring-black\/20:hover{--tw-ring-color:rgb(0 0 0 / 0.2)}.focus\:outline-none:focus{outline:2px solid transparent;outline-offset:2px}.focus-visible\:ring-1:focus-visible{--tw-ring-offset-shadow:var(--tw-ring-inset) 0 0 0 var(--tw-ring-offset-width) var(--tw-ring-offset-color);--tw-ring-shadow:var(--tw-ring-inset) 0 0 0 calc(1px + var(--tw-ring-offset-width)) var(--tw-ring-color);box-shadow:var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow, 0 0 #0000)}.focus-visible\:ring-\[\#FF2D20\]:focus-visible{--tw-ring-opacity:1;--tw-ring-color:rgb(255 45 32 / var(--tw-ring-opacity))}@media (min-width: 640px){.sm\:size-16{width:4rem;height:4rem}.sm\:size-6{width:1.5rem;height:1.5rem}.sm\:pt-5{padding-top:1.25rem}}@media (min-width: 768px){.md\:row-span-3{grid-row:span 3 / span 3}}@media (min-width: 1024px){.lg\:col-start-2{grid-column-start:2}.lg\:h-16{height:4rem}.lg\:max-w-7xl{max-width:80rem}.lg\:grid-cols-3{grid-template-columns:repeat(3, minmax(0, 1fr))}.lg\:grid-cols-2{grid-template-columns:repeat(2, minmax(0, 1fr))}.lg\:flex-col{flex-direction:column}.lg\:items-end{align-items:flex-end}.lg\:justify-center{justify-content:center}.lg\:gap-8{gap:2rem}.lg\:p-10{padding:2.5rem}.lg\:pb-10{padding-bottom:2.5rem}.lg\:pt-0{padding-top:0px}.lg\:text-\[\#FF2D20\]{--tw-text-opacity:1;color:rgb(255 45 32 / var(--tw-text-opacity))}}@media (prefers-color-scheme: dark){.dark\:block{display:block}.dark\:hidden{display:none}.dark\:bg-black{--tw-bg-opacity:1;background-color:rgb(0 0 0 / var(--tw-bg-opacity))}.dark\:bg-zinc-900{--tw-bg-opacity:1;background-color:rgb(24 24 27 / var(--tw-bg-opacity))}.dark\:via-zinc-900{--tw-gradient-to:rgb(24 24 27 / 0)  var(--tw-gradient-to-position);--tw-gradient-stops:var(--tw-gradient-from), #18181b var(--tw-gradient-via-position), var(--tw-gradient-to)}.dark\:to-zinc-900{--tw-gradient-to:#18181b var(--tw-gradient-to-position)}.dark\:text-white\/50{color:rgb(255 255 255 / 0.5)}.dark\:text-white{--tw-text-opacity:1;color:rgb(255 255 255 / var(--tw-text-opacity))}.dark\:text-white\/70{color:rgb(255 255 255 / 0.7)}.dark\:ring-zinc-800{--tw-ring-opacity:1;--tw-ring-color:rgb(39 39 42 / var(--tw-ring-opacity))}.dark\:hover\:text-white:hover{--tw-text-opacity:1;color:rgb(255 255 255 / var(--tw-text-opacity))}.dark\:hover\:text-white\/70:hover{color:rgb(255 255 255 / 0.7)}.dark\:hover\:text-white\/80:hover{color:rgb(255 255 255 / 0.8)}.dark\:hover\:ring-zinc-700:hover{--tw-ring-opacity:1;--tw-ring-color:rgb(63 63 70 / var(--tw-ring-opacity))}.dark\:focus-visible\:ring-\[\#FF2D20\]:focus-visible{--tw-ring-opacity:1;--tw-ring-color:rgb(255 45 32 / var(--tw-ring-opacity))}.dark\:focus-visible\:ring-white:focus-visible{--tw-ring-opacity:1;--tw-ring-color:rgb(255 255 255 / var(--tw-ring-opacity))}}
        </style>
    @endif
</head>
<body class="bg-gray-100 w-full h-screen relative flex flex-col">

    <nav class="flex items-center justify-between flex-wrap bg-red-600 p-6">
        <div class="flex items-center flex-shrink-0 text-white mr-6">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100" width="54" height="54" class="fill-current h-8 w-8 mr-2">
            <!-- Contorno da Pokébola -->
            <circle cx="50" cy="50" r="48" fill="none" stroke="white" stroke-width="4" />
            
            <!-- Linha divisória -->
            <line x1="10" y1="50" x2="90" y2="50" stroke="white" stroke-width="4" />
            
            <!-- Botão central -->
            <circle cx="50" cy="50" r="10" fill="white" />
            <circle cx="50" cy="50" r="6" fill="none" stroke="white" stroke-width="2" />
            </svg>

            <span class="font-semibold text-xl tracking-tight"> @yield('title_nav') </span>
        </div>
        <div class="block lg:flex lg:items-center lg:w-auto">
            <div class="text-sm lg:flex-grow">
                <a href="{{ url('pokemon') }}" class="block mt-4 lg:inline-block lg:mt-0 text-teal-200 hover:text-white mr-4">
                    Pokemon
                </a>
                <a href="{{ url('coaches') }}" class="block mt-4 lg:inline-block lg:mt-0 text-teal-200 hover:text-white mr-4">
                    Coach
                </a>
            </div>
            <div>
            </div>
        </div>
    </nav>

    <div class="h-full flex">
        @yield('content')
    </div>
</body>
</html>
```

- coaches/create.blade.php:

```jsx
@extends('layouts.base')
@section('title', 'Coaches - Create new')
@section('title_nav', 'Coaches - Create new')
@section('content')

    <div class="w-2/3 h-full flex justify-center items-center">
        <div>

            <div class="flex justify-left w-full mb-10">
                <a
                    class="group transition-all relative inline-flex items-center overflow-hidden border border-black px-8 py-3 text-black hover:bg-red-200 focus:outline-none active:text-red-600"
                    href="{{ url('coaches') }}">
                    <span class="absolute -start-full transition-all group-hover:start-4">
                        <svg
                            class="size-5 rtl:rotate-180"
                            xmlns="http://www.w3.org/2000/svg"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke="currentColor">
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                stroke-width="2"
                                d="M7 16l-4-4m0 0l4-4m-4 4h14" />
                        </svg>
                    </span>

                    <span class="text-sm font-medium transition-all group-hover:ms-4"> Back </span>
                </a>
            </div>
            <h1 class="mt-6 text-2xl font-bold text-gray-900 sm:text-3xl md:text-4xl">
                Create and store your Coach! 🧑
            </h1>

            <p class="mt-4 leading-relaxed text-gray-500">
                Create your Trainer, give him a name and image and gotta catch'em all!
            </p>

            <form action="{{ url('coaches') }}" method="POST" enctype="multipart/form-data" class="mx-auto mt-10 w-5/6">
                @csrf
                
                <div class="relative z-0 w-full mb-8 group">
                    <input type="text" name="name" id="name" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                    <label for="name" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Name</label>
                </div>
                <div class="relative z-0 w-full mb-8 group">
                    <input type="file" name="image" id="image" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                    <label for="image" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Image</label>
                </div>
                <div class="text-right mt-10">
                    <button type="submit" class="group relative inline-block focus:outline-none focus:ring">
                        <span
                            class="absolute inset-0 translate-x-1.5 translate-y-1.5 bg-red-200 transition-transform group-hover:translate-x-0 group-hover:translate-y-0"></span>

                        <span
                            class="relative inline-block border-2 border-current px-8 py-3 text-sm font-bold uppercase tracking-widest text-black group-active:text-opacity-75">
                            Create Coach
                        </span>
                    </button>
                </div>
            </form>
        </div>
    </div>

    <div class="w-1/3 h-full">
        <aside class="relative block h-16 lg:order-last lg:col-span-5 lg:h-full xl:col-span-6">
            <img
                alt=""
                src="{{ asset('assets/images/coach_create.png') }}"
                class="absolute inset-0 h-full w-full object-cover" />
        </aside>
    </div>

@endsection

```

coaches/edit.blade.php

```jsx
@extends('layouts.base')
@section('title', 'Coaches - Edit')
@section('title_nav', 'Coaches - Edit')
@section('content')
<div class="flex h-screen w-full">

    <div class="w-1/3">
        <aside class="relative block h-16 lg:order-last lg:col-span-5 lg:h-full xl:col-span-6">
            <img
                alt=""
                src="{{ asset('assets/images/coach_create.png') }}"
                class="absolute inset-0 h-full w-full object-cover" />
        </aside>
    </div>

    <div class="w-2/3 flex justify-center items-center">
        <div>

            <div class="flex justify-left w-full mb-10">
                <a
                    class="group transition-all relative inline-flex items-center overflow-hidden border border-black px-8 py-3 text-black hover:bg-red-200 focus:outline-none active:text-red-600"
                    href="{{ url('coaches') }}">
                    <span class="absolute -start-full transition-all group-hover:start-4">
                        <svg
                            class="size-5 rtl:rotate-180"
                            xmlns="http://www.w3.org/2000/svg"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke="currentColor">
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                stroke-width="2"
                                d="M7 16l-4-4m0 0l4-4m-4 4h14" />
                        </svg>
                    </span>

                    <span class="text-sm font-medium transition-all group-hover:ms-4"> Back </span>
                </a>
            </div>
            <h1 class="mt-6 text-2xl font-bold text-gray-900 sm:text-3xl md:text-4xl">
                Edit and update your Coach 🧑
            </h1>

            <p class="mt-4 leading-relaxed text-gray-500">
                Here you can edit and update your Coach in case you made something wrong!
            </p>

            <form action="{{ url('coaches/' . $coach->id) }}" method="POST" enctype="multipart/form-data" class="mx-auto mt-10 w-5/6">
                @csrf
                @method('PUT')

                <div class="relative z-0 w-full mb-8 group">
                    <input value="{{ $coach->name }}" type="text" name="name" id="name" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                    <label for="name" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Name</label>
                </div>
                <div class="relative z-0 w-full h-32 mb-8 group flex">
                    <img src="{{ asset($coach->image) }}" alt="{{ $coach->name }}" class="object-cover h-16 pr-2">
                    <div class="relative z-0 w-full h-48 mb-8 group">
                        <input type="file" name="image" id="image" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " />
                        <label for="image" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Image</label>

                    </div>
                </div>
                <div class="text-right mt-10">
                    <button type="submit" class="group relative inline-block focus:outline-none focus:ring">
                        <span
                            class="absolute inset-0 translate-x-1.5 translate-y-1.5 bg-red-200 transition-transform group-hover:translate-x-0 group-hover:translate-y-0"></span>

                        <span
                            class="relative inline-block border-2 border-current px-8 py-3 text-sm font-bold uppercase tracking-widest text-black group-active:text-opacity-75">
                            Update Coach
                        </span>
                    </button>
                </div>
            </form>
        </div>
    </div>

</div>
@endsection
```

coaches/index.blade.php:

```jsx
@extends('layouts.base')
@section('title', 'Coaches - List All')
@section('title_nav', 'Coaches')

@section('content')

<div class="absolute bottom-10 right-10">
    <a
        class="inline-block rounded-full bg-red-600 px-3 py-3 text-sm font-medium text-white transition hover:scale-110 hover:shadow-xl focus:outline-none active:bg-red-500"
        href="{{url('coaches/create')}}"
        title="Create new Coach">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24" fill="currentColor">
            <path d="M12 5c.55 0 1 .45 1 1v5h5c.55 0 1 .45 1 1s-.45 1-1 1h-5v5c0 .55-.45 1-1 1s-1-.45-1-1v-5H6c-.55 0-1-.45-1-1s.45-1 1-1h5V6c0-.55.45-1 1-1z" />
        </svg>
    </a>
</div>

<div class="w-full grid grid-cols-4 auto-rows-[200px] gap-10 mt-12 mx-12">

    @foreach($coaches as $entity)
    <div class="h-48 relative flex flex-col items-center bg-white border border-gray-200 rounded-lg shadow md:flex-row md:max-w-md hover:bg-gray-100">
        <div class="absolute top-4 right-5">
            <p class="mb-3 font-normal text-gray-700"> {{ $entity->id }} </p>
        </div>
        <img class="object-cover rounded-t-lg w-1/3 h-32 md:h-48 md:w-48 md:rounded-none md:rounded-l-lg" src="{{ asset($entity->image) }}" alt="{{ $entity->name }}">

        <div class="flex flex-col justify-between p-4 leading-normal w-2/3">
            <h5 class="mb-2 text-2xl font-bold tracking-tight text-gray-900">{{ $entity->name }}</h5>

            <div class="text-right">
                <span class="inline-flex mt-2 -space-x-px overflow-hidden rounded-md border border-gray-900 shadow-sm">
                    <a href="{{ url('coaches/'.$entity->id.'/edit') }}" class="inline-block px-4 py-2 text-sm font-medium text-gray-900 hover:bg-gray-600 hover:text-white focus:relative">Edit</a>

                    <form action="{{ url('coaches/' . $entity->id) }}" method="POST" class="inline-block text-sm font-medium text-red-500 hover:bg-red-500 hover:text-white focus:relative">
                        @csrf
                        @method('DELETE')
                        <button class="px-4 py-2" type="submit">Delete</button>
                    </form>
                </span>
            </div>
        </div>
    </div>
    @endforeach

</div>

@endsection
```

pokemon/create.blade.php

```jsx
@extends('layouts.base')
@section('title', 'Pokemon - Create new')
@section('title_nav', 'Pokemon - Create new')

@section('content')

<div class="w-2/3 h-full flex justify-center items-center">
    <div>

        <div class="flex justify-left w-full mb-10">
            <a
                class="group transition-all relative inline-flex items-center overflow-hidden border border-black px-8 py-3 text-black hover:bg-red-200 focus:outline-none active:text-red-600"
                href="{{ url('pokemon') }}">
                <span class="absolute -start-full transition-all group-hover:start-4">
                    <svg
                        class="size-5 rtl:rotate-180"
                        xmlns="http://www.w3.org/2000/svg"
                        fill="none"
                        viewBox="0 0 24 24"
                        stroke="currentColor">
                        <path
                            stroke-linecap="round"
                            stroke-linejoin="round"
                            stroke-width="2"
                            d="M7 16l-4-4m0 0l4-4m-4 4h14" />
                    </svg>
                </span>

                <span class="text-sm font-medium transition-all group-hover:ms-4"> Back </span>
            </a>
        </div>
        <h1 class="mt-6 text-2xl font-bold text-gray-900 sm:text-3xl md:text-4xl">
            Create and store your Pokemon! 👹
        </h1>

        <p class="mt-4 leading-relaxed text-gray-500">
            Create your Pokémon, choose its type and Power Points, and get ready for adventures and battles!
        </p>

        <form action="{{ url('pokemon') }}" method="POST" enctype="multipart/form-data" class="mx-auto mt-10 w-5/6">
            @csrf

            <div class="relative z-0 w-full mb-8 group">
                <input type="text" name="name" id="name" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                <label for="name" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Name</label>
            </div>
            <div class="grid md:grid-cols-2 md:gap-6">
                <div class="relative z-0 w-full mb-5 group">
                    <input type="text" name="type" id="type" class="block py-2.5 px-0 w-full text-sm text-gray-900 bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                    <label for="type" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Type</label>
                </div>
                <div class="relative z-0 w-full mb-5 group">
                    <input type="number" name="power" id="power" min="0" class="block py-2.5 px-0 w-full text-sm text-gray-900 bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                    <label for="power" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Power</label>
                </div>
            </div>
            <div class="relative z-0 w-full mb-8 group">
                <input type="file" name="image" id="image" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                <label for="image" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Image</label>
            </div>

            <div class="relative z-0 w-full mb-8 group">
                <select name="coach_id" id="coach_id" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" required>
                    <option value="" selected disabled hidden>Please select a coach</option>
                    @foreach ($coaches as $coach)
                    <option value="{{ $coach->id }}"> {{ $coach->name }} </option>
                    @endforeach
                </select>
                <label for="coach_id" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Name</label>
            </div>
            <div class="text-right mt-10">
                <button type="submit" class="group relative inline-block focus:outline-none focus:ring">
                    <span
                        class="absolute inset-0 translate-x-1.5 translate-y-1.5 bg-red-200 transition-transform group-hover:translate-x-0 group-hover:translate-y-0"></span>

                    <span
                        class="relative inline-block border-2 border-current px-8 py-3 text-sm font-bold uppercase tracking-widest text-black group-active:text-opacity-75">
                        Create Pokemon
                    </span>
                </button>
            </div>
        </form>
    </div>
</div>

<div class="w-1/3 h-full">
    <aside class="relative block h-16 lg:order-last lg:col-span-5 lg:h-full xl:col-span-6">
        <img
            alt=""
            src="{{ asset('assets/images/pokemon_create.jpg') }}"
            class="absolute inset-0 h-full w-full object-cover" />
    </aside>
</div>

@endsection
```

pokemon/edit.blade.php:

```jsx
@extends('layouts.base')
@section('title', 'Pokemon - Edit')
@section('title_nav', 'Pokemon - Edit')
@section('content')
<div class="flex h-screen w-full">

    <div class="w-1/3">
        <aside class="relative block h-16 lg:order-last lg:col-span-5 lg:h-full xl:col-span-6">
            <img
                alt=""
                src="{{ asset('assets/images/pokemon_create.jpg') }}"
                class="absolute inset-0 h-full w-full object-cover" />
        </aside>
    </div>

    <div class="w-2/3 flex justify-center items-center">
        <div>

            <div class="flex justify-left w-full mb-10">
                <a
                    class="group transition-all relative inline-flex items-center overflow-hidden border border-black px-8 py-3 text-black hover:bg-red-200 focus:outline-none active:text-red-600"
                    href="{{ url('pokemon') }}">
                    <span class="absolute -start-full transition-all group-hover:start-4">
                        <svg
                            class="size-5 rtl:rotate-180"
                            xmlns="http://www.w3.org/2000/svg"
                            fill="none"
                            viewBox="0 0 24 24"
                            stroke="currentColor">
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                stroke-width="2"
                                d="M7 16l-4-4m0 0l4-4m-4 4h14" />
                        </svg>
                    </span>

                    <span class="text-sm font-medium transition-all group-hover:ms-4"> Back </span>
                </a>
            </div>
            <h1 class="mt-6 text-2xl font-bold text-gray-900 sm:text-3xl md:text-4xl">
                Edit and update your Pokemon 👹
            </h1>

            <p class="mt-4 leading-relaxed text-gray-500">
                Here you can edit and update your Pokemon in case you made something wrong!
            </p>

            <form action="{{ url('pokemon/' . $pokemon->id) }}" method="POST" enctype="multipart/form-data" class="mx-auto mt-10 w-5/6">
                @csrf
                @method('PUT')

                <div class="relative z-0 w-full mb-8 group">
                    <input value="{{ $pokemon->name }}" type="text" name="name" id="name" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                    <label for="name" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Name</label>
                </div>
                <div class="grid md:grid-cols-2 md:gap-6">
                    <div class="relative z-0 w-full mb-5 group">
                        <input value="{{ $pokemon->type }}" type="text" name="type" id="type" class="block py-2.5 px-0 w-full text-sm text-gray-900 bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                        <label for="type" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Type</label>
                    </div>
                    <div class="relative z-0 w-full mb-5 group">
                        <input value="{{ $pokemon->power }}" type="number" name="power" id="power" min="0" class="block py-2.5 px-0 w-full text-sm text-gray-900 bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " required />
                        <label for="power" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Power</label>
                    </div>
                </div>
                <div class="relative z-0 w-full mb-2 group flex">
                    <img src="{{ asset($pokemon->image) }}" alt="{{ $pokemon->name }}" class="object-cover h-16 pr-2">
                    <div class="relative z-0 w-full mb-2 group">
                        <input type="file" name="image" id="image" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" placeholder=" " />
                        <label for="image" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Image</label>

                    </div>
                </div>

                <div class="relative z-0 w-full mb-8 group">
                    <select name="coach_id" id="coach_id" class="block py-2.5 px-0 w-full text-sm text-black bg-transparent border-0 border-b-2 border-gray-300 appearance-none focus:outline-none focus:ring-0 focus:border-red-600 peer" required>
                    @foreach ($coaches as $coach)
                        @if($pokemon->coach->id === $coach->id)
                            <option value="{{ $coach->id }}" selected> {{ $coach->name }} </option>
                        @else
                            <option value="{{ $coach->id }}"> {{ $coach->name }} </option>
                        @endif
                    @endforeach
                    </select>
                    <label for="coach_id" class="peer-focus:font-medium absolute text-sm text-gray-500 duration-300 transform -translate-y-6 scale-75 top-3 -z-10 origin-[0] peer-focus:start-0 rtl:peer-focus:translate-x-1/4 rtl:peer-focus:left-auto peer-focus:text-red-600 peer-placeholder-shown:scale-100 peer-placeholder-shown:translate-y-0 peer-focus:scale-75 peer-focus:-translate-y-6">Name</label>
                </div>
                <div class="text-right mt-10">
                    <button type="submit" class="group relative inline-block focus:outline-none focus:ring">
                        <span
                            class="absolute inset-0 translate-x-1.5 translate-y-1.5 bg-red-200 transition-transform group-hover:translate-x-0 group-hover:translate-y-0"></span>

                        <span
                            class="relative inline-block border-2 border-current px-8 py-3 text-sm font-bold uppercase tracking-widest text-black group-active:text-opacity-75">
                            Update Pokemon
                        </span>
                    </button>
                </div>
            </form>
        </div>
    </div>

</div>
@endsection
```

pokemon/index.blade.php:

```jsx
@extends('layouts.base')
@section('title', 'Pokemon - List all')
@section('title_nav', 'Pokemon')
@section('content')

@can('create', App\Models\Pokemon::class)
<div class="absolute bottom-10 right-10">
    <a
        class="inline-block rounded-full bg-red-600 px-3 py-3 text-sm font-medium text-white transition hover:scale-110 hover:shadow-xl focus:outline-none active:bg-red-500"
        href="{{url('pokemon/create')}}"
        title="Create new Pokemon">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24" fill="currentColor">
            <path d="M12 5c.55 0 1 .45 1 1v5h5c.55 0 1 .45 1 1s-.45 1-1 1h-5v5c0 .55-.45 1-1 1s-1-.45-1-1v-5H6c-.55 0-1-.45-1-1s.45-1 1-1h5V6c0-.55.45-1 1-1z" />
        </svg>
    </a>
</div>
@endcan

<div class="w-full grid grid-cols-4 auto-rows-[200px] gap-10 mt-12 mx-12">

    @foreach($pokemon as $entity)
    <div class="h-48 relative flex flex-col items-center bg-white border border-gray-200 rounded-lg shadow md:flex-row md:max-w-md hover:bg-gray-100">
        <div class="absolute top-4 right-5">
            <p class="mb-3 font-normal text-gray-700"> {{ $entity->id }} </p>
        </div>
        <img class="object-cover rounded-t-lg w-1/3 h-32 md:h-48 md:w-48 md:rounded-none md:rounded-l-lg" src="{{ asset($entity->image) }}" alt="{{ $entity->name }}">

        <div class="flex flex-col justify-between p-4 leading-normal w-2/3">
            <h5 class="mb-2 text-2xl font-bold tracking-tight text-gray-900">{{ $entity->name }}</h5>
            <p class="mb-3 font-normal text-gray-700">{{ $entity->power }} PP</p>
            @if(isset($entity->coach))
                <p class="mb-3 font-normal text-gray-700">{{ $entity->coach->name }}</p>
            @else
                <p class="mb-3 font-normal text-gray-700">No coach</p>
            @endif
            <div class="text-right">
                <span class="inline-flex mt-2 -space-x-px overflow-hidden rounded-md border border-gray-900 shadow-sm">
                    <a href="{{ url('pokemon/'.$entity->id.'/edit') }}" class="inline-block px-4 py-2 text-sm font-medium text-gray-900 hover:bg-gray-600 hover:text-white focus:relative">Edit</a>

                    <form action="{{ url('pokemon/' . $entity->id) }}" method="POST" class="inline-block text-sm font-medium text-red-500 hover:bg-red-500 hover:text-white focus:relative">
                        @csrf
                        @method('DELETE')
                        <button class="px-4 py-2" type="submit">Delete</button>
                    </form>
                </span>
            </div>
        </div>
    </div>
    @endforeach

</div>

@endsection
```

26 - Refresh, buildar e rodar:

`php artisan migrate:fresh`

`npm run build`

`php artisan serve`