# Lab Two

## Context

Lab Two is about understanding the basics of Laravel: the folder structure, how the web server responds to requests, basic MVC architecture that the framework implements, and how to add a new "page" in a project like this and see results in the browser.

## Acceptance Criteria
Create a model, controller, route, and view using the following steps:
1. Follow the above local setup instructions to get the project running
2. Create a feature branch for your work `git checkout -b your-feature-branch`
3. Run `./vendor/bin/sail artisan make:model Pet -c` to generate a Pet model and controller
4. In `app/Models/Pet.php`: 
   1. add the following code to the class definition, which is how we define model attributes (more on this when we talk about Eloquent):
        ```
        protected $fillable = [
            'name',
            'age',
            'type'
        ];
        ```
   2. Add a public `toString()` function that outputs the fillable properties into a descriptive string about the pet
      - e.g. `"$this->name is a $this->age year old $this->type."`
5. Create the view `resources/views/pets.blade.php`
   1. Make sure it is a valid html5 document
   2. Add the following code to display the list of pets, calling the `toString()` method you added in step 3:
      ```
      @foreach($pets as $pet)
        <li>{{$pet->toString()}}</li>
      @endforeach
      ```
6. In `app/Http/Controllers/PetController` create a `show()` controller action method
   1. It should return the `pets` view created in step 4
   2. It should return an array of pets which you can hardcode in this class
   3. Use `Pet::make(['name' => 'Salem', 'age' => 8, 'type' => 'dog'])` to create your pets
7. In `routes/web.php` define a GET route that uses the `PetController::show` action
8. Commit your changes: `git add -A && git commit -m "your commit message"`
9. Push your feature branch to Github: `git push origin your-feature-branch`
10. Create a pull request in Github
