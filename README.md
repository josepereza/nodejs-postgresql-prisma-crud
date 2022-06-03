# nodejs-postgresql-prisma
Repo for Node.js (express) and postgresql database using prisma ORM
## Set Up Express.js
We'll begin by setting up Express.js for our programming languages Quotes API demo project. First, run the following to create a new project:
```
mkdir nodejs-postgres-prisma
cd nodejs-postgres-prisma
npm init -y

```

Next, install Express.js, our web framework for the Quotes REST API:

```
npm install --save express

```
Next, we'll add an index.js file with an API showing that Express is working. Create the file in the root of the project and add the following:

```
//index.js
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;
 
app.get('/', (req, res) => {
  res.json({message: 'alive'});
});
 
app.listen(port, () => {
  console.log(`Listening to requests on port ${port}`);
});

```
Let’s do a quick dissection of what the above code does. First, we initialize an Express app and declare a constant called port. If there is an environment variable called PORT we use it - otherwise, it defaults to 3000.

We add a GET route on the root that responds with a simple JSON. Finally, we start the Express server and listen to the specified port with a message for requests. We run the server with:

```
node index.js

```
Resulting in:

```
Listening to requests on port 3000

```
After that, if we hit http://localhost:3000 on a browser of our choice, it will show us something like the below:


```
#localhost:3000

{"message":"alive"}
```
The code I've shown you up to now is available in a pull request for your reference.

Next up, we'll set up a PostgreSQL database to create our tables for the Quotes API.

## Set Up a Local PostgreSQL Database

There are multiple ways to run a PostgreSQL database on our local machine. If you already have one, you can move on to the next step. If you want to run a PostgreSQL database with Docker, use the following command:

```
docker run --rm --name postgres-quotes -p 5432:5432 -e POSTGRES_PASSWORD=mysecretpassword -d postgres:13-alpine

```
It will give us this output:

```
Unable to find image 'postgres:13-alpine' locally
13-alpine: Pulling from library/postgres
Digest: sha256:ff384947eb9f5939b7fc5ef2ce620fad088999590973f05e6812037d163c770e
Status: Downloaded newer image for postgres:13-alpine
71885633db053d9d70df7e3871595a0dd8be78575fbe0fefc926acd0072e4b5a

```
The docker run command creates a new docker container named postgres-quotes, exposing the container port 5432 to the local port 5432. The --rm parameter is added to remove the container when it stops.

As per the docs, we also set the password to be “mysecretpassword” for the default user postgres. Finally, in the docker run command, we opt to use the 13-alpine image of postgres because it is smaller than the default one.

Our database is up and running. Next up, we will add Prisma to our Node.js project and create our schema.

## Add Prisma ORM to Your Node.js Project
Execute the following command:

```
npm install prisma --save-dev

```
It will install Prisma as a dev dependency and result in output similar to this:

```
 
> prisma@2.26.0 preinstall /path/to/project/nodejs-postgresql-prisma/node_modules/prisma
> node scripts/preinstall-entry.js
 
 
> prisma@2.26.0 install /path/to/project//nodejs-postgresql-prisma/node_modules/prisma
> node scripts/install-entry.js
 
 
> @prisma/engines@2.26.0-23.9b816b3aa13cc270074f172f30d6eda8a8ce867d postinstall /path/to/project//nodejs-postgresql-prisma/node_modules/@prisma/engines
> node download/index.js
 
+ prisma@2.26.0
added 2 packages from 1 contributor and audited 52 packages in 4.319s
found 0 vulnerabilities

```
After that, initialize the Prisma schema with the following command:

```
npx prisma init

```
On initializing the Prisma schema, we will get this output:

```
✔ Your Prisma schema was created at prisma/schema.prisma
  You can now open it in your favorite editor.
 
Next steps:
1. Set the DATABASE_URL in the .env file to point to your existing database. If your database has no tables yet, read https://pris.ly/d/getting-started
2. Set the provider of the data source block in `schema.prisma` to match your database: postgresql, mysql, sqlserver or sqlite.
3. Run prisma db pull to turn your database schema into a Prisma data model.
4. Run prisma generate to install Prisma Client. You can then start querying your database.
 
More information in our documentation:
https://pris.ly/d/getting-started

```
Consequently, we can add our local database connection string in the .env file created. It will look as follows after the change:

