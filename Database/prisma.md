### Initialize the Database

1. Install the Prisma CLI.
   `npm install prisma --save-dev`
1. Initialize Prisma to use sqlite.
   `npx prisma init --datasource-provider sqlite`
3. In the generated `.env` file, set `DATABASE_URL` to `"file:<dev.db>"`.
4. Add models to your `schema.prisma` file according to the database schema above.
```schema.prisma
// schema.prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
} 

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}  

//By default the field is required, if you want an attribute to be optional put a '?' at the end for example: 'String?'
model Message {
  id        Int    @id @default(autoincrement())
  message   String 
}
```
5. Create and run the initial migration.
   `npx prisma migrate dev --name init`
6. Explore the created database. You should see two empty models: `Author` and `Book`.\
   `npx prisma studio`
7. If you made a mistake in your `schema.prisma`, instead of running another migration, you can instead use [`db push`](https://www.prisma.io/docs/guides/migrate/prototyping-schema-db-push) to sync your database with the schema. This is useful while _prototyping_.
   `npx prisma db push`

### Seed the Database

1. Install Prisma Client, which we will use to interact with the database.
   `npm install @prisma/client`
1. Create and export a new `PrismaClient` in `prisma/index.js`.
   ```js
   //index.js
   const { PrismaClient } = require('@prisma/client');
   const prisma = new PrismaClient();
   module.exports = prisma;
   ```
1. In `prisma/seed.js`, seed 20 authors into the database. Each author should have 3 corresponding books. Refer to [the docs on how to create related records](https://www.prisma.io/docs/concepts/components/prisma-client/relation-queries#create-a-related-record).
   ```js
   //seed.js
   
   const prisma = require('../prisma');
   const seed = async () => {
     try {
       await prisma.message.create({
         data: {
           message: "Hello World!"
         },
       });
     // Returns error in a readable format
     } catch (error) {
       console.eerror(`Error creating message: ${error}`)
     }
   };
   
   seed()
     .then(async () => await prisma.$disconnect())
     .catch(async (e) => {
       console.error(e);
       await prisma.$disconnect();
       process.exit(1);
     });
   ```
1. Update `package.json` to configure Prisma's integrated seeding functionality.
   ```json
   //package.json
   
   "prisma": {
     "seed": "node prisma/seed.js"
   }
   ```
1. Use Prisma Migrate to completely reset and seed the database.
   `npx prisma migrate reset`
   - Note: this is designed to be used in _development_ only! Another option is `npx prisma db seed`, but that will not clear existing data. `reset` is simpler to use (for now).
1. Confirm that the database is correctly seeded with authors and books.
   `npx prisma studio`

###Serve using Express

1. Install express using the CLI\ 
   `npm install express`
2. Create a server.js file 
3. In your server.js file add this code to start a server and generate routes
```javascript
//server.js

require("dotenv").config();
const path = require("path");
const express = require("express");
const morgan = require("morgan");
const { createServer: createViteServer } = require("vite");

const PORT = process.env.PORT ?? 3000;

/**
 * The app has to be created in a separate async function
 * since we need to wait for the Vite server to be created
 */
const createApp = async () => {
  const app = express(); 

  // Logging middleware
  app.use(morgan("dev")); 

  // Body parsing middleware
  app.use(express.json());
  app.use(express.urlencoded({ extended: true }));  

  // API routes
  app.use("/api", require("./api"));

  // Add any aditional routes these are just examples but need to be changed depending on your folder/file structure
  app.use"/api/hello", require("./api/hello")
  
  // Serve static HTML in production & Vite dev server in development
  if (process.env.NODE_ENV === "production") {
    app.use(express.static(path.resolve(__dirname, "../../dist/")));
  } else {
    // Pulled from https://vitejs.dev/config/server-options.html#server-middlewaremode
    const vite = await createViteServer({
      server: { middlewareMode: true },
    });
    
    app.use(vite.middlewares);
  }
  // Simple error handling middleware
  app.use((err, req, res, next) => {
    console.error(err);
    res.status(err.status ?? 500).send(err.message ?? "Internal server error.");
  }); 

  app.listen(PORT, () => {
    console.log(`Server listening on port ${PORT}.`);
  });
};
```
4. Create a router file. This example displays "Hello World!" from
```javascript
//hello.js

const { ServerError } = require("../errors");
const prisma = require("../prisma");
const router = require("express").Router();
module.exports = router;  

// /api/hello - GET message that displays Hello World!
router.get('/', async (req, res, next) => {
  try {
    // for testing that path can be reached:
    // console.log('You have reached /api/message page')
    // res.send('You have reached the message page!')

    // for actual GET request for message:
    const message = await prisma.message.findUnique({
      where: {
        message: "Hello World!"
      }
    });
    res.json(message);
    // For additional CRUD operations refer to
    // https://www.prisma.io/docs/concepts/components/prisma-client/crud
  } catch (err) {
    next(err);
  }
});
```

###Using Faker Data
1. install faker 
   `npm install @faker-js/faker`
2. If you want to use data imported from faker.io use this code in your seed.js
```js
//seed.js

// This section is required
const prisma = require("../prisma");
const { faker } = require('@faker-js/faker');

  

//**seeds the database with 5 students randomly generated using faker data */

const seed = async (numStudents =5) => {
  try {
    for (let i = 0; i < numStudents; i++) {
      // Variables for faker data
      const generatedFirstName = faker.person.firstName();
      const generatedLastName = faker.person.lastName();
      const generatedEmail = faker.internet.email();
      const generatedGpa = faker.number.int({ min: 0, max: 4, precision: 0.01 });

      // Checks if faker data does not return null
      if (!generatedFirstName || !generatedLastName || !generatedEmail) {
        console.error(`Invalid data for student ${i + 1}. Skipping.`);
        continue;
      } 

      // Creates a database entry using the data supplied by faker.
      // Refer to you database schema to assign appropriate fields.
      await prisma.student.create({
        data: {
          firstName: generatedFirstName,
          lastName: generatedLastName,
          email: generatedEmail,
          gpa: generatedGpa,
        },
      });
    } 

  // Returns errors in a readable format
  } catch (error) {
    console.error(`Error creating student: ${error}`);
  }
}  

seed()
  .then(async () => await prisma.$disconnect())
  .catch(async (err) => {
    console.error(err);
    await prisma.$disconnect();
    process.exit(1);
  });
```
