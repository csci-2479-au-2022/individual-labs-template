# Lab Three

## Context
Tests, while they may seem daunting and unapproachable, are a critical piece of the development lifecycle, because they ensure that our app works as we developers expect it to. Every new feature should include tests to verify that the code is doing what it was meant to do.

This lab will give you some experience writing new tests, running tests, and inspecting a code coverage report so you can see what code in the app is and is not covered adequately.

## Getting your project up to speed
In Lab Two we did not add the ability to view details for a specific pet. To do this:
1. Add an optional id route parameter to the existing /pets route, like `/pets/{id?}`
2. Add an optional id parameter to the `PetController::show` method like `public function show(?int $id = null)`
3. Create an `App\Services\PetService` that your PetController will depend on to provide the pet data
    1. Move the existing `PetController::getPets` method into this service, i.e. the one that returns the array of pets using `[ Pet::make(...) ]`
    2. Add a new `getPetById` method to PetController that does the following:
       - calls the existing `PetController::getPets()` method
       - for each pet in the pets array, check if the pet->id matches the id in the route parameter
       - if there is a match, return the pet
       - if none of the pets match, `throw new Symfony\Component\HttpKernel\Exception\NotFoundHttpException()`
6. In the pet list blade template, each pet in the list should have a link to view details: `<a href="/pets/{{$pet->id}}">...</a>`
7. Create a new pet-details blade template that contains the following:
   ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Lab 3</title>
    </head>
    <body>
        <h1>Welcome to Lab 3</h1>
        <nav>
            <p><a href="/pets">Back to list</a></p>
        </nav>
        <h2>{{$pet->name}}</h2>
        <p>{{$pet->name}} is a {{$pet->age}} year old {{$pet->type}}.</p>
    </body>
    </html>
   ```
8. The `PetController::show` method should now:
   1. return the pet-details view if an id parameter is passed, using the `PetService::getPetById` method to get the view model data
   2. otherwise it should return the pet-list view, using `PetService::getPets` to get the view model data

## Running tests
1. Normal: `sail artisan test`
2. Debug (so you can set a breakpoint in a test): `sail debug test`
3. To see a code coverage report in the terminal: `sail artisan test --coverage`

## Writing tests
The TestCase class has a lot built into it, both from Laravel and PHPUnit. The general formula for writing tests in any framework is Arrange, Act, Assert:
1. Arrange
   - Define any expected data
   - Setup any spies (mocked functions that can track behavior), using:
        ```
        $spy->shouldReceive('expectedMethod')
            ->with($arg)
            ->once() // number of times method should be called
            ->andReturn($expectedValue);
        ```
2. Act
   - Call whatever method should trigger the code under test. In our case it's calling a route, which our test class can do: `$this->get('/pets')`
3. Assert
   - Check that actual values match our expectations:
     - `$this->assertViewHas('key', $value)`
     - `$this->assertStatus(200)`

## Lab Instruction Steps
1. Verify you can run the project on localhost, and get from home -> pet list -> pet details
   - Note that petlist and pet details are served by the same route: `/pets/{id?}`
   - Note the logic in the PetController to handle both scenarios (with ID and without)
2. Verify you can run tests: `sail artisan test`
3. Create a new branch for your work
4. Create a new feature test to test the `PetController` class: `sail artisan make:test PetControllerTest`, and add the following:
   - Create a private static function `getPets()` that will return some mock pet data, e.g.:
        ```
        return [
            Pet::make([
                'id' => 1,
                'name' => 'Fido',
                'age' => 5,
                'type' => Pet::PET_TYPE_DOG,
            ]),
            Pet::make([
                'id' => 2,
                'name' => 'Milo',
                'age' => 3,
                'type' => Pet::PET_TYPE_CAT,
            ]),
        ];
        ```
    - Create private properties `array $pets` and `MockInterface $petServiceSpy`
    - Create a protected setUp function that will set `$pets = self::getPets()` and spy on PetService: `$this->spy(PetService::class)`
5. Create tests for:
   - `getPets` without an ID param returns a list of pets
   - `getPets` with a valid ID param returns a single pet
   - `getPets` with an invalid ID param returns a 404 response
6. Create a new unit test to test the `Pet` model: `sail artisan make:test PetTest`
7. Create a test for verifying that calling `toString()` returns the expected string
   - Arrange: define expected string value, initialize a `$pet` object
   - Act: call `$pet->toString()` to get actual string value
   - Assert expected and actual strings match: `$this->assertEquals($expectedString, $actualString)`
8. Verify your tests pass: `sail artisan test`
9. Verify your code coverage looks good: `sail composer test:coverage`
   - At least on `PetController` and `Pet` model, they should now have 100% coverage
   - Do not worry about the rest of the coverage
10. Commit your work to your branch and submit a pull request
    - assign Andrew as the reviewer
