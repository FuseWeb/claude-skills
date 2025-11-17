# PestPHP Expert Skill

You are an expert in PestPHP testing framework with comprehensive knowledge of Pest v3, v4, and modern PHP testing practices for 2025.

## Core Expertise

PestPHP is an elegant PHP testing framework built on PHPUnit 11, offering a developer-friendly syntax with powerful features for comprehensive test coverage.

### Latest Versions (2025)
- **Pest v4.x** (2025) - Latest version, requires PHP 8.3+, includes browser testing
- **Pest v3.x** (2024) - Stable version, requires PHP 8.2+
- Built on **PHPUnit 11** foundation

## Installation & Setup

### Installation
```bash
composer require pestphp/pest --dev --with-all-dependencies

# Install Pest plugin for Laravel
composer require pestphp/pest-plugin-laravel --dev

# Install other plugins
composer require pestphp/pest-plugin-arch --dev
composer require pestphp/pest-plugin-type-coverage --dev
composer require pestphp/pest-plugin-drift --dev
```

### Initialize Pest
```bash
./vendor/bin/pest --init
```

### Modern Configuration API (Pest 3+)
```php
// tests/Pest.php

use PHPUnit\Framework\TestCase;

pest()
    ->extends(TestCase::class)
    ->in('Unit');

pest()
    ->extends(Tests\TestCase::class)
    ->use(RefreshDatabase::class)
    ->in('Feature');

// Global expectations
pest()->project()
    ->expect()->toUseStrictEquality()
    ->expect()->toHaveMethodsDocumented();
```

## Basic Test Syntax

### Simple Test
```php
test('user can be created', function () {
    $user = User::create([
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]);

    expect($user->name)->toBe('John Doe');
    expect($user->email)->toBe('john@example.com');
});

// Alternative syntax using it()
it('creates a user', function () {
    $user = User::create([
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]);

    expect($user)->toBeInstanceOf(User::class);
});
```

### Nested Describes (Pest 3+)
```php
describe('User', function () {
    it('can be created', function () {
        $user = User::create(['name' => 'John']);
        expect($user)->toBeInstanceOf(User::class);
    });

    describe('authentication', function () {
        it('can login', function () {
            $user = User::factory()->create();
            $response = $this->post('/login', [
                'email' => $user->email,
                'password' => 'password',
            ]);

            $response->assertStatus(200);
        });

        it('cannot login with wrong password', function () {
            $user = User::factory()->create();
            $response = $this->post('/login', [
                'email' => $user->email,
                'password' => 'wrong',
            ]);

            $response->assertStatus(401);
        });
    });

    describe('profile', function () {
        it('can be updated', function () {
            // Test implementation
        });
    });
});
```

## Powerful Expectations

### Basic Expectations
```php
expect($value)->toBe('expected');
expect($value)->toEqual($expected);
expect($value)->toBeTrue();
expect($value)->toBeFalse();
expect($value)->toBeNull();
expect($value)->toBeEmpty();
expect($value)->toBeGreaterThan(10);
expect($value)->toBeLessThan(100);
```

### Type Expectations
```php
expect($user)->toBeInstanceOf(User::class);
expect($value)->toBeInt();
expect($value)->toBeFloat();
expect($value)->toBeString();
expect($value)->toBeBool();
expect($value)->toBeArray();
expect($value)->toBeObject();
expect($value)->toBeCallable();
expect($value)->toBeIterable();
```

### String Expectations
```php
expect($string)->toContain('substring');
expect($string)->toStartWith('prefix');
expect($string)->toEndWith('suffix');
expect($string)->toMatch('/regex/');
expect($email)->toBeEmail();
expect($url)->toBeUrl();
expect($json)->toBeJson();
```

### Array Expectations
```php
expect($array)->toHaveCount(3);
expect($array)->toHaveKey('name');
expect($array)->toContain('value');
expect($array)->toHaveKeys(['name', 'email']);
expect($array)->each->toBeString();
expect($array)->sequence(
    fn ($item) => $item->toBe('first'),
    fn ($item) => $item->toBe('second'),
);
```

### Collection Expectations (Laravel)
```php
expect($users)->toHaveCount(10);
expect($users)->first()->name->toBe('John');
expect($users)->each->toBeInstanceOf(User::class);
expect($users->pluck('email'))->toContain('john@example.com');
```

### Exception Expectations
```php
expect(fn () => User::find(999))->toThrow(ModelNotFoundException::class);
expect(fn () => $service->process())->toThrow(InvalidArgumentException::class, 'Invalid input');
expect(fn () => $calculator->divide(10, 0))->toThrow(DivisionByZeroError::class);
```

### Custom Expectations
```php
// tests/Pest.php
expect()->extend('toBeValidEmail', function () {
    return $this->toMatch('/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/');
});

// Usage
expect('test@example.com')->toBeValidEmail();
```

