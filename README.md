
![SharpAPI GitHub cover](https://sharpapi.com/sharpapi-github-laravel-bg.jpg "SharpAPI Laravel Client")

# SharpAPI Laravel Integration Guide

Welcome to the **SharpAPI Laravel Integration Guide**! This repository provides a comprehensive, step-by-step tutorial on how to integrate SharpAPI into your next [Laravel AI](https://sharpapi.com/) application. Whether you're looking to enhance your app with AI-powered features or automate workflows, this guide will walk you through the entire process, from authentication to making API calls and handling responses.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Setting Up the Laravel Project](#setting-up-the-laravel-project)
3. [Installing the SharpAPI PHP Client](#installing-the-sharpapi-php-client)
4. [Configuration](#configuration)
   - [Environment Variables](#environment-variables)
   - [Service Provider (Optional)](#service-provider-optional)
5. [Authentication with SharpAPI](#authentication-with-sharpapi)
6. [Making API Calls](#making-api-calls)
   - [Example: Generating a Job Description](#example-generating-a-job-description)
7. [Handling Responses](#handling-responses)
8. [Error Handling](#error-handling)
9. [Testing the Integration](#testing-the-integration)
10. [Advanced Usage](#advanced-usage)
    - [Asynchronous Requests](#asynchronous-requests)
    - [Caching Responses](#caching-responses)
11. [Conclusion](#conclusion)
12. [Support](#support)
13. [License](#license)

---

## Prerequisites

Before you begin, ensure you have met the following requirements:

- **PHP**: >= 8.1
- **Composer**: Dependency manager for PHP
- **Laravel**: Version 9 or higher
- **SharpAPI Account**: Obtain an API key from [SharpAPI.com](https://sharpapi.com/)
- **Basic Knowledge of Laravel**: Familiarity with Laravel framework and MVC architecture

---

## Setting Up the Laravel Project

If you already have a Laravel project, you can skip this step. Otherwise, follow these instructions to create a new Laravel project.

1. **Install Laravel via Composer**

   ```bash
   composer create-project --prefer-dist laravel/laravel laravel-ai-integration-guide
   ```

2. **Navigate to the Project Directory**

   ```bash
   cd laravel-ai-integration-guide
   ```

3. **Serve the Application**

   ```bash
   php artisan serve
   ```

   The application will be accessible at `http://localhost:8000`.

---

## Installing the SharpAPI PHP Client

To interact with SharpAPI, you'll need to install the SharpAPI PHP client library.

**Require the SharpAPI Package via Composer**

   ```bash
composer require sharpapi/sharpapi-laravel-client
php artisan vendor:publish --tag=sharpapi-laravel-client
   ```

---

## Configuration

### Environment Variables

Storing sensitive information like API keys in environment variables is a best practice. Laravel uses the `.env` file for environment-specific configurations.

1. **Open the `.env` File**

   Located in the root directory of your Laravel project.

2. **Add Your SharpAPI API Key**

   ```env
   SHARP_API_KEY=your_actual_sharpapi_api_key_here
   ```

   **Note**: Replace `your_actual_sharpapi_api_key_here` with your actual SharpAPI API key.

3. **Accessing Environment Variables in Code**

   Laravel provides the `env` helper function to access environment variables.

   ```php
   $apiKey = env('SHARP_API_KEY');
   ```

### Service Provider (Optional)

If the SharpAPI PHP client offers a Laravel service provider, you can register it for easier integration. Check the SharpAPI documentation for availability.

1. **Register the Service Provider**

   Add the service provider to the `providers` array in `config/app.php`:

   ```php
   'providers' => [
       // Other Service Providers

       SharpAPI\SharpApiServiceProvider::class,
   ],
   ```

2. **Publish Configuration (If Available)**

   ```bash
   php artisan vendor:publish --provider="SharpAPI\SharpApiServiceProvider"
   ```

   This step is optional and depends on whether SharpAPI provides a configuration file.

---

## Authentication with SharpAPI

Authentication is required to interact with SharpAPI's endpoints securely.

1. **Initialize the SharpAPI Client**

   Create a service or use it directly in your controllers.

   ```php
   <?php

   namespace App\Services;

   use SharpAPI\SharpApiService;

   class SharpApiClient
   {
       protected $client;

       public function __construct()
       {
           $this->client = new SharpApiService(env('SHARP_API_KEY'));
       }

       public function getClient()
       {
           return $this->client;
       }
   }
   ```

2. **Binding the Service in a Service Provider (Optional)**

   This allows you to inject the service wherever needed.

   ```php
   <?php

   namespace App\Providers;

   use Illuminate\Support\ServiceProvider;
   use App\Services\SharpApiClient;

   class AppServiceProvider extends ServiceProvider
   {
       public function register()
       {
           $this->app->singleton(SharpApiClient::class, function ($app) {
               return new SharpApiClient();
           });
       }

       public function boot()
       {
           //
       }
   }
   ```

3. **Using the Service in a Controller**

   ```php
   <?php

   namespace App\Http\Controllers;

   use Illuminate\Http\Request;
   use App\Services\SharpApiClient;

   class SharpApiController extends Controller
   {
       protected $sharpApi;

       public function __construct(SharpApiClient $sharpApi)
       {
           $this->sharpApi = $sharpApi->getClient();
       }

       public function ping()
       {
           $response = $this->sharpApi->ping();
           return response()->json($response);
       }
   }
   ```

4. **Defining Routes**

   Add routes to `routes/web.php` or `routes/api.php`:

   ```php
   use App\Http\Controllers\SharpApiController;

   Route::get('/sharpapi/ping', [SharpApiController::class, 'ping']);
   ```

---

## Making API Calls

Once authenticated, you can start making API calls to various SharpAPI endpoints. Below are examples of how to interact with different endpoints.

### Example: Generating a Job Description

1. **Create a Job Description Parameters DTO**

   ```php
   <?php

   namespace App\Http\Controllers;

   use Illuminate\Http\Request;
   use App\Services\SharpApiClient;
   use SharpAPI\Dto\JobDescriptionParameters;

   class SharpApiController extends Controller
   {
       protected $sharpApi;

       public function __construct(SharpApiClient $sharpApi)
       {
           $this->sharpApi = $sharpApi->getClient();
       }

       public function generateJobDescription()
       {
           $jobDescriptionParams = new JobDescriptionParameters(
               "Software Engineer",
               "Tech Corp",
               "5 years",
               "Bachelor's Degree in Computer Science",
               "Full-time",
               [
                   "Develop software applications",
                   "Collaborate with cross-functional teams",
                   "Participate in agile development processes"
               ],
               [
                   "Proficiency in PHP and Laravel",
                   "Experience with RESTful APIs",
                   "Strong problem-solving skills"
               ],
               "USA",
               true,   // isRemote
               true,   // hasBenefits
               "Enthusiastic",
               "Category C driving license",
               "English"
           );

           $statusUrl = $this->sharpApi->generateJobDescription($jobDescriptionParams);
           $resultJob = $this->sharpApi->fetchResults($statusUrl);

           return response()->json($resultJob->getResultJson());
       }
   }
   ```

2. **Define the Route**

   ```php
   Route::get('/sharpapi/generate-job-description', [SharpApiController::class, 'generateJobDescription']);
   ```

3. **Accessing the Endpoint**

   Visit `http://localhost:8000/sharpapi/generate-job-description` to see the generated job description.

---

## Handling Responses

SharpAPI responses are typically encapsulated in job objects. To handle these responses effectively:

1. **Understanding the Response Structure**

   ```json
   {
       "id": "uuid",
       "type": "JobType",
       "status": "Completed",
       "result": {
           // Result data
       }
   }
   ```

2. **Accessing the Result**

   Use the provided methods to access the result data.

   ```php
   $resultJob = $this->sharpApi->fetchResults($statusUrl);
   $resultData = $resultJob->getResultObject(); // As a PHP object
   // or
   $resultJson = $resultJob->getResultJson(); // As a JSON string
   ```

3. **Example Usage in Controller**

   ```php
   public function generateJobDescription()
   {
       // ... (initialize and make API call)

       if ($resultJob->getStatus() === 'Completed') {
           $resultData = $resultJob->getResultObject();
           // Process the result data as needed
           return response()->json($resultData);
       } else {
           return response()->json(['message' => 'Job not completed yet.'], 202);
       }
   }
   ```

---

## Error Handling

Proper error handling ensures that your application can gracefully handle issues that arise during API interactions.

1. **Catching Exceptions**

   Wrap your API calls in try-catch blocks to handle exceptions.

   ```php
   public function generateJobDescription()
   {
       try {
           // ... (initialize and make API call)

           $resultJob = $this->sharpApi->fetchResults($statusUrl);
           return response()->json($resultJob->getResultJson());
       } catch (\Exception $e) {
           return response()->json([
               'error' => 'An error occurred while generating the job description.',
               'message' => $e->getMessage()
           ], 500);
       }
   }
   ```

2. **Handling API Errors**

   Check the status of the job and handle different statuses accordingly.

   ```php
   if ($resultJob->getStatus() === 'Completed') {
       // Handle success
   } elseif ($resultJob->getStatus() === 'Failed') {
       // Handle failure
       $error = $resultJob->getResultObject()->error;
       return response()->json(['error' => $error], 400);
   } else {
       // Handle other statuses (e.g., Pending, In Progress)
       return response()->json(['message' => 'Job is still in progress.'], 202);
   }
   ```

---

## Testing the Integration

Testing is crucial to ensure that your integration with SharpAPI works as expected.

1. **Writing Unit Tests**

   Use Laravel's built-in testing tools to write unit tests for your SharpAPI integration.

   ```php
   <?php

   namespace Tests\Feature;

   use Tests\TestCase;
   use App\Services\SharpApiClient;
   use SharpAPI\Dto\JobDescriptionParameters;

   class SharpApiTest extends TestCase
   {
       protected $sharpApi;

       protected function setUp(): void
       {
           parent::setUp();
           $this->sharpApi = new SharpApiClient();
       }

       public function testPing()
       {
           $response = $this->sharpApi->ping();
           $this->assertEquals('OK', $response['status']);
       }

       public function testGenerateJobDescription()
       {
           $jobDescriptionParams = new JobDescriptionParameters(
               "Backend Developer",
               "InnovateTech",
               "3 years",
               "Bachelor's Degree in Computer Science",
               "Full-time",
               ["Develop APIs", "Optimize database queries"],
               ["Proficiency in PHP and Laravel", "Experience with RESTful APIs"],
               "USA",
               true,
               true,
               "Professional",
               "Category B driving license",
               "English"
           );

           $statusUrl = $this->sharpApi->generateJobDescription($jobDescriptionParams);
           $resultJob = $this->sharpApi->fetchResults($statusUrl);

           $this->assertEquals('Completed', $resultJob->getStatus());
           $this->assertNotEmpty($resultJob->getResultObject());
       }

       // Add more tests for other methods...
   }
   ```

2. **Running Tests**

   Execute your tests using PHPUnit.

   ```bash
   ./vendor/bin/phpunit
   ```

---

## Advanced Usage

### Asynchronous Requests

For handling multiple API requests concurrently, consider implementing asynchronous processing using Laravel Queues.

1. **Setting Up Queues**

   Configure your queue driver in the `.env` file.

   ```env
   QUEUE_CONNECTION=database
   ```

   Run the necessary migrations.

   ```bash
   php artisan queue:table
   php artisan migrate
   ```

2. **Creating a Job**

   ```bash
   php artisan make:job ProcessSharpApiRequest
   ```

   ```php
   <?php

   namespace App\Jobs;

   use Illuminate\Bus\Queueable;
   use Illuminate\Contracts\Queue\ShouldQueue;
   use Illuminate\Foundation\Bus\Dispatchable;
   use Illuminate\Queue\InteractsWithQueue;
   use Illuminate\Queue\SerializesModels;
   use App\Services\SharpApiClient;
   use SharpAPI\Dto\JobDescriptionParameters;

   class ProcessSharpApiRequest implements ShouldQueue
   {
       use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

       protected $params;

       public function __construct(JobDescriptionParameters $params)
       {
           $this->params = $params;
       }

       public function handle(SharpApiClient $sharpApi)
       {
           $statusUrl = $sharpApi->generateJobDescription($this->params);
           $resultJob = $sharpApi->fetchResults($statusUrl);
           // Handle the result...
       }
   }
   ```

3. **Dispatching the Job**

   ```php
   use App\Jobs\ProcessSharpApiRequest;

   public function generateJobDescriptionAsync()
   {
       $jobDescriptionParams = new JobDescriptionParameters(
           // ... parameters
       );

       ProcessSharpApiRequest::dispatch($jobDescriptionParams);
       return response()->json(['message' => 'Job dispatched successfully.']);
   }
   ```

4. **Running the Queue Worker**

   ```bash
   php artisan queue:work
   ```

### Caching Responses

To optimize performance and reduce redundant API calls, implement caching.

1. **Using Laravel's Cache Facade**

   ```php
   use Illuminate\Support\Facades\Cache;

   public function generateJobDescription()
   {
       $cacheKey = 'job_description_' . md5(json_encode($jobDescriptionParams));
       $result = Cache::remember($cacheKey, 3600, function () use ($jobDescriptionParams) {
           $statusUrl = $this->sharpApi->generateJobDescription($jobDescriptionParams);
           $resultJob = $this->sharpApi->fetchResults($statusUrl);
           return $resultJob->getResultJson();
       });

       return response()->json(json_decode($result, true));
   }
   ```

2. **Invalidating Cache**

   When the underlying data changes, ensure to invalidate the relevant cache.

   ```php
   Cache::forget('job_description_' . md5(json_encode($jobDescriptionParams)));
   ```

---

## Conclusion

Integrating SharpAPI into your Laravel application unlocks a myriad of AI-powered functionalities, enhancing your application's capabilities and providing seamless workflow automation. This guide has walked you through the essential steps, from setting up authentication to making API calls and handling responses. With the examples and best practices provided, you're well-equipped to leverage SharpAPI's powerful features in your Laravel projects.

---

## Support

If you encounter any issues or have questions regarding the integration process, feel free to open an issue on the [GitHub repository](https://github.com/sharpapi/laravel-integration/issues) or contact our support team at [contact@sharpapi.com](mailto:contact@sharpapi.com).

---

## License

This project is licensed under the [MIT License](LICENSE).

---
