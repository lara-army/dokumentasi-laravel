# Pembuatan URL

- [Pengantar](#introduction)
- [Dasar](#the-basics)
    - [Menghasilkan URL](#generating-urls)
    - [Mengakses URL Saat ini](#accessing-the-current-url)
- [URL Untuk Rute Bernama](#urls-for-named-routes)
    - [Signed URLs](#signed-urls)
- [URLs Untuk _Action Controller_](#urls-for-controller-actions)
- [Nilai _Default_](#default-values)

<a name="introduction"></a>
## Pengantar

Laravel menyediakan beberapa _helper_ untuk mendampingi Anda dalam menghasilkan URL untuk aplikasi Anda. _Helper_ ini akan sangat berguna ketika membuat pranala (_link_) pada _template_ dan respons API Anda, atau ketika membuat respons _redirect_ yang menuju bagian lain aplikasi Anda.

<a name="the-basics"></a>
## Dasar

<a name="generating-urls"></a>
### Menghasilkan URL

_Helper_ `url` dapat digunakan untuk menghasilkan URL untuk aplikasi Anda. URL yang dihasilkan akan secara otomatis menggunakan skema (HTTP atau HTTPS) dan _host_ dari _request_ yang saat ini ditangani oleh aplikasi:

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### Mengakses URL Saat Ini

Jika tidak ada _path_ yang diberikan pada _helper_ `url`, _instance_ `Illuminate\Routing\UrlGenerator` akan dikembalikan, memungkinkan Anda untuk mengakses informasi terkait URL saat ini:

    // Mengambil URL saat ini tanpa string query...
    echo url()->current();

    // Mengambil URL saat ini dengan string query...
    echo url()->full();

    // Mengambil URL penuh dari request sebelumnya...
    echo url()->previous();

Semua _method_ ini dapat diakses via [_facade_](/docs/{{version}}/facades) `URL`:

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## URL Untuk Rute Bernama

_Helper_ `route` dapat digunakan untuk menghasilkan URL untuk [_named routes_](/docs/{{version}}/routing#named-routes). route yang telah diberi nama memungkinkan Anda untuk membuat URL tanpa 

The `route` helper may be used to generate URLs to [named routes](/docs/{{version}}/routing#named-routes). Named routes allow you to generate URLs without being  to the actual URL defined on the route. Therefore, if the route's URL changes, no changes need to be made to your calls to the `route` function. For example, imagine your application contains a route defined like the following:

    Route::get('/post/{post}', function (Post $post) {
        //
    })->name('post.show');

To generate a URL to this route, you may use the `route` helper like so:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

Of course, the `route` helper may also be used to generate URLs for routes with multiple parameters:

    Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
        //
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

Any additional array elements that do not correspond to the route's definition parameters will be added to the URL's query string:

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

<a name="eloquent-models"></a>
#### Eloquent Models

You will often be generating URLs using the route key (typically the primary key) of [Eloquent models](/docs/{{version}}/eloquent). For this reason, you may pass Eloquent models as parameter values. The `route` helper will automatically extract the model's route key:

    echo route('post.show', ['post' => $post]);

<a name="signed-urls"></a>
### Signed URLs

Laravel allows you to easily create "signed" URLs to named routes. These URLs have a "signature" hash appended to the query string which allows Laravel to verify that the URL has not been modified since it was created. Signed URLs are especially useful for routes that are publicly accessible yet need a layer of protection against URL manipulation.

For example, you might use signed URLs to implement a public "unsubscribe" link that is emailed to your customers. To create a signed URL to a named route, use the `signedRoute` method of the `URL` facade:

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

If you would like to generate a temporary signed route URL that expires after a specified amount of time, you may use the `temporarySignedRoute` method. When Laravel validates a temporary signed route URL, it will ensure that the expiration timestamp that is encoded into the signed URL has not elapsed:

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

<a name="validating-signed-route-requests"></a>
#### Validating Signed Route Requests

To verify that an incoming request has a valid signature, you should call the `hasValidSignature` method on the incoming `Illuminate\Http\Request` instance:

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

Sometimes, you may need to allow your application's frontend to append data to a signed URL, such as when performing client-side pagination. Therefore, you can specify request query parameters that should be ignored when validating a signed URL using the `hasValidSignatureWhileIgnoring` method. Remember, ignoring parameters allows anyone to modify those parameters on the request:

    if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
        abort(401);
    }

Instead of validating signed URLs using the incoming request instance, you may assign the `Illuminate\Routing\Middleware\ValidateSignature` [middleware](/docs/{{version}}/middleware) to the route. If it is not already present, you should assign this middleware a key in your HTTP kernel's `routeMiddleware` array:

    /**
     * The application's route middleware.
     *
     * These middleware may be assigned to groups or used individually.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

Once you have registered the middleware in your kernel, you may attach it to a route. If the incoming request does not have a valid signature, the middleware will automatically return a `403` HTTP response:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="responding-to-invalid-signed-routes"></a>
#### Responding To Invalid Signed Routes

When someone visits a signed URL that has expired, they will receive a generic error page for the `403` HTTP status code. However, you can customize this behavior by defining a custom "renderable" closure for the `InvalidSignatureException` exception in your exception handler. This closure should return an HTTP response:

    use Illuminate\Routing\Exceptions\InvalidSignatureException;

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->renderable(function (InvalidSignatureException $e) {
            return response()->view('error.link-expired', [], 403);
        });
    }

<a name="urls-for-controller-actions"></a>
## URLs For Controller Actions

The `action` function generates a URL for the given controller action:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

If the controller method accepts route parameters, you may pass an associative array of route parameters as the second argument to the function:

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="default-values"></a>
## Default Values

For some applications, you may wish to specify request-wide default values for certain URL parameters. For example, imagine many of your routes define a `{locale}` parameter:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

It is cumbersome to always pass the `locale` every time you call the `route` helper. So, you may use the `URL::defaults` method to define a default value for this parameter that will always be applied during the current request. You may wish to call this method from a [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) so that you have access to the current request:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        /**
         * Handle the incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return \Illuminate\Http\Response
         */
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

Once the default value for the `locale` parameter has been set, you are no longer required to pass its value when generating URLs via the `route` helper.

<a name="url-defaults-middleware-priority"></a>
#### URL Defaults & Middleware Priority

Setting URL default values can interfere with Laravel's handling of implicit model bindings. Therefore, you should [prioritize your middleware](/docs/{{version}}/middleware#sorting-middleware) that set URL defaults to be executed before Laravel's own `SubstituteBindings` middleware. You can accomplish this by making sure your middleware occurs before the `SubstituteBindings` middleware within the `$middlewarePriority` property of your application's HTTP kernel.

The `$middlewarePriority` property is defined in the base `Illuminate\Foundation\Http\Kernel` class. You may copy its definition from that class and overwrite it in your application's HTTP kernel in order to modify it:

    /**
     * The priority-sorted list of middleware.
     *
     * This forces non-global middleware to always be in the given order.
     *
     * @var array
     */
    protected $middlewarePriority = [
        // ...
         \App\Http\Middleware\SetDefaultLocaleForUrls::class,
         \Illuminate\Routing\Middleware\SubstituteBindings::class,
         // ...
    ];
