# Laravel

## Migraciones
Las migraciones sirven para crear la base de datos usando codigo php, revisar la seccion de `Diseno de la base de datos`.

Crear una migracion sin modelo muy util para las tablas intermedias ejemplo para la tabla `post_tag`, al crearlo con esta convencion se crea automaticamente el metodo `up` y `down`
```php
php atisan make:migration create_post_tag_table
```
Ejecutar las migraciones
```php
php arisan migrate
```
Ejecutar las migraciones desde 0 (borra y crea de nuevo todas las tablas), usar `--seed` para ejecutar los seeders nuevamente
```php
php artisan migrate:fresh
```

## Factory
Basicamente es una fabrica de modelos, por estandar el nombre del Factory debe coincidir con el nombre del modelo segido de la palabra Factory.
Factories (retornan instancia de un modelo) y estos a su ves se pueden apoyar en `faker` (fachada para crear datos como nombres, correos etc) para crear datos de pruebas.
```
php artisan make:factory CategoryFactory
```

## Seeders
Basicamente siven para insertar datos previos en la base de datos, los seeders se apoyan en los 
factories. Cuando se llaman a ejecutar los factories por medio de artisan se ejecuta el archivo `DatabaseSeeder.php` asi que en ese archivo se realiza las llamadas a los factories, pero cuando hay que realizar pasos extra se recomienda crear un nuevo seeder por ejemplo crear el seeder PostSeeder
```bash
php artisan make:seeder PostSeeder
```
asi de esta manera se llama dicho seeder desde el archivo `DatabaseSeeder.php` de esta manera
```php
$this->call(UserSeeder::class);
```

## Controllers
Crear controlador de tipo api indicando el modelo para la inyeccion implita de modelo
```bash
php artisan make:conroller Api\CategoryController --api --model=Category
```

## Modelos
los modelos son una entidad que basicamente repensentan un registro de una tabla de la base de datos.

Comando de ayuda `Muy Util`
```
php artisan make:model --help
```
Crear el modelo con la -`m`igracion
```php
php artisan make:model Post -m
```
Crear -`m`odelo y -`c`ontrolador de tipo -`r`ecurso web
```php
php artisan make:model Post -mcr
```
## Relaciones entre modelos
### Uno a muchos
Vamos a definir que un Usuario tiene muchos Post entonces en el modelo `User.php` definimos el metodo posts
```php
public function posts(){
    return $this->hasMany(Post::class);
}
```
Su para definir su relacion inversa es decir un Post pertenece a un Usuario en el modelo `Post.php` defimos el metodo user
```php
public function user(){
    return $this->belongsTo(User::class);
}
```
### Muchos a muchos
Supongamos que temos la tabla posts y la tabla tags y ambas tablas tiene una relacion de muchos a muchos por medio de la tabla post_tag notese el buen uso de las convenciones de laravel para el Diseno de bases de datos ahora para definir la relacion de muchos a muchos primero vamos al modelo Post.php y definimos la funcion tags
```php
public function tags() {
    return $this->belongsToMany(Tag::class);
}
```
Ahora vamos al modelo Tag.php y definimos la funcion posts
```php
public function posts(){
    return $this->belongsToMany(Post::class);
}
```
## Relaciones entre modelos Polimorficas
### Uno a muchos
Pensemos que la tabla images es polimorfica tiene los campos `imageable_id` y `imageable_type` notese que tienen el prefijo `"imageable"` que sera usando en las definiones de las funciones que relacionan los modelos, primero empecemos con la relacion inversa una imagen pertendece a (BelongsTo) primero definimos una funcion con el nombre del prefijo imageable en el modelo `Image.php` 
```php
public function imageable(){
    return $this->morpTo();
}
```
Como una imagen puede pertenecer a muchos otros modelos entonces la function anterior morpTo no lleva paramentros. Ahora imaginemos que un post tiene muchas imagenes, vamos a el modelo Post.php y definimos el metodo images.
```php
public function images(){
    return $this->morpMany(Image::class, 'imageable');
}
```
Notese que el segundo paramentro de la anterior funcion fue el prefijo `imageable` esto es por el nombre de la primera funcion que se definio en el modelo Image.php
## Resource
basicamente funcionan como los dto en java
```bash
php artisan make:resource CategoryResource
``` 
# Bases de datos en laravel - eloquent
## Diseno de bases de datos
Lo recomentable es usar los estandares de laravel para realizar los disenos de la base de datos lo que permitira ahorrar mucho codigo en el futuro, cabe mensionar que dichos estandares sigen las reglas gramaticales en ingles.
  - El nombre de la llave primaria es `id`
  - El nombre de las bases de datos deben ser en prural `categories` o `posts`
  - En nombre de la llave primaria es en singular `id_category` o `id_post`
  - Cuando hay una tabla intermedia (rompimiento) el nombre de las tablas son en singular 
  y con guion bajo, ademas empieza la tabla que va primero alfabeticamente ejemplo tabla intermedia entre `posts` y `tags`
  seria `post_tag`