## Higher Order Tests

### Using Higher Order Functions
```php
test('user has expected attributes')
    ->expect(fn () => User::factory()->create())
    ->toHaveProperties(['name', 'email', 'created_at'])
    ->name->toBeString()
    ->email->toBeEmail();
```

## Datasets

### Basic Dataset
```php
it('generates correct slugs', function (string $input, string $expected) {
    expect(Str::slug($input))->toBe($expected);
})->with([
    ['Hello World', 'hello-world'],
    ['Pest PHP', 'pest-php'],
    ['Testing 123', 'testing-123'],
]);
```

### Named Datasets
```php
dataset('emails', [
    'valid gmail' => 'user@gmail.com',
    'valid yahoo' => 'user@yahoo.com',
    'invalid' => 'not-an-email',
]);

it('validates emails', function (string $email) {
    expect(filter_var($email, FILTER_VALIDATE_EMAIL))->not->toBeFalse();
})->with('emails');
```

### Shared Datasets
```php
// tests/Datasets/Users.php
dataset('users', [
    ['John Doe', 'john@example.com'],
    ['Jane Doe', 'jane@example.com'],
]);

// Use in tests
it('creates users', function (string $name, string $email) {
    $user = User::create(compact('name', 'email'));
    expect($user->name)->toBe($name);
})->with('users');
```

### Combining Datasets
```php
it('tests combinations', function (string $name, string $email, string $role) {
    // Test implementation
})->with('names', 'emails', 'roles');
```

## Hooks (Setup and Teardown)

### beforeEach / afterEach
```php
beforeEach(function () {
    $this->user = User::factory()->create();
});

afterEach(function () {
    Mail::fake();
});

it('sends welcome email', function () {
    $this->user->sendWelcomeEmail();

    Mail::assertSent(WelcomeEmail::class);
});
```

### beforeAll / afterAll
```php
beforeAll(function () {
    // Runs once before all tests in the file
});

afterAll(function () {
    // Runs once after all tests in the file
});
```

## Architectural Testing (Pest 3+)

### Arch Presets

#### PHP Preset
```php
// Prevents debugging functions and deprecated code
arch()->preset()->php();

// Custom configuration
arch()->preset()->php()->ignoring('var_dump');
```

#### Security Preset
```php
// Prevents security vulnerabilities
arch()->preset()->security();

// Allow specific functions
arch()->preset()->security()->ignoring('md5');
```

#### Laravel Preset
```php
// Enforces Laravel conventions
arch()->preset()->laravel();
```

#### Strict Preset
```php
// Mandates strict types and final classes
arch()->preset()->strict();
```

#### Relaxed Preset
```php
// Opposite of strict
arch()->preset()->relaxed();
```

### Custom Arch Rules
```php
arch('controllers should extend base controller')
    ->expect('App\Http\Controllers')
    ->toExtend('App\Http\Controllers\Controller');

arch('models should be in Models namespace')
    ->expect('App\Models')
    ->toBeClasses()
    ->not->toBeAbstract();

arch('services should have interface')
    ->expect('App\Services')
    ->toHaveMethod('__construct')
    ->toImplement('*Interface');

arch('no debugging functions in source')
    ->expect(['dd', 'dump', 'var_dump'])
    ->not->toBeUsed();

arch('strict types must be declared')
    ->expect('App')
    ->toHaveFileSystemPermissions(0644);
```

### Global Expectations
```php
pest()->project()
    ->expect()->toUseStrictEquality()
    ->expect()->toHaveMethodsDocumented()
    ->expect()->not->toHavePrivateMethods();

// In your architecture tests
arch('use strict equality')
    ->expect('App')
    ->toUseStrictEquality();

arch('all methods should be documented')
    ->expect('App\Services')
    ->toHaveMethodsDocumented();

arch('no private methods')
    ->expect('App')
    ->not->toHavePrivateMethods();
```

## Mutation Testing (Pest 3+)

### Basic Mutation Testing
```php
// Add covers annotation
covers(TodoController::class);

it('lists todos', function () {
    $response = $this->getJson('/todos');
    $response->assertStatus(200);
    $response->assertJsonStructure(['data']);
});

// Run mutation tests
// ./vendor/bin/pest --mutate
```

### Mutation Testing Configuration
```bash
# Run mutation testing in parallel
./vendor/bin/pest --mutate --parallel

# Target specific files
./vendor/bin/pest --mutate --covered-only

# Set minimum mutation score
./vendor/bin/pest --mutate --min=80
```

### Understanding Mutation Score
```php
// Goal: Aim for 100% mutation score
// - Tests should catch all intentional code changes
// - Indicates comprehensive test coverage
// - Better metric than code coverage percentage
```

## Team Management (Pest 3+)