```
#.env
# Environment variables declared in this file are automatically made available to Prisma.
# See the documentation for more detail: https://pris.ly/d/prisma-schema#using-environment-variables
 
# Prisma supports the native connection string format for PostgreSQL, MySQL, SQL Server and SQLite.
# See the documentation for all the connection string options: https://pris.ly/d/connection-strings
 
DATABASE_URL="postgresql://postgres:mysecretpassword@localhost:5432/postgres?schema=quotes"


```
The only change here is the DATABASE_URL. For security reasons, it is best to pass the database URL as an environment variable in a production-like environment rather than put the credentials in a file.

## Add Models and Run Prisma Migration
We will open the prisma.schema file in the prisma folder and define our database tables for Quotes and Authors with their relation.

As one author can have multiple quotes and one quote will always have only one author, it will be defined as:

```
#prisma/schema.prisma
 
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema
 
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
 
generator client {
  provider = "prisma-client-js"
}
 
model Author {
  id    Int     @id @default(autoincrement())
  name  String  @unique
  Quotes Quote[]
}
 
model Quote {
  id       Int    @id @default(autoincrement())
  quote    String @unique
  author   Author @relation(fields: [authorId], references: [id])
  authorId Int
}

```
We have added two tables. The first one is the author table with id and name, and the name of the author is unique. The relation is that one author can have one or more quotes.

The following table is the quote table, which has an auto-increment ID and quote that is a unique string. It also has an author id to show which author said the quote.

To convert these models into PostgreSQL database tables, run the following command:

```
npx prisma migrate dev --name init

```
This generates the migration SQL that creates the tables and runs it against the specified database, resulting in this output:

```
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db": PostgreSQL database "postgres", schema "quotes" at "localhost:5432"
 

The following migration(s) have been created and applied from new schema changes:
 
migrations/
  └─ 20210702122209_init/
    └─ migration.sql
 
Your database is now in sync with your schema.
 
Running generate... (Use --skip-generate to skip the generators)
 
> prisma@2.26.0 preinstall /path/to/project/nodejs-postgresql-prisma/node_modules/prisma
> node scripts/preinstall-entry.js
 
 
> prisma@2.26.0 install /path/to/project/nodejs-postgresql-prisma/node_modules/prisma
> node scripts/install-entry.js
 
+ prisma@2.26.0
updated 1 package and audited 52 packages in 1.987s
found 0 vulnerabilities
 
 
> @prisma/client@2.26.0 postinstall /path/to/project/nodejs-postgresql-prisma/node_modules/@prisma/client
> node scripts/postinstall.js
 
+ @prisma/client@2.26.0
added 2 packages from 1 contributor and audited 54 packages in 6.499s
found 0 vulnerabilities
 
 
✔ Generated Prisma Client (2.26.0) to ./node_modules/@prisma/client in 130ms

```
If the migration is successful, it will have installed the Prisma client and added it to our package.json file. It will also create a migration.sql file that looks as shown below:


```
#prisma/migrations/20210702122209_init/migration.sql
-- CreateTable
CREATE TABLE "Author" (
    "id" SERIAL NOT NULL,
    "name" TEXT NOT NULL,
 
    PRIMARY KEY ("id")
);
 
-- CreateTable
CREATE TABLE "Quote" (
    "id" SERIAL NOT NULL,
    "quote" TEXT NOT NULL,
    "authorId" INTEGER NOT NULL,
 
    PRIMARY KEY ("id")
);
 
-- CreateIndex
CREATE UNIQUE INDEX "Author.name_unique" ON "Author"("name");
 
-- CreateIndex
CREATE UNIQUE INDEX "Quote.quote_unique" ON "Quote"("quote");
 
-- AddForeignKey
ALTER TABLE "Quote" ADD FOREIGN KEY ("authorId") REFERENCES "Author"("id") ON DELETE CASCADE ON UPDATE CASCADE;


```
The above database tables match the model that we defined in the prisma.schema file. The code in this part is available as a pull request here.

