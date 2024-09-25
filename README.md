## INSTALL BREEZE COMPOSER (Secure Multi-Auth Systems in Laravel 11 with Laravel Breeze)

->composer require laravel/breeze --dev
->php artisan breeze:install
->Which Breeze stack would you like to install?: [Blade with Alpine .................................................................. blade]
->Would you like dark mode support? (yes/no) [no]
->Which testing framework do you prefer? [Pest]

->npm install
->npm run build

## Migrate Custom Table

php artisan migrate:fresh

Register a dummy user in http://127.0.0.1:8000/register for testing purpose with following account:
1: Super Admin | 2: Admin | 3: Normal User
Normal User
Admin
Super Admin

## Update User Column Using command

->php artisan tinker

find(2) is the id of the user

$user->role is the column that you update

save() to execute and save the database comand

Type this in prompt
->$user = App\Models\User::find(2)
->$user->role = 1
->$user->save()

## Create a separate auth role

->php artisan make:view super-admin/dashboard

## Modify The Controller

File Location: app/Http/Controllers/Auth/AuthenticatedSessionController.php

1.  Go to store() function
2.  Add this following code
    //Determine the user role
    $loggedInUserRole = $request->user()->role;
    //Super Admin
    if ($loggedInUserRole == 1){
    return redirect()->intended(route('super-admin.dashboard',absolute: false));
    }
    //Admin
    elseif($loggedInUserRole == 2){
    return redirect()->intended(route('admin.dashboard', absolute: false));
    }
    //user or if the account is not admin
    return redirect()->intended(route('dashboard', absolute: false));

## Add Middleware due to redirecting to other role link eg. user redirect to superadmin/dashboard

File Location: File Location: app/Http/Controllers/Middleware

1.  Add this following command.
    ->php artisan make:middleware SuperAdmin
    ->php artisan make:middleware Admin
    ->php artisan make:middleware NormalUser
2.  Go to SuperAdmin.php, Admin.php, NormalUser.php
3.  Add or import this code to Middleware php class:
    use Illuminate\Support\Facades\Auth;
4.  Add this code in the handle() function:
    //if user is not logged in
    if(!Auth::check()){
    return redirect()->route('login');
    }

    $userRole = Auth()::user()->role;

    //Super Admin
    if($userRole == 1){
        return $next($request);
    }

    // Admin
    elseif($userRole == 2){
    return redirect()->route('admin.dashboard');
    }

    //Normal User
    elseif($userRole == 3){
    return redirect()->route('dashboard');
    }

## Optimized Bootstrap Files MiddleWare Roles

File Location bootstrap/app.php
->php artisan optimize:clear

1. Import this following codes in app.php:
   use App\Http\Middleware\Admin;
   use App\Http\Middleware\SuperAdmin;
   use App\Http\Middleware\NormalUser;
2. Add the following code:
   ->withMiddleware(function (Middleware $middleware) {
   $middleware->alias([
   'super-admin' => SuperAdmin::class,
   'admin' => Admin::class,
   'user' => NormalUser::class,
   ]);
   })

## Apply Middleware alias in Routes using these code

//Normal User
Route::get('/dashboard', function () {
return view('dashboard');
})->middleware(['auth', 'verified', 'user'])->name('dashboard');

//Super Admin
Route::get('/super-admin/dashboard', function () {
return view('super-admin.dashboard');
})->middleware(['auth', 'verified', 'super-admin'])->name('super-admin.dashboard');

//Admin
Route::get('/admin/dashboard', function () {
return view('admin.dashboard');
})->middleware(['auth', 'verified', 'admin'])->name('admin.dashboard');

## Dynamic Url

1.  Go to views/layouts/navigation.blade.php
2.  Add These Follwing Code in Line 1
    @php
    $roleType = Auth::user()->role;
    $redirectUrl = 'dashboard';
    switch ($roleType) {
    case 1:
    $redirectUrl = 'super-admin.dashboard';
    break;
    case 2:
    $redirectUrl = 'admin.dashboard';
    break;
    default:
    $redirectUrl = 'dashboard';
    break;
    }
    @endphp