### Todo Tracking
```php
it('has contact page', function () {
    // Test implementation pending
})->todo(assignee: 'john@example.com', issue: 123);

it('sends notification email', function () {
    // Not implemented yet
})->todo();

// Filter by assignee
// ./vendor/bin/pest --todo-assignee=john@example.com

// Filter by issue
// ./vendor/bin/pest --todo-issue=123
```

## Test Organization

### Test Attributes
```php
use Pest\Attributes\Group;
use Pest\Attributes\Depends;

#[Group('integration')]
it('connects to database', function () {
    // Test implementation
});

test('user creation')
    ->group('users', 'integration')
    ->skip('Not ready yet');

it('depends on previous test', function () {
    // This test runs only if the dependent test passes
})->depends('user creation');
```

### Skip and Only
```php
it('is skipped', function () {
    // Will not run
})->skip();

it('skips on condition', function () {
    // Will not run if condition is true
})->skipOnPhp('8.4');

it('skips on OS', function () {
    // Will not run on Windows
})->skipOnWindows();

it('only runs this test', function () {
    // Only this test will run
})->only();
```

## Laravel Integration

### Database Testing
```php
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

it('creates a user in database', function () {
    $user = User::factory()->create(['name' => 'John']);

    $this->assertDatabaseHas('users', [
        'name' => 'John',
    ]);

    expect(User::count())->toBe(1);
});
```

### HTTP Testing
```php
it('returns a successful response', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
    $response->assertSee('Welcome');
});

it('creates a user via API', function () {
    $response = $this->postJson('/api/users', [
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]);

    $response->assertStatus(201);
    $response->assertJson([
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]);
});
```

### Authentication Testing
```php
it('allows authenticated users to access dashboard', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)->get('/dashboard');

    $response->assertStatus(200);
});

it('redirects unauthenticated users', function () {
    $response = $this->get('/dashboard');

    $response->assertRedirect('/login');
});
```

### Event Testing
```php
use Illuminate\Support\Facades\Event;

it('dispatches user created event', function () {
    Event::fake([UserCreated::class]);

    $user = User::factory()->create();

    Event::assertDispatched(UserCreated::class, function ($event) use ($user) {
        return $event->user->id === $user->id;
    });
});
```

### Queue Testing
```php
use Illuminate\Support\Facades\Queue;

it('dispatches job to queue', function () {
    Queue::fake();

    dispatch(new ProcessUser($user));

    Queue::assertPushed(ProcessUser::class);
});
```

### Mail Testing
```php
use Illuminate\Support\Facades\Mail;

it('sends welcome email', function () {
    Mail::fake();

    $user = User::factory()->create();
    $user->sendWelcomeEmail();

    Mail::assertSent(WelcomeEmail::class, function ($mail) use ($user) {
        return $mail->hasTo($user->email);
    });
});
```

## Custom Helpers

### Global Functions
```php
// tests/Pest.php

function createUser(array $attributes = []): User
{
    return User::factory()->create($attributes);
}

function actingAsAdmin(): TestCase
{
    $admin = User::factory()->admin()->create();
    return test()->actingAs($admin);
}

// Usage in tests
it('allows admin to delete users', function () {
    actingAsAdmin();
    $user = createUser();

    $response = $this->delete("/users/{$user->id}");

    $response->assertStatus(200);
});
```

## Parallel Testing

### Run Tests in Parallel
```bash
# Parallel execution
./vendor/bin/pest --parallel

# Specify number of processes
./vendor/bin/pest --parallel --processes=4
```

## Code Coverage

### Generate Coverage Report
```bash
# HTML coverage report
./vendor/bin/pest --coverage --coverage-html=coverage

# Minimum coverage threshold
./vendor/bin/pest --coverage --min=80

# Coverage for specific paths
./vendor/bin/pest --coverage --path=app/Services
```

## Type Coverage (Pest Plugin)

### Check Type Coverage
```bash
# Install plugin
composer require pestphp/pest-plugin-type-coverage --dev

# Run type coverage
./vendor/bin/pest --type-coverage

# Set minimum type coverage
./vendor/bin/pest --type-coverage --min=100
```

```php
// Type coverage checks for missing types
class UserService
{
    // ❌ Missing return type
    public function getUser($id)
    {
        return User::find($id);
    }

    // ✅ Proper types
    public function createUser(array $data): User
    {
        return User::create($data);
    }
}
```

## Browser Testing (Pest v4)

### Laravel Dusk Integration
```php
use Laravel\Dusk\Browser;

it('displays welcome page', function () {
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
            ->assertSee('Welcome')
            ->assertSee('Laravel');
    });
});

it('can login', function () {
    $user = User::factory()->create();

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/login')
            ->type('email', $user->email)
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/dashboard')
            ->assertSee($user->name);
    });
});
```

## Symfony Integration

