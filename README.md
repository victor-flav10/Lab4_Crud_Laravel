# Lab4_Crud_Laravel

### Este repositorio contiene la documentación técnica y el código fuente del laboratorio desarrollado en Laravel, implementando un sistema CRUD (Create, Read, Update, Delete) conectado a una base de datos relacional MySQL gestionada mediante WampServer.
---

# 👤 Datos Personales

    Nombre: Víctor Flavio

    Universidad: Universidad Tecnológica de Panamá (UTP)

    Entorno de Trabajo: Windows 11, WampServer (MySQL), PHP 8.5, Laravel 13, Visual Studio Code, Node.js / Vite.

# Introducción del Laboratorio

### El propósito de este laboratorio es la construcción práctica de una aplicación web basada en el framework Laravel para la gestión de productos (Products). El proceso abarca desde la inicialización del proyecto y la configuración del entorno local hasta la resolución de conflictos de compatibilidad en bases de datos, la gestión de migraciones, la implementación de modelos con asignación masiva, validaciones mediante peticiones personalizadas (FormRequest), controladores con recursos completos y vistas dinámicas apoyadas por plantillas Blade y Bootstrap.
---

# Paso 1: Creación e Inicialización del Proyecto

Se configuró el entorno local dentro del directorio raíz del servidor web de WampServer utilizando el instalador de Composer para Laravel.

* Comandos ejecutados:
<br></br>

    ```

    cd C:\wamp64\www
    composer create-project laravel/laravel Laboratorio_CrudLaravel
    cd Laboratorio_CrudLaravel`
* Qué hice: Descarge e instalé la estructura de archivos estándar de Laravel, estableciendo la base del proyecto en la ruta <i>`C:\wamp64\www\Laboratorio_CrudLaravel.`</i>

# Paso 2: Configuración del Entorno y Conexión a MySQL (.env)