Below, you can see how the schema looks after importing the generated SQL to dbdiagram.io:

RDBMS Entity Relationship Model for Autor and Quotes

Next, we will seed the database with one author and a couple of quotes from that author.

## How to Seed the PostgreSQL Database
To seed the PostgreSQL database with some initial data, we will create a seed.js file in the same folder where we have our prisma.schema file. Create the file and add the following to it:

```
# prisma/seed.js
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();
 
(async function main() {
  try {
    const martinFowler = await prisma.author.upsert({
      where: { name: 'Martin Fowler' },
      update: {},
      create: {
        name: 'Martin Fowler',
        Quotes: {
          create: [
            {
              quote: 'Any fool can write code that a computer can understand. Good programmers write code that humans can understand.',
            },
            {
              quote: `I'm not a great programmer; I'm just a good programmer with great habits.`,
            },
          ],
        },
      },
    });
 
    console.log('Create 1 author with 2 quotes: ', martinFowler);
  } catch(e) {
    console.error(e);
    process.exit(1);
  } finally {
    await prisma.$disconnect();
  }
})();


```
The above seed file instantiates the Prisma client and then calls an Immediately Invoked Function Expression (IIEF) called main. In the main function, we upsert an author, Martin Fowler, with two amazing quotes.

If there is an error, it is logged, and the process exits. In the case of either success or error, we always disconnect from the database in the finally part of the main function.

To seed the database with the data, run:

```
npx prisma db seed --preview-feature

```
The above command will result in something like:

```
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Running seed from "prisma/seed.js" ...
Result:
Create 1 author with 2 quotes:  { id: 1, name: 'Martin Fowler' }
 
🌱  Your database has been seeded.

```
Read more about Prisma Migrate in the official docs. You can reference the seed changes in this pull request.

Hurray! We have one author and two related quotes from that author in the database. Now, we will expose these quotes in the form of JSON over a REST API.

## API to View Quotes
We added quotes via a REST API endpoint with a GET call. We will change the index.js file we created in a previous step with a Hello world API to add the new GET Quotes API.

Make the following changes to index.js:

```
#index.js
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;
 
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();
 
app.get('/', (req, res) => {
  res.json({message: 'alive'});
});
 
app.get('/quotes', async (req, res) => {
  const currentPage = req.query.page || 1;
  const listPerPage = 5;
  const offset = (currentPage - 1) * listPerPage;
 
  const allQuotes =  await prisma.quote.findMany({
    include: { author: true },
    skip: offset,
    take: listPerPage,
  });
 
  res.json({
    data: allQuotes,
    meta: {page: currentPage}
  });
});
 
app.listen(port, () => {
  console.log(`Listening to requests on port ${port}`);
});


```
The main change here is that we instantiated the Prisma client. We also added the /quotes GET API endpoint, which gets the quotes data with its authors using Prisma’s findMany method. We paginate the quotes with 5 per page — that's why we use skip and take in the findMany parameters. There are other ways to paginate the rows with Prisma. We are opting for the offset-based approach in this example.

At this point, we can rerun the app with:

```
node index.js

```
And if we hit http://localhost:3000/quotes on a browser, we'll see output as shown below:

```
{
  "data": [
    {
      "id": 1,
      "quote": "Any fool can write code that a computer can understand. Good programmers write code that humans can understand.",
      "authorId": 1,
      "author": {
        "id": 1,
        "name": "Martin Fowler"
      }
    },
    {
      "id": 2,
      "quote": "I'm not a great programmer; I'm just a good programmer with great habits.",
      "authorId": 1,
      "author": {
        "id": 1,
        "name": "Martin Fowler"
      }
    }
  ],
  "meta": {
    "page": 1
  }
}

```
It may not be formatted as above, but the data is pulled from the table and served up as JSON with effectively 12 lines of code and no written SQL.

