# Database: Seeding

- [Introdução](#introduction)
- [Escrevendo Seeders](#writing-seeders)
    - [Usando Model Factories](#using-model-factories)
    - [Calling Additional Seeders](#calling-additional-seeders)
- [Running Seeders](#running-seeders)

<a name="introduction"></a>
## Introdução

O Laravel inclui um método simples de criar seu banco de dados com dados de testes usando classes _seeder_. Todas as classes _seeder_ são armazenadas no diretório `database/seeds`. As classes de semente podem ter qualquer nome que você deseja, mas provavelmente deve seguir uma convenção sensata, como `UsersTableSeeder`, etc. Por padrão, uma classe `DatabaseSeeder` está definida para você. Nesta classe, você pode usar o método `call` para executar outras classes _seeder_, permitindo que você controle a ordem da execução.

<a name="writing-seeders"></a>
## Escrevendo Seeders

Para gerar uma classe _seeder_, execute o comando do [Artisan](docs/{{version}}/artisan) `make:seeder`. Todas as _seeders_ geradas pelo Laravel serão colocados no diretório `database/seeds`:

    php artisan make:seeder UsersTableSeeder

Uma classe _seeder_ por padrão contém apenas um método: `run`. Esse método é chamado quando o comando do [Artisan](/docs/{{version}}/artisan) `db:seed` é executado. Dentro do método `run`, você pode inserir dados no seu banco de dados, como quiser. Você pode usar o [query builder](/docs/{{version}}/queries) para inserir dados manualmente ou pode usar [Eloquent model factories](/docs/{{version}}/database-testing#writing-factories).

> {tip} [Mass assignment protection](/docs/{{version}}/eloquent#mass-assignment) é desativada automaticamente durante a execução das seeders.

Como um exemplo, vamos modificar a classe padrão `DatabaseSeeder` e adicionar uma instrução de inserção do banco de dados no método `run`:

    <?php

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeds.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }

<a name="using-model-factories"></a>
### Usando Model Factories

Claro, especificar manualmente os atributos para cada seeder é complicado. Em vez disso, você pode usar [model factories](/docs/{{version}}/database-testing#writing-factories) para gerar, grandes quantidades de registros de banco de dados. Primeiro reveja a [documentação do model factory documentation](/docs/{{version}}/database-testing#writing-factories) para aprender como criar suas factories. Uma vez que você definiu suas factories, você pode usar a função helper `factory` para inserir registros no seu banco.

Por exemplo, crie 50 usuários e atribua um relacionamento a cada usuário:

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function ($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }

<a name="calling-additional-seeders"></a>
### Calling Additional Seeders

Within the `DatabaseSeeder` class, you may use the `call` method to execute additional seed classes. Using the `call` method allows you to break up your database seeding into multiple files so that no single seeder class becomes overwhelmingly large. Simply pass the name of the seeder class you wish to run:

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UsersTableSeeder::class,
            PostsTableSeeder::class,
            CommentsTableSeeder::class,
        ]);
    }

<a name="running-seeders"></a>
## Running Seeders

Once you have written your seeder, you may need to regenerate Composer's autoloader using the `dump-autoload` command:

    composer dump-autoload

Now you may use the `db:seed` Artisan command to seed your database. By default, the `db:seed` command runs the `DatabaseSeeder` class, which may be used to call other seed classes. However, you may use the `--class` option to specify a specific seeder class to run individually:

    php artisan db:seed

    php artisan db:seed --class=UsersTableSeeder

You may also seed your database using the `migrate:refresh` command, which will also rollback and re-run all of your migrations. This command is useful for completely re-building your database:

    php artisan migrate:refresh --seed
