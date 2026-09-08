# 🏛️ Dictadura de Arquitectura de Software (Laravel)

Este documento contiene el conjunto de reglas, estándares y buenas prácticas obligatorias para el desarrollo de aplicaciones Laravel en la materia de Arquitectura de Software. El incumplimiento de estas normas conlleva penalizaciones en la evaluación.

---

## 📑 Tabla de Contenidos
1. [Reglas para Rutas](#1-reglas-para-rutas-routeswebphp)
2. [Reglas para Controladores](#2-reglas-para-controladores)
3. [Reglas para Modelos y Base de Datos](#3-reglas-para-modelos-y-base-de-datos)
4. [Reglas para Vistas (Blade)](#4-reglas-para-vistas-blade)
5. [Ordenamiento Estricto dentro del Modelo](#5-ordenamiento-estricto-dentro-del-modelo)
6. [Ejemplo Práctico Completo: Entidad `Cat`](#6-ejemplo-práctico-completo-entidad-cat)
7. [Checklist Rápidio para Evaluaciones](#7-checklist-rápido-para-evaluaciones)

---

## 1. 🔀 Reglas para Rutas (`routes/web.php`)

* **Responsabilidad Única:** Toda ruta debe estar obligatoriamente asociada a un método de un controlador. Queda estrictamente prohibido incluir lógica de negocio, closures (`function() {}`) o validaciones dentro del archivo de rutas.
* **Formato String-Based Reference:** Se debe utilizar el formato de referencia basado en String para definir las rutas (ej. `'App\Http\Controllers\CatController@index'` o la sintaxis acordada en la dictadura).
* **Controladores Separados por Dominio:** No manejes la ruta raíz (`/`) ni la vista `home` desde un controlador específico de entidad (como `CatController` o `ProductController`). Las vistas generales deben ser gestionadas por un controlador dedicado (ej. `HomeController`).

---

## 2. 🎮 Reglas para Controladores

* **Cero Validaciones:** Prohibido realizar validaciones dentro de los métodos del controlador. Se deben utilizar **Form Requests** (`php artisan make:request CatRequest`) para mantener el código DRY y desacoplado.
* **Tipado Estricto de Parámetros y Retornos:** Es obligatorio tipar todos los parámetros de entrada y especificar el tipo de retorno después de la función:
  ```php
  public function show(string $id): View
  public function store(CatRequest $request): RedirectResponse
  ```
* **Cero "Route Model Binding":** Prohibido inyectar el modelo directamente en la firma del método (ej. `public function show(Cat $cat)`). Se debe recibir únicamente el identificador como `$id` (`string` o `int`) y realizar la búsqueda dentro del método.
* **Paso de Datos a Vistas mediante `$viewData`:**
  * Usar **única y exclusivamente** un arreglo asociativo llamado `$viewData`.
  * Queda **estrictamente prohibido** el uso de la función `compact()`, el método `with()` explícito para variables individuales, o nombres genéricos como `$data`.
  ```php
  $viewData = [];
  $viewData['title'] = 'Cat Details';
  $viewData['cat'] = $cat;
  return view('cat.show')->with('viewData', $viewData);
  ```
* **Idioma 100% Inglés:** Nombres de variables, métodos, clases y comentarios deben estar redactados en inglés sin combinar con español.
* **Prohibido Código de Depuración:** No dejar instrucciones `dd()`, `die()` o `echo` en el código entregado.

---

## 3. 📦 Reglas para Modelos y Base de Datos

* **DocBlock de Atributos:** Todo modelo debe incluir un comentario inicial detallando cada uno de sus atributos, su tipo de dato y descripción:
  ```php
  /**
   * CAT ATTRIBUTES
   * $this->attributes['id'] - int - primary key
   * $this->attributes['name'] - string - name of the cat
   * $this->attributes['age'] - int - age of the cat in years
   * $this->attributes['created_at'] - string - creation timestamp
   * $this->attributes['updated_at'] - string - update timestamp
   */
  ```
* **Encapsulamiento Estricto (Getters y Setters):**
  * Todos los atributos deben accederse y modificarse mediante métodos explícitos.
  * Prohibido acceder a propiedades directamente en cualquier parte del código (ej. `$cat->name` es incorrecto; debe usarse `$cat->getName()`).
* **Factories y Seeders:**
  * Si un modelo incluye el trait `use HasFactory;`, es **obligatorio** que el archivo Factory correspondiente exista en `database/factories`.
  * La carga masiva de datos iniciales debe realizarse mediante Seeders.
* **Estándares de Base de Datos:**
  * Nombres de tablas siempre en plural (`cats`).
  * Columnas en `snake_case` (ej. `created_at`).
  * Migraciones realizadas mediante el ORM Eloquent, evitando sentencias SQL crudas.

---

## 4. 🎨 Reglas para Vistas (Blade)

* **Extensión de Layout Base:** Toda vista debe extender de un layout principal (ej. `layouts.app`).
* **Uso de Getters en Blade:** Al renderizar datos de un modelo en una plantilla Blade, es obligatorio invocar los métodos getters:
  ```blade
  <!-- INCORRECTO -->
  <p>{{ $viewData['cat']->name }}</p>

  <!-- CORRECTO -->
  <p>{{ $viewData['cat']->getName() }}</p>
  ```
* **Prohibido PHP Puro e Inline CSS:**
  * No abrir/cerrar etiquetas `<?php ?>`. Usar únicamente directivas de Blade (`@foreach`, `@if`, etc.).
  * No incluir estilos CSS con la etiqueta `<style>` dentro de las vistas Blade.
* **Internacionalización (Lang):** No colocar textos fijos ("quemados") en las plantillas HTML. Utilizar los archivos de traducción en `resources/lang/` mediante `__('messages.key')` o `@lang('messages.key')`.
* **Limpieza:** Eliminar archivos de vistas por defecto que no se utilicen (ej. `welcome.blade.php`).

---

## 5. 🧱 Ordenamiento Estricto dentro del Modelo

Los métodos y propiedades dentro de una clase Modelo deben organizarse en el siguiente orden:

1. **Definición de Atributos Protegidos:** `$fillable`, `$casts`, etc.
2. **Bloque de Setters Primitivos:** Todos los métodos mutadores (`setId()`, `setName()`, etc.).
3. **Bloque de Getters Primitivos:** Todos los métodos accesores (`getId()`, `getName()`, etc.).
4. **Bloque de Métodos No Primitivos y Relaciones:** Consultas avanzadas, lógica de negocio y relaciones Eloquent.

---

## 6. 🐈 Ejemplo Práctico Completo: Entidad `Cat`

### A. Modelo (`app/Models/Cat.php`)
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Collection;

/**
 * CAT ATTRIBUTES
 * $this->attributes['id'] - int - primary key
 * $this->attributes['name'] - string - name of the cat
 * $this->attributes['age'] - int - age of the cat in years
 * $this->attributes['created_at'] - string - creation timestamp
 * $this->attributes['updated_at'] - string - update timestamp
 */
class Cat extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'age'];

    // --- SETTERS ---
    public function setId(int $id): void
    {
        $this->attributes['id'] = $id;
    }

    public function setName(string $name): void
    {
        $this->attributes['name'] = $name;
    }

    public function setAge(int $age): void
    {
        $this->attributes['age'] = $age;
    }

    // --- GETTERS ---
    public function getId(): int
    {
        return $this->attributes['id'];
    }

    public function getName(): string
    {
        return $this->attributes['name'];
    }

    public function getAge(): int
    {
        return $this->attributes['age'];
    }

    // --- NON-PRIMITIVE METHODS / RELATIONS ---
    public static function getTop3Oldest(): Collection
    {
        return self::orderBy('age', 'desc')->take(3)->get();
    }
}
```

### B. Form Request (`app/Http/Requests/CatRequest.php`)
```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class CatRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name' => 'required|string|max:255',
            'age' => 'required|integer|min:0',
        ];
    }
}
```

### C. Controlador (`app/Http/Controllers/CatController.php`)
```php
<?php

namespace App\Http\Controllers;

use App\Models\Cat;
use App\Http\Requests\CatRequest;
use Illuminate\View\View;
use Illuminate\Http\RedirectResponse;

class CatController extends Controller
{
    public function index(): View
    {
        $viewData = [];
        $viewData['title'] = 'Cats List';
        $viewData['cats'] = Cat::all();

        return view('cat.index')->with('viewData', $viewData);
    }

    public function create(): View
    {
        $viewData = [];
        $viewData['title'] = 'Create Cat';

        return view('cat.create')->with('viewData', $viewData);
    }

    public function store(CatRequest $request): RedirectResponse
    {
        Cat::create($request->validated());

        return redirect()->route('cats.index');
    }

    public function show(string $id): View
    {
        $viewData = [];
        $cat = Cat::findOrFail($id);
        $viewData['title'] = $cat->getName() . ' Details';
        $viewData['cat'] = $cat;

        return view('cat.show')->with('viewData', $viewData);
    }

    public function topOldest(): View
    {
        $viewData = [];
        $viewData['title'] = 'Top 3 Oldest Cats';
        $viewData['cats'] = Cat::getTop3Oldest();

        return view('cat.top_oldest')->with('viewData', $viewData);
    }
}
```

### D. Vista (`resources/views/cat/index.blade.php`)
```blade
@extends('layouts.app')

@section('title', $viewData['title'])

@section('content')
<h1>{{ $viewData['title'] }}</h1>

<ul>
    @foreach ($viewData['cats'] as $cat)
        <li>
            <a href="{{ route('cats.show', ['id' => $cat->getId()]) }}">
                {{ $cat->getName() }} - {{ $cat->getAge() }} years old
            </a>
        </li>
    @endforeach
</ul>
@endsection
```

---

## 7. ✅ Checklist Rápido para Evaluaciones

Antes de entregar tu proyecto o parcial, verifica:

- [ ] ¿Toda ruta invoca un controlador y no hay lógica en `routes/web.php`?
- [ ] ¿Se eliminó cualquier validación dentro del Controlador y se usó Form Request?
- [ ] ¿Se especificaron los tipos de retorno en todos los métodos de los controladores?
- [ ] ¿Se evitó el Route Model Binding en los parámetros del controlador?
- [ ] ¿Se enviaron todos los datos a la vista usando exactamente la variable `$viewData`?
- [ ] ¿El modelo cuenta con el comentario DocBlock con la lista de atributos?
- [ ] ¿Se definieron y usaron getters/setters tanto en el código PHP como en las vistas Blade?
- [ ] ¿Los métodos del modelo cumplen con el orden estricto (Setters -> Getters -> No Primitivos)?
- [ ] Si el modelo usa `HasFactory`, ¿existe el archivo correspondiente en `database/factories`?
- [ ] ¿Todas las vistas extienden de un layout base y carecen de bloques `<style>` o etiquetas PHP puras?