If we use AppSignal, we can also find the exact query and its performance on production. AppSignal has a magic dashboard for Node.js and PostgreSQL as well.

The code for GET Quotes API is available in this pull request.

Now we'll bring in a create Quotes API to add more quotes to our service.

## Introduce a POST API to Add Quotes to PostgreSQL
To be able to add Quotes to the PostgreSQL database, we will introduce a POST API.

We'll add a route in the index.js file. First, add the following line after the express app is defined:

```
#indes.js:4
app.use(express.json());

```
By adding the JSON middleware, express can now parse the JSON sent in the request body. We will add a route to handle additional Quotes as a POST quotes API endpoint, as follows:

```
#index.js:30
app.post('/quotes', async (req, res) => {
  const authorName = req.body.author;
  const quote = {
    quote: req.body.quote
  };
 
  if (!authorName || !quote.quote) {
    return res.status(400).json({message: 'Either quote or author is missing'});
  }
 
  try {
    const message = 'quote created successfully';
    const author = await prisma.author.findFirst({
      where: { name: authorName }
    });
 
    if(!author) {
      await prisma.author.create({
        data: {
          'name': authorName,
          Quotes: {
            create: quote
          }
        }
      });
      console.log('Created author and then the related quote');
      return res.json({message});
    }
 
    await prisma.quote.create({
      data: {
        quote: quote.quote,
        author: { connect: { name: authorName } }
      }
    });
    console.log('Created quote for an existing author');
    return res.json({message});
  } catch(e) {
    console.error(e);
    return res.status(500).json({message: 'something went wrong'});
  }
});


```
The main logic here is to check if both the quote and author are in the request body. If that basic validation passes, we check if the author exists.

If the author does not exist, we create them and relate the quote to the author. If the author exists, we just create the quote and relate it to the existing author using the connect option in Prisma.

If there is an error on the server-side, we send back a 500 response code with a simple message and log the error for our reference. To test it out when the server is running, we will hit the API with the following curl:

```
curl -i -X POST -H 'Accept: application/json' -H 'Content-type: application/json' http://localhost:3000/quotes --data '{"quote":"Before software can be reusable it first has to be usable.","author":"Ralph Johnson"}'

```
It will come back to us with the following output:

```
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 40
ETag: W/"28-5h9zKgCDdv2FIu4KoJVfcy36GpQ"
Date: Sat, 03 Jul 2021 12:28:01 GMT
Connection: keep-alive
Keep-Alive: timeout=5
 
{"message":"quote created successfully"}

```
As the author is not there, the quote creates them. If we try the following curl command:

```
curl -i -X POST -H 'Accept: application/json' -H 'Content-type: application/json' http://localhost:3000/quotes --data '{"quote":"A heuristic we follow is that whenever we feel the need to comment something, we write a method instead.","author":"Martin Fowler"}'

```
It will not create the author, it will just add a quote and relate it to the existing author id 1 with the name Martin Fowler.

I hit the API with some more curl commands, and after adding the sixth quote, I tried http://localhost/quotes?page=2 to test out the pagination. It gave me only one quote — the sixth one I had added, as follows:


```
#localhost:3000/quotes?page=2
```

The code that adds the create Quotes API endpoint is accessible in this pull request.

I would strongly recommend you add the update and delete functionality. The Prisma docs can help you with that.

Please keep in mind that the validation done for this tutorial is super basic to keep things simple. In a real-life project, I recommend that you use a full-on validation library like Joi.

This brings us to wrapping up.

## Wrap-up
We built a Quotes API with Express.js and Prisma ORM, using a PostgreSQL database running on Docker. I hope this gave you some new insights and was a good starting point to explore all of the software used in this tutorial.

Just keep in mind that an ORM is a great tool to reach for when your project is relatively small or mid-sized. In the case of lots of requests, the general nature of an ORM stands in the way of high performance.

As mentioned earlier, using an ORM often involves a trade-off — a balance between convenience and control. Be very careful with what you are leaving behind and make sure what you gain is worth it.

To finish up, I suggest you further explore some great Prisma ORM features. For instance, query optimization is a good one to read about.