Se estableció la conexión entre el framework y el servidor de base de datos MySQL ejecutado por WampServer.
Configuración aplicada en el archivo .env:
    Fragmento de código

    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=laboratorio_crudlaravel
    DB_USERNAME=root
    DB_PASSWORD= ```

* Qué se hizo: Se definieron los parámetros de conexión apuntando a la base de datos laboratorio_crudlaravel creada previamente desde el panel de phpMyAdmin, utilizando el usuario administrador por defecto sin contraseña.

# Paso 3: Ajustes de Compatibilidad de Índices (AppServiceProvider)

Para evitar el error común de restricciones de longitud en los índices de claves primarias y foráneas de las tablas del sistema de Laravel (SQLSTATE[42000]: 1071 Specified key was too long; max key length is 1000 bytes), se configuró el proveedor principal de servicios de la aplicación.
Código implementado en app/Providers/AppServiceProvider.php:
```
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Schema;

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
        Schema::defaultStringLength(191);
    }
}
```
* Comandos de limpieza ejecutados:
```
php artisan config:clear
php artisan migrate:fresh
```

* Qué se hizo: Se estableció un límite de longitud predeterminado de 191 caracteres para las columnas de tipo cadena (string), se limpió la caché de configuración del framework y se ejecutaron las migraciones iniciales para verificar la correcta creación de las tablas base (users, cache, jobs).

# Paso 4: Creación del Modelo y Migración de Productos
Se generó el modelo Product junto con su migración para estructurar los campos de la tabla en la base de datos de MySQL.
 Comando ejecutado:`<i>Bashphp artisan make:model Product -m</i>`
<br></br>
```
Código de la migración (database/migrations/..._create_products_table.php):PHPSchema::create('products', function (Blueprint $table) {<br></br>
    $table->id();<br></br>
    $table->string('description');<br></br>
    $table->decimal('price', 8, 2);<br></br>
    $table->integer('stock');<br></br>
    $table->timestamps();<br></br>
});
```
* Configuración del Modelo (app/Models/Product.php):<br></br>
Se añadió la importación del rasgo HasFactory y la propiedad $fillable para permitir la asignación masiva de datos:   PHPnamespace App\Models;
```
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    use HasFactory;

    protected $fillable = [
        'description',
        'price',
        'stock'
    ];
}
```
* Qué se hizo: Se definió la estructura relacional de la tabla de productos y se habilitó el modelo para recibir los datos enviados desde los formularios web de forma segura.

# Paso 5: Configuración de Reglas de Validación (ProductRequest)
Para garantizar la integridad de los datos ingresados por el usuario, se configuraron las reglas de validación en la clase de petición dedicada.Código implementado en <i>`app/Http/Requests/ProductRequest.php:PHPpublic function rules(): array`</i>
```
{
    return [
        'description' => 'required|string|max:255',
        'price' => 'required|numeric',
        'stock' => 'required|integer',
    ];
}
```
* Qué se hizo: Se establecieron restricciones obligatorias (required), tipos de datos numéricos y de texto para evitar inserciones vacías o corruptas en la base de datos de WampServer.

# Paso 6: Implementación del Controlador (ProductController)
Se configuraron las acciones lógicas del CRUD (Index, Create, Store, Show, Edit, Update, Destroy) para interactuar entre el modelo y las vistas.<br></br>
Fragmento clave del método store en app/Http/Controllers/ProductController.php:PHPpublic function store(ProductRequest $request): RedirectResponse
```
{
    Product::create($request->validated());

    return Redirect::route('products.index')
        ->with('success', 'Product created successfully.');
}
```
* Qué se hizo: Se conectó la validación del formulario con el método de creación masiva del modelo y se estructuró la redirección automática con mensajes de éxito.

# Paso 7: Compilación de Assets con Vite y Pruebas en InterfazSe compilaron los recursos visuales del sistema mediante Node.js para que las vistas con plantillas Blade y Bootstrap cargaran correctamente sin errores de manifiesto.   
* Comandos ejecutados:
```
Bashnpm install
npm run build <br></br>
```
* Ejecución del servidor de desarrollo:<i>`Bashphp artisan serve`</i>
* Qué se hizo: Se levantó el entorno local y se accedió a la ruta [http://127.0.0.1:8000/products](http://127.0.0.1:8000/products) para validar visualmente la interfaz gráfica del CRUD.
---

# Problemas Vistos y Cómo Se Resolvieron
Durante el proceso de desarrollo del laboratorio, se documentaron y solucionaron los siguientes escenarios técnicos:<br></br>
### Trait HasFactory no encontrado:
  * Causa: El archivo del modelo carecía de la importación de su espacio de nombres correspondiente.
  * Solución: Se agregó la línea use Illuminate\Database\Eloquent\Factories\HasFactory; al inicio de Product.php.<br></br>
### Excepción ViteManifestNotFoundException:
  * Causa: El sistema buscaba el archivo manifest.json compilado en la carpeta public/build y el entorno de assets estaba vacío.
  * Solución: Se ejecutaron los comandos npm install y npm run build en la terminal para generar los recursos visuales requeridos por las vistas.<br></br>
### Error SQL Field 'description' doesn't have a default value:
  * Causa: El formulario HTML no tenía correctamente definidos los atributos name o el archivo ProductRequest devolvía un arreglo vacío, enviando peticiones sin datos a la base de datos.
  * Solución: Se estructuraron los inputs con los nombres exactos en inglés (description, price, stock) dentro de form.blade.php y se limpió la duplicidad de la función rules() en el archivo de validación.
---
# Bibliografía y Referencias

* __Documentación Oficial de Laravel: Laravel 11/13 Documentation - Eloquent, Validation, and Routing.__

* __Guías de WampServer y MySQL: Configuración de entornos locales para desarrollo web PHP y gestión de bases de datos relacionales.__

* __Paquete CRUD Generator: ibex/crud-generator utilizado para el scaffolding automatizado de rutas, controladores y vistas Blade.__

---
# Ilustraciones/Imagenes
<img width="530" height="377" alt="Screenshot 2026-11-06 185641" src="https://github.com/user-attachments/assets/3017fbe4-c067-4e9b-89b1-a82cf4595bbd" />
<i>img#1:Creación del laboratorio de Crud-Laravel</i> <br></br>

<img width="430" height="530" alt="Screenshot 2026-11-06 185727" src="https://github.com/user-attachments/assets/49158e5e-1925-48af-898f-f8e2c8c7970a" />
<i>img#2:Configuración de la Base de Datos</i> <br></br>

<img width="1130" height="693" alt="Screenshot 2026-11-06 185936" src="https://github.com/user-attachments/assets/87d6946f-0796-4b01-bf9f-bb200dcb3fff" /><br></br>
<img width="1160" height="577" alt="Screenshot 2026-11-06 190200" src="https://github.com/user-attachments/assets/6c0f6969-1be4-455e-8807-c36dda6058dd" /><br></br>
<img width="831" height="412" alt="Screenshot 2026-11-06 190929" src="https://github.com/user-attachments/assets/13e123c1-6401-48e0-b042-71cd3d294689" /><br></br>
<img width="680" height="592" alt="Screenshot 2026-11-06 191146" src="https://github.com/user-attachments/assets/4aa0fffb-2468-4069-8b83-cdc7a9f87101" /><br></br>
<img width="822" height="153" alt="Screenshot 2026-11-06 191206" src="https://github.com/user-attachments/assets/bd2222f5-2dc0-481e-9c68-7c9acba710d1" /><br></br>
<img width="822" height="420" alt="Screenshot 2026-11-06 192123" src="https://github.com/user-attachments/assets/e33bf105-1691-4d00-82f7-7a269d3f44cd" /><br></br>
<img width="817" height="697" alt="Screenshot 2026-11-06 192230" src="https://github.com/user-attachments/assets/ff203ace-e1b5-4320-911e-d016ad4477bb" /><br></br>
<i>img#3, 4, 5, 6, 7, 8, 9:Comandos, Codigos y Archivos modificados durante el Laboratorio</i> <br></br>

<img width="1520" height="410" alt="Screenshot 2026-11-07 032052" src="https://github.com/user-attachments/assets/ea273470-0caf-4a1c-9c93-3f80a445ef78" />
<img width="1520" height="255" alt="Screenshot 2026-11-07 032323" src="https://github.com/user-attachments/assets/e8f6ec1b-3773-4810-b163-1fb205201342" />
<img width="1526" height="158" alt="Screenshot 2026-11-07 033122" src="https://github.com/user-attachments/assets/3e96085c-8a8d-451f-ad09-7633f0091082" />
<i>img#10, 11, 12:Edición de la Base de Datos desde la pagina web</i> <br></br>

<img width="927" height="576" alt="Screenshot 2026-11-07 033136" src="https://github.com/user-attachments/assets/31eecdc0-4086-4d57-8e64-0edcb5b2c3ab" />
<i>img#13:Datos almacenados e las tablas de la BD</i> <br></br>