### Ejemplo llave foranea
```php
$table->unsignedBigInteger('category_id');
$table->foreign('category_id')->references('id')->on('categories');
```
Si se ha realizado todo con las convenciones de laravel se puede simplicar lo anterior como
```php
$table->foreignId('category_id')->constrained();
```
### Nota: `onUpdate` `onDelete`
Las llaves foraneas son campos que estan en una tabla pero que hacen referencia a otra tabla por eso son `foraneas`, vamos a decir que hacen parte de la tabla `local` y es en esta tabla donde se define el campo y la referencia (foreing key) hacia la tabla extranjera, ahora un tip al usar `onUpdate` o `onDelete`, `que se define enla tabla local` te puedes preguntar que pasa si la referencia o llave foranea desaparece ¿que hago con los registros locales? los borro (cascade) o los mantengo (no action) los mimo ocurre para la actualizacion.

## Relaciones polimorficas

### Uno a muchos
Ejemplo muchas tablas pueden estar asociadas a la tabla imagen, puede ser que un `User` tenga muchas imagenes o un `Post` tenga muchas imagenes esto es una `relacion polimimorfica de uno a muchos (1:N)`, entonces para definir esta relacion podemos agregar dos campos a la tabla images
- `imageable_id`
- `imageable_type`

`imageable_id`contiene el nombre la case que representa la tabla a la que hace referencia, y `imageable_id` el id del registro de dicha tabla. Asi se peuden agregar estos campos en una migracion de la tabla images.
```php
$table->unsignedBigInteger('imageable_id');
$table->string('imageable_type');
```
Si se ha realizado todo con las convenciones de laravel se puede simplicar lo anterior como
```php
$table->morphs('imageable');
```
# Eloquent

### with
Incluye una relacion que se halla definido en el modelo por ejemplo en el modelo Category se tiene una relacion posts (se ha creado la funcion posts) lo cual indica que una categoria tiene muchos post, asi se puede realizar el query pasando un id de categoria
```php
Category::with(['posts'])->find($id);
```
Ademas pensemos que en el modelo Post se tiene una relacion definida con User mediante la funcion user es decir un post pertenece a un usuario, esta relacion tambien se puede incluir en el query de la siguiente forma
```php
Category::with(['posts.user'])->find($id);
```
## Query Scopes
Es una funcion que se define en el modelo y que debe inicar con la palabra `scope` seguido del nombre que se le quiera dar, permite modificar una consulta basicamente (funciona como un pipe) Ejemplo de un query scope llamado Included en el modelo Category:

```php
//En el modelo Category
public function scopeIncluded(Builder $query)
{
    if (empty(request('included'))) { return; }
    //obtiene el valor del query parameter included y lo convierte en un array
    $relations = explode(',', request('included'));
    $query->with($relations);
}
```

```php
//En el controlador de Category
public function show($id)
{
    $category = Category::included()->findOrFail($id);
    return $category;
}
```

# Almacenamiento de Archvios
Bien laravel nos proporciona una serie de drivers para hacer uso de los archivos, estos drivers estan en `config/filesystems.php` tenemos por defento tres drivers bastante descriptivos por si solos `local`, `public` y `s3`; Como puede apreciarse en este archivo se puede seleccionar el driver mediante la variable de entorno `FILESYSTEM_DISK` que se encuentra en el archivo `.env`

## Almacenamiento en carpeta publica
Para esto selecionamos el driver `public` en el archivo .env
```
FILESYSTEM_DISK=public
```
Esto permitira que almacenemos los archivos en el dirctorio `storage/app/public` y para vinvular este directorio con la caperta `public` de nuestro proyecto que es la unica carpeta que se puede aceder directamente usamos el comando
```bash
php artisan storage:link
```
Con esto podremos acceder a nustros archivos de manera publica atraves de la ruta `http://localhost:8000/storage/file.png`, Nota: puede crear nuevos subdirectorios en `storage/app/public` de ser necesario

# APIS
## Notas
### Rertornar JSON en las validaciones del formulario en las APIS.
En la clase `App\Exceptions` la clase que hereda es `ExceptionHandler` en esta superclase se manejan todas las excepciones del framework algo asi como un ControllerAdvice en Java, esto sucede en el metodo `render($request, Throwable $e)`, algo para tener en cuenta es la excepcion `ValidationException` que es la excepcion que es lanzada en cuendo falla la validacion de un formulario al usar metodo `validate` puede retonar respuesta HTML o JSON, para que la respuesta sea JSON (que es lo que se requiere al construir un API) se debe enviar en las cabeceras `{ Accept : application/json }` en la peticion.