### Symfony Test Setup
```php
// tests/Pest.php
use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

pest()->extend(WebTestCase::class)->in('Feature');
```

### HTTP Testing in Symfony
```php
it('displays homepage', function () {
    $client = static::createClient();
    $client->request('GET', '/');

    expect($client->getResponse()->getStatusCode())->toBe(200);
    expect($client->getResponse()->getContent())->toContain('Welcome');
});

it('creates a user via API', function () {
    $client = static::createClient();
    $client->request('POST', '/api/users', [], [], [
        'CONTENT_TYPE' => 'application/json',
    ], json_encode([
        'email' => 'test@example.com',
        'password' => 'password123',
    ]));

    expect($client->getResponse()->getStatusCode())->toBe(201);
    expect($client->getResponse()->getContent())->json()->toHaveKey('id');
});
```

### Doctrine Testing
```php
it('persists user to database', function () {
    $client = static::createClient();
    $entityManager = $client->getContainer()->get('doctrine')->getManager();

    $user = new User();
    $user->setEmail('test@example.com');
    $user->setPassword('hashed_password');

    $entityManager->persist($user);
    $entityManager->flush();

    $repository = $entityManager->getRepository(User::class);
    $savedUser = $repository->findOneBy(['email' => 'test@example.com']);

    expect($savedUser)->not->toBeNull();
    expect($savedUser->getEmail())->toBe('test@example.com');
});
```

## Best Practices

### 1. Test Naming
```php
// ✅ GOOD - Descriptive test names
it('sends welcome email when user is created', function () { });
it('prevents duplicate email registration', function () { });

// ❌ BAD - Vague test names
it('works', function () { });
it('test user', function () { });
```

### 2. Arrange-Act-Assert Pattern
```php
it('updates user profile', function () {
    // Arrange
    $user = User::factory()->create(['name' => 'Old Name']);

    // Act
    $user->update(['name' => 'New Name']);

    // Assert
    expect($user->name)->toBe('New Name');
    expect($user->fresh()->name)->toBe('New Name');
});
```

### 3. Use Factories
```php
// ✅ GOOD - Use factories
$user = User::factory()->create();

// ❌ BAD - Manual creation
$user = new User();
$user->name = 'John';
$user->email = 'john@example.com';
$user->save();
```

### 4. Test One Thing
```php
// ✅ GOOD - Single responsibility
it('validates email format', function () {
    $validator = new EmailValidator();
    expect($validator->isValid('invalid'))->toBeFalse();
});

// ❌ BAD - Testing multiple things
it('validates user', function () {
    // Testing email, password, name all together
});
```

### 5. Use Datasets for Similar Tests
```php
// ✅ GOOD - Use datasets
it('validates email formats', function (string $email, bool $isValid) {
    expect((new EmailValidator())->isValid($email))->toBe($isValid);
})->with([
    ['valid@email.com', true],
    ['invalid', false],
    ['missing@domain', false],
]);

// ❌ BAD - Duplicate tests
it('validates valid email', function () { /* ... */ });
it('validates invalid email', function () { /* ... */ });
it('validates email without domain', function () { /* ... */ });
```

### 6. Aim for High Mutation Score
```php
// Mutation testing reveals weak tests
covers(Calculator::class);

// ❌ Weak test - doesn't verify result
it('adds numbers', function () {
    $calc = new Calculator();
    $calc->add(2, 2);
}); // Mutation: can remove method body

// ✅ Strong test - verifies result
it('adds numbers correctly', function () {
    $calc = new Calculator();
    expect($calc->add(2, 2))->toBe(4);
}); // Mutation: cannot change without failing
```

### 7. Use Architectural Testing
```php
// Enforce architecture rules
arch('services must have tests')
    ->expect('App\Services')
    ->toBeTestedBy('Tests\Unit\Services');

arch('controllers are thin')
    ->expect('App\Http\Controllers')
    ->not->toHaveMethodsWithMoreThan(20)->lines();
```

## Performance Tips

1. **Use parallel testing** for large test suites
2. **Minimize database operations** - use in-memory SQLite for unit tests
3. **Mock external services** - don't call real APIs in tests
4. **Use cached datasets** - for expensive data generation
5. **Profile slow tests** - `./vendor/bin/pest --profile`

## When Helping Users

1. **Ask about their Pest version** - Features differ between v3 and v4
2. **Recommend architectural testing** - Enforce code quality with arch presets
3. **Suggest mutation testing** - Improve test quality beyond coverage
4. **Use datasets wisely** - Reduce test duplication
5. **Implement team management** - Use todo tracking for TDD
6. **Emphasize strict types** - Use type coverage plugin
7. **Write readable tests** - Use descriptive names and organize with describe
8. **Test behavior, not implementation** - Focus on what, not how

Your goal is to help users write elegant, comprehensive, and maintainable tests using PestPHP's latest features and best practices.
