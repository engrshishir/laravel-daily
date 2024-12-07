<details>
<summary style="display: flex; justify-content: space-between; align-items: center; background-color: ##28a745; color: white; padding: 5px 10px; border-radius: 5px; cursor:pointer;">
  <span>🚀 Define a foreign key constraint</span>
  <span style="margin-left: auto; font-weight: bold;  color:red;">📅 November 27, 2024</span>
</summary>

```php
$table->foreignId('organization_id');
```

- This creates an organization_id column in the table.
- The column is an unsigned big integer (BIGINT UNSIGNED) by default, which is
  the same type used for primary keys when using $table->id()

```php
 ->constrained();
```

- This automatically sets up a foreign key constraint for the organization_id
  column.
- By default, it assumes the foreign key references the id column on the
  organizations table (based on the column name organization_id)

```php
$table->foreignId('organization_id')->constrained();

//OR

$table->unsignedBigInteger('organization_id');
$table->foreign('organization_id')->references('id')->on('organizations');
```

## Customizing foreign key setup behavior

If the foreign key references a column other than id or a table with a different
name, you can explicitly specify it:

- Field in the Current Table (organization_id)
- Referenced Table (companies)
- Referenced Field (company_id)

```php
$table->foreignId('organization_id')->constrained('companies', 'company_id');

// OR

$table->unsignedBigInteger('organization_id');
$table->foreign('organization_id')
      ->references('company_id')
      ->on('companies');

```

## Foreign key Cascading Options

- cascade: Automatically delete or update dependent rows.
- restrict: Prevent deletion or updates if there are dependent rows.
- set null: Set the foreign key column to NULL if the parent is deleted.

```php
$table->foreignId('organization_id')
      ->constrained()
      ->onDelete('cascade')
      ->onUpdate('cascade');

```

</details>









<br>
<details>
<summary style="display: flex; justify-content: space-between; align-items: center; background-color: ##1345; color: white; padding: 5px; border-radius: 5px; cursor:pointer;">
  <span>🚀 One to One Relationship</span>
  <span style="margin-left: auto; font-weight: bold;">📅 November 27, 2024</span>
</summary>

- Parent table: organizations
- Child table: abouts

```php
// In Organization model
public function about()
{
      return $this->hasOne(About::class);
}

// In About model
public function organization()
{
      return $this->belongsTo(Organization::class);
}

```

## Accessing Related Data
To efficiently load the organization relationship when retrieving an About record, you can use the with method.
```php
// Approach 1
$organization = Organization::find(1);
$about = $organization->about;

// Approach 2
$about = About::with('organization')->find(1);
$organization = $about->organization;
```
</details>





<br>
<details>
<summary style="display: flex; justify-content: space-between; align-items: center; background-color: ##1345; color: white; padding: 5px; border-radius: 5px; cursor:pointer;">
  <span>🚀 Fillable & Mass assignment</span>
  <span style="margin-left: auto; font-weight: bold; color:red;">📅 December 07, 2024</span>
</summary>

- $fillable: Defines a whitelist of attributes that are allowed for mass assignment.
- $guarded: Setting it to an empty array ([]) allows all attributes to be mass-assignable. 

```php
protected $fillable = ['name', 'phone', 'email', 'age', 'country'];

About::create([
    'name' => 'John Doe', 
    'phone' => '123456789',
    'email' => 'johndoe@example.com',
    'age' => 30,
]);

```

```php
protected $guarded = [];

About::create([
    'name' => 'John Doe', 
    'phone' => '123456789',
    'email' => 'johndoe@example.com',
    'age' => 30,
    'id' => 1,         // Overwrites the first record
    'is_admin' => true // Unexpected, if such a field exists
]);

```


+ mass-assignment protection is completely disabled. This means Laravel will allow any attribute provided in the input array to be directly inserted into the database

```php
protected $guarded = ['id'];

About::create([
    'name' => 'John Doe',
    'phone' => '123456789',
    'email' => 'johndoe@example.com',
    'id' => 10, // Will be ignored because it's guarded
]);
```
</details>





<br>
<details>
<summary style="display: flex; justify-content: space-between; align-items: center; background-color: ##1345; color: white; padding: 5px; border-radius: 5px; cursor:pointer;">
  <span>🚀 Update the Database Schema</span>
  <span style="margin-left: auto; font-weight: bold;">📅 December 07, 2024</span>
</summary>

- You need to add the is_admin column to the users table. Create a new migration:

```php
 php artisan make:migration add_is_admin_to_users_table --table=users
```
+ Edit the generated migration file:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class AddIsAdminToUsersTable extends Migration
{
    /**
     * php artisan migrate
     * This executes the up() method, adding the is_admin column.
     */
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->boolean('is_admin')->default(false)->after('password'); // Add the is_admin column
        });
    }

    /**
     * Rollback the Migration:
     * php artisan migrate:rollback
     * This executes the down() method, removing the is_admin column.
     */
    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('is_admin'); // Remove the is_admin column
        });
    }
}

```
+  Update the User Model
```php
    protected $fillable = [
        'is_admin',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
            'is_admin' => 'boolean', // Add this line
        ];
    }
```
</details>










<br>
<details>
<summary style="display: flex; justify-content: space-between; align-items: center; background-color: ##1345; color: white; padding: 5px; border-radius: 5px; cursor:pointer;">
  <span>🚀 Middleware </span>
  <span style="margin-left: auto; font-weight: bold;">📅 December 07, 2024</span>
</summary>

- php artisan make:middleware IsAdmin

```php
php artisan make:middleware IsAdmin
```
+ IsAdmin middleware file:

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class IsAdmin
{
    public function handle(Request $request, Closure $next)
    {
        if (auth()->check() && auth()->user()->is_admin) {
            return $next($request);
        }

        abort(403, 'Unauthorized'); // Deny access
    }
}
```


+  Register the middleware in `app/Http/Kernel.php` if kernel.php present:
```php
protected $routeMiddleware = [
    // Other middleware
    'is_admin' => \App\Http\Middleware\IsAdmin::class,
];

```

+  Register the middleware in `bootstrap/app.php` if kernel.php not present:
```php
<?php
   
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
   
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'isAdmin' => \App\Http\Middleware\IsAdmin::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

+ Apply Middleware
```php
<?php
Route::middleware(['isAdmin'])->group(function () {
    Route::get('/dashboard', function () {
        return 'Dashboard';
    });
      
    Route::get('/users', function () {
        return 'Users';
    });
});
```
</details>