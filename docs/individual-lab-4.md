# Lab Four

## Context
You may be familiar with creating databases and data/model layers in applications, but perhaps have used a more manual approach. For instance, in 2412 we created a database using an SQL file, and did CRUD operations in PHP by writing raw SQL strings within our PHP code. This is a "quick and dirty" method that can work in a pinch, but it's not great for scaling larger applications with more complex business logic, and is prone to errors.

This lab will give you some experience creating a data layer using a code-first approach thanks to Laravel's built-in migration features and Eloquent object-relational mapper (ORM). The migrations will allow us to maintain our database schema as it changes over time, and the ORM will allow us to use PHP methods to access our data records, without worrying too much about the SQL queries being executed under the hood.

## Objective
In our PetController we are returning raw data currently, e.g. `['name' => 'Salem', 'age' => 9, 'type' => 'dog']`. We want a database to store this information in, and we have a one-to-many relationship between the type of pet (dog) and the pet (Salem, Merlin, etc). We nee to create our database tables with migrations, and use Eloquent models in our code to refer to those tables.

## Running migrations
Be aware that you can always run `sail artisan migrate:fresh --seed` to start over from scratch with your tables and seeders. This command drops all tables and runs all migrations starting from the oldest timestamp in the migration file names.

Also know that running `sail artisan migrate` will run any new migrations in your project that have not been ran on your database.

## Lab Instruction Steps
1. Verify you can run the project on localhost
2. Delete the existing Pet model and PetController from the previous lab
3. Create models for our pets and pet_types tables with migrations, seeders, factories, controllers
   - `sail artisan make:model Pet -mfsc`
   - add a fillable property to Pet:
     ```
      protected $fillable = ['name', 'age'];
     ```
   - repeat for PetType: `sail artisan make:model PetType -mfsc`
   - add a fillable property to PetType:
     ```
      protected $fillable = ['name'];
     ```
4. Add table colums in the respective migration file (the tables are already there):
   1. pet_types:
      - `$table->string('name');`
      - `$table->string('code', 4);`
   3. pets:
      - `$table->string('name');`
      - `$table->integer('age')->unsigned();`
      - `$table->foreignId('pet_type_id')->constrained()` which is a foreign key to the pet_types table
5. Run your migrations and verify they are successful (connect to your database via Workbench or `sail artisan db` to see your tables)
   - `sail artisan migrate`
   - you should see the pets and pet_types tables
6. Setup your [model relationships](https://laravel.com/docs/9.x/eloquent-relationships#defining-relationships):
   1. On the PetType and Pet models, add a one-to-many relationship
      - on Pet, add:
      ```
        public function petType()
        {
            return $this->belongsTo(PetType::class);
        }
      ```
      - on PetType, add:
      ```
        public function pets()
        {
            return $this->hasMany(Pet::class);
        }
      ```
6. Use the seeders to populate some data:
   1. Add the model seeders we generated to the main DatabaseSeeder:
   ```
    public function run()
    {
        $this->call([
            PetTypeSeeder::class,
            PetSeeder::class,
        ]);
    }
   ```
   3. Add some pet types to the PetTypeSeeder (dog, cat, gerbil, komodo dragon, etc)
   ```
    $factory = PetType::factory();
    $factory->create(['name' => 'dog']);
    $factory->create(['name' => 'cat']);
    // etc...
   ```
   5. In the PetSeeder, add some pets with a relationship to pet type:
      1. Get the pet type you want to use:
      ```
        $dog = PetType::where('name', 'dog')->first();
      ```
      3. Create some pets attached to the pet type record:
      ```
        $dog->pets()->createMany([
            [
                'name' => 'Salem',
                'age' => 9,
            ],
            // could add additional pets here
        ]);
      ```
   6. Add some logic in the PetController to render the view with pets from the database (Yes, we are being lazy in this lab and not doing this in a service layer)
   ```
    public function show()
    {
        return view('pets', [
            'pets' => self::getPets(),
        ]);
    }

    private static function getPets()
    {
        return Pets::all();
    }
   ```
   7. Make sure in the pets view we refer to the parent record's (i.e. PetType) name property:
   ```
    @foreach($pets as $pet)
    <li>{{$pet->name}} is a {{$pet->petType->name}} who is {{$pet->age}} years old</li>
    @endforeach
   ```
   8. View your handiwork in a browser
10. Commit your work to your branch and submit a pull request
    - assign Andrew as the reviewer
