---
layout: post
title: "Build an admin panel using Laravel 5"
date: 2018-03-01
categories: ["Laravel"]
toc: false
lede: "Authorization and admin panels have always been a sticky point for me. This is the simplist way I know how to build one, using Laravel"
author: Connor Leech
published: true
---

Authorization can be tricky. There are thousands of posts about how to perform authentication, but actually verifying who someone is and managing user permissions can be a whole can of worms. Fortunately, Laravel has systems in place that make a tiered login system very easy to implement.

Before we get started I’d like to give props 👏 to [Nick Basile](https://nick-basile.com/blog/post/how-to-build-an-admin-in-laravel-using-tdd) and his excellent blog posts on this topic. To add authentication to a Laravel 5 app, all you need is one command:

```
$ php artisan make:auth
```

That’s it. If you’re new to Laravel, welcome. For Laravel developers this feature has been around for a long time. Now we’ve got our auth system with login forms and everything. That all works, but for the purposes of this post we’re interested in authorization.

> Source code for this article is available [on github](https://github.com/connor11528/laravel-vue-tasks)

Easy solution for making a Laravel admin page using custom middleware
I am using Laravel 5.5 right now, the latest release. The only specific Laravel 5.5 thing going on is the @guest helper in the frontend Blade directives. In the HTML section of the application, these helpers allow us to easily check if the user is logged in or not:

```
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

If you’re not using Laravel 5.5 there are other workarounds but you might as well upgrade to the latest version for the new features!

## 🔒 Okay, actually building the system now

There are lots of ways to build an authorization system. There are pre-built packages that allow you to manage roles and permissions. I’m sure they’re great as they are maintained by Spatie, who makes all the bomb Laravel packages. For this though, I didn’t want to bring in a heavy package and make it work. All I wanted for this Laravel app was to have a little quiet place that only I, as an administrator can login to. Everyone else can login and see their dashboard or profile or whatever but only I can login and see an admin page.

How we achieve this is to add a type column on the users table and check if a user has that type via custom middleware. It sounds fancy but it’s pretty easy!

1 - Add the types you want to the User model and a method to check if a user is an admin.

```
/* app/User.php */
const ADMIN_TYPE = 'admin';
const DEFAULT_TYPE = 'default';
public function isAdmin()    {        
    return $this->type === self::ADMIN_TYPE;    
}
```

2 - Add the type column to the migration that created your users table

```
/* database/migrations/2014_10_12_000000_create_users_table.php */
$table->string('type')->default('default');
```

3 - Add a type value to the create method in register controller

```
/* app/Http/Controllers/Auth/RegisterController.php */
protected function create(array $data)    {        
    return User::create([            
        'name' => $data['name'],
        'email' => $data['email'],            
        'password' => bcrypt($data['password']),            
        'type' => User::DEFAULT_TYPE,        
    ]);    
}
```

4 - Create a custom middleware file to check if a user is an admin. Generate this file using php artisan make:middleware IsAdmin


5 - Register the middleware you just created

```
/* app/Http/Kernel.php */
'is_admin' => \App\Http\Middleware\IsAdmin::class,
```

6 - Add some routes that invoke the middleware

```
/* routes/web.php */
Route::view('/', 'welcome');
Auth::routes();
Route::get('/home', 'HomeController@index')    
    ->name('home');
Route::get('/admin', 'AdminController@admin')    
    ->middleware('is_admin')    
    ->name('admin');
```

7 - Create an admin controller with `php artisan make:controller AdminController`. This controller returns the dashboard for whatever view you want your admin to see.


## 🎉 That’s pretty much it!

Now if you visit /admin and you’re not logged in or logged in as an administrator you won’t be able to access the page. In order to create an admin user you can use the tinker artisan comman:

```
$ php artisan tinker
>>> use App\User;
>>>User::where('email', 'connorleech@gmail.com')->update(['type' => 'admin']);
```

Then when you login as that user you will be able to see the admin page!

----

Full post [on Medium](https://medium.com/@connorleech/easily-build-administrator-login-into-a-laravel-5-app-8a942e4fef37)