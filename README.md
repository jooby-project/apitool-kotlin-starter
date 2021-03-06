[![Build Status](https://travis-ci.org/jooby-project/apitool-kotlin-starter.svg?branch=master)](https://travis-ci.org/jooby-project/apitool-kotlin-starter)
# apitool

Starter project for [apitool](http://jooby.org/doc/apitool/) module.

## screenshots

### swagger
![Swagger Api Tool](http://jooby.org/resources/images/apitool-swagger.png)

### raml
![RAML Api Tool](http://jooby.org/resources/images/apitool-raml.png)

## quick preview

This project contains a simple application that:

- Defines a Pet API using script routes
- Export API to [Swagger](https://swagger.io) and [RAML](https://raml.org)

```kotlin
/**
 * Kotlin ApiTool.
 */
class App : Kooby({

    /** JSON: */
    use(Jackson())

    /** Database: */
    use(Jdbc())
    use(Flywaydb())
    use(Jdbi3()
            .doWith { jdbi ->
                jdbi.installPlugin(SqlObjectPlugin())
                jdbi.installPlugin(KotlinPlugin())
                jdbi.installPlugin(KotlinSqlObjectPlugin())
            }
            /** Simple transaction per request and bind the PetRepository to it:  */
            .transactionPerRequest(
                    TransactionalRequest()
                            .attach(PetRepository::class.java)
            ))

    /** Export API to Swagger and RAML: */
    use(ApiTool()
            .filter { r -> r.pattern().startsWith("/api") }
            .swagger()
            .raml())

    // Home page redirect to Swagger:
    get { Results.redirect("/swagger") }

    /**
     * Everything about your Pets.
     */
    path("/api/pet") {
        /**
         * List pets ordered by id.
         *
         * @param start Start offset, useful for paging. Default is `0`.
         * @param max Max page size, useful for paging. Default is `20`.
         * @return Pets ordered by name.
         */
        get {
            val db = require(PetRepository::class)

            val start = param("start").intValue(0)
            val max = param("max").intValue(20)

            db.list(start, max)
        }

        /**
         * Find pet by ID
         *
         * @param id Pet ID.
         * @return Returns `200` with a single pet or `404`
         */
        get("/:id") {
            val db = require(PetRepository::class)

            val id = param<Long>("id")

            val pet = db.findById(id) ?: throw Err(Status.NOT_FOUND)

            pet
        }

        /**
         * Add a new pet to the store.
         *
         * @param body Pet object that needs to be added to the store.
         * @return Returns a saved pet.
         */
        post {
            val db = require(PetRepository::class)

            val pet = body<Pet>()

            val id = db.insert(pet)

            Pet(id, pet.name)
        }

        /**
         * Update an existing pet.
         *
         * @param body Pet object that needs to be updated.
         * @return Returns a saved pet.
         */
        put {
            val db = require(PetRepository::class)

            val pet = body<Pet>()
            if (!db.update(pet)) {
                throw Err(Status.NOT_FOUND)
            }
            pet
        }

        /**
         * Deletes a pet by ID.
         *
         * @param id Pet ID.
         * @return A `204`
         */
        delete("/:id") {
            val db = require(PetRepository::class)

            val id = param<Long>("id")

            if (!db.delete(id)) {
                throw Err(Status.NOT_FOUND)
            }
            Results.noContent()
        }
    }
})

```

## run

    mvn jooby:run

Try:

- http://localhost:8080/swagger
- http://localhost:8080/raml

## build

    mvn clean package

The `apitool` plugin generates an `App.json` file to keep documentation available at deploy time.

```
[INFO] --- jooby-maven-plugin:apitool (default) @ apitool-starter ---
[INFO] API file: apitool-starter/target/classes/starter/apitool/App.json
``` 

## help

* Read the [module documentation](http://jooby.org/doc/apitool)
* Join the [channel](https://gitter.im/jooby-project/jooby)
* Join the [group](https://groups.google.com/forum/#!forum/jooby-project)
