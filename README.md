# The Docker platform

The Docker platform essentially consists of three parts:

- Docker Client: Can run commands by sending them to the Docker daemon, which is either running on the local machine or a remote environment.
- Docker Host: Contains the Docker daemon, images, and containers.
- Docker Registry: Hosts and stores docker images, extensions, and plugins. By default, the public registry Docker Hub will be used to search for images.

Docker images can be thought of as read-only templates and are used to create containers. Images can be based on other images. For example, the mongo image, which contains a MongoDB server, is based on the ubuntu image.

Docker containers are instances of images. They run an operating system with a configured service (such as a MongoDB server on Ubuntu). Additionally, they can be configured, for example, to forward some ports from within the container to the host, or to mount a storage volume in the container that stores data on the host machine. By default, a container is isolated from the host machine, so if we want to access ports or storage from it on the host, we need to tell Docker to allow this.

# Installing Docker

The easiest way to set up the Docker platform for local development is using Docker Desktop. It can be downloaded from the official Docker website (https://www.docker.com/products/docker-desktop/). Follow the instructions to install it and start the Docker engine. After installation, you should have a docker command available in your Terminal.

- Run the following command to verify that it is working properly:

```
docker -v
```

- This command should output the Docker version, like in the following example:

```
Docker version 27.2.0, build 3ab4256
```

After installing and starting Docker, we can move on to creating a container.

# Creating a container

Docker Client can instantiate a container from an image via the docker run command. Let’s now create an ubuntu container and run a shell (/bin/bash) in it:

```
docker run -i -t ubuntu:24.04 /bin/bash
```

- The docker run command does the following:
  - If you have never run a container based on the ubuntu image before, Docker will start by pulling the image from the Docker registry (this is equivalent to executing docker pull ubuntu).
  - After the image is downloaded, Docker creates a new container (the equivalent to executing docker container create).
  - Then, Docker configures a read-write filesystem for the container and creates a default network interface.
  - Finally, Docker starts the container and executes the specified command. In our case, we specified the /bin/bash command. Because we passed the -i (keeps STDIN open) and -t (allocates a pseudo-tty) options, Docker attaches the container’s shell to our currently running Terminal, allowing us to use the container as if we were directly accessing a Terminal on our host machine.

As we can see, Docker is very useful for creating self-contained environments for our apps and services to run in. Later in this book, we are going to learn how to package our own apps in Docker containers. For now, we are only going to use Docker to run services without having to install them on our host system

- Note: The :24.04 string after the image name is called the tag, and it can be used to pin images to certain versions. In this project, we use tags to pull specific versions of images so that the steps are reproducible even when new versions are released. By default, if no tag is specified, Docker will attempt to use the latest tag.

A new shell will open. We can verify that this shell is running in the container by executing the following command to see which operating system is running:

```
uname -a
```

If you get a version number that ends with -linuxkit, you have successfully run a command in the container, because LinuxKit is a toolkit to create small Linux VMs!

You can now type the following command to exit the shell and the container:

```
exit
```

The following figure shows the result of running these commands:

```
    Unable to find image 'ubuntu:24.04' locally
    24.04: Pulling from library/ubuntu
    5a7813e071bf: Download complete
    Digest: sha256:72297848456d5d37d1262630108ab308d3e9ec7ed1c3286a32fe09856619a782
    Status: Downloaded newer image for ubuntu:24.04
    root@028f3e896a24:/# uname -a
    Linux 028f3e896a24 5.15.167.4-microsoft-standard-WSL2 #1 SMP Tue Nov 5 00:21:55 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
    root@028f3e896a24:/# exit
    exit
```

# Introducing MongoDB, a document database

MongoDB, at the time of writing, is the most popular NoSQL database. Unlike Structured Query Language (SQL) databases (such as MySQL or PostgreSQL), NoSQL means that the database specifically does not use SQL to query the database. Instead, NoSQL databases have various other ways to query the database and often have a vastly different structure of how data is stored and queried.

- The following main types of NoSQL databases exist:
  - Key-value stores (for example, Valkey/Redis)
  - Column-oriented databases (for example, Amazon Redshift)
  - Graph-based databases (for example, Neo4j)
  - Document-based databases (for example, MongoDB)

MongoDB is a document-based database, which means that each entry in the database is stored as a document. In MongoDB, these documents are basically JSON objects (internally, they are stored as BSON – a binary JSON format to save space and improve performance, among other advantages). Instead, SQL databases store data as rows in tables. As such, MongoDB provides a lot more flexibility. Fields can be freely added or left out in documents. The downside of such a structure is that we do
not have a consistent schema for documents.

However, this can be solved by using libraries, such as Mongoose.

MongoDB is also based on a JavaScript engine. Since version 3.2, it has been using SpiderMonkey (the JavaScript engine that Firefox uses) instead of V8. Nevertheless, this still means we can execute JavaScript code in MongoDB. For example, we can use JavaScript in the MongoDB Shell to help with administrative tasks.

Again, we must be careful with this, though, as the MongoDB environment is vastly different from a browser or Node.js environment.

# Setting up a MongoDB server

Before we can start using MongoDB, we need to set up a server. Since we already have Docker installed, we can make things easier for ourselves by running MongoDB in a Docker container. Doing so also allows us to have separate, clean MongoDB instances for our apps by creating separate containers.

Let’s get started with the steps:

- Make sure Docker Desktop is running and Docker is started. You can verify this by running the following command, which lists all running containers:

```
docker ps
```

If Docker is not started properly, you will get a Cannot connect to the Docker daemon error. In that case, make sure Docker Desktop is running and the Docker Engine is not paused.

If Docker is started properly, you will see the following output:

```
CONTAINER
ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

If you already have some containers running, it will be followed by a list of started containers.

- Run the following Docker command to create a new container with a MongoDB server:

```
docker run -d --name dbserver -p 27017:27017 --restart unless-stopped mongo:6.0.4
```

The docker run command creates and runs a new container. The arguments are as follows:

    - -d: Runs the container in the background (daemon mode).
    - --name: Specifies a name for the container. In our case, we named it dbserver.
    - -p: Maps a port from the container to the host. In our case, we map the default MongoDB server port 27017 in the container to the same port on our host. This allows us to access the MongoDB server running within our container from outside of it. If you already have a MongoDB server running on that port, feel free to change the first number to some other port, but make sure to also adjust the port number from 27017 to your specified port in the following guides.
    - --restart unless-stopped: Makes sure to automatically start (and restart) the container unless we manually stop it. This ensures that every time we start Docker, our MongoDB server will already be running.
    - mongo: This is the image name. The mongo image contains a MongoDB server.

- Install the MongoDB Shell on your host system (not within the container) by following the instructions on the MongoDB website (https://www.mongodb.com/docs/mongodb-shell/install/).

- On your host system, run the following command to connect to the MongoDB server using the MongoDB Shell (mongosh). After the hostname and port, we specify a database name. We are going to call our database testDB:

```
mongosh mongodb://localhost:27017/testDB
```

You will see some output from the database server, and at the end, we get a shell running on our selected database, as can be seen by the testDB> prompt. Here, we can enter commands to be executed on our database. Interestingly, MongoDB, like Node.js, also exposes a JavaScript
engine, but with yet another different environment. So, we can run JavaScript code, such as the following:

```
testDB> console.log("test")
```

The following figure shows JavaScript code being executed in the MongoDB Shell:

```
    Current Mongosh Log ID: 67ebb1644c948c737fb71235
    Connecting to:          mongodb://localhost:27017/testDB?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.4.2
    Using MongoDB:          8.0.5
    Using Mongosh:          2.4.2

    For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/

    ------
    The server generated these startup warnings when booting
    2025-03-28T19:46:20.466+13:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
    ------

    testDB> console.log("test")
    test

    testDB>
```

Now that we have a shell connected to our MongoDB database server, we can start practicing running commands directly on the database.

# Running commands directly on the database

Before we get started creating a backend service that interfaces with MongoDB, let’s spend some time getting familiar with MongoDB itself via the MongoDB Shell. The MongoDB Shell is very important for debugging and doing maintenance tasks on the database, so it is a good idea to get to know it well.

# Creating a collection and inserting and listing documents

Collections in MongoDB are the equivalent of tables in relational databases. They store documents, which are like JSON objects. To make it easier to understand, a collection can be seen as a very large
JSON array containing JSON objects. Unlike simple arrays, collections support the creation of indices, which speed up the lookup of certain fields in documents. In MongoDB, a collection is automatically
created when we attempt to insert a document into it or create an index for it.

Let’s use the MongoDB Shell to insert a document into our database in the users collection:

- To insert a new user document into the users collection, run the following command in the MongoDB Shell:

```
db.users.insertOne({ username: 'fate', fullName: 'Aye Chan Zaw', age: 38 })
```

Commands that access the database are prefixed with db, then the collection name follows, and finally comes the operation, all separated by periods.
Note: While insertOne() allows us to insert a single document into the collection, there is also an insertMany() method, where we can pass an array of documents to add to the collection.

- We can now list all documents from the users collection by running the following command:

```
db.users.find()
```

Doing so will return an array with our previously inserted document:

```
[
    {
        _id: ObjectId('67ebb2e84c948c737fb71236'),
        username: 'fate',
        fullName: 'Aye Chan Zaw',
        age: 38
    }
]
```

As we can see, MongoDB automatically created a unique ID (ObjectId) for our document. This ID consists of 12 bytes in hexadecimal format (so each byte is displayed as two characters). The bytes are defined as follows:

-     - The first 4 bytes are a timestamp, representing the creation of the ID measured in seconds since the Unix epoch
      - The next 5 bytes are a random value unique to the machine and currently running database process
      - The last 3 bytes are a randomly initialized incrementing counter
  Note: The way ObjectId identifiers are generated in MongoDB ensures that IDs are unique, avoiding ID collisions even when two ids are generated at the same time from different instances, without requiring a form of communication between the instances, which would slow down the creation
  of documents when scaling the database.

# Querying and sorting documents

Now that we have inserted some documents, we can query them by accessing different fields from the object. We can also sort the list of documents returned from MongoDB. Follow these steps:

- Before we get started querying, let’s insert two more documents into our users collection:

```
db.users.insertMany([
{ username: 'jane', fullName: 'Jane Doe', age: 32 },
{ username: 'john', fullName: 'John Doe', age: 30 }
])
```

- Now we can start querying for a certain username by using findOne and passing an object with the username field. When using findOne, MongoDB will return the first matching object:

```
db.users.findOne({ username: 'jane' })
```

Output

```
{
  _id: ObjectId('67ebb4a94c948c737fb71237'),
  username: 'jane',
  fullName: 'Jane Doe',
  age: 32
}
```

- We can also query for full names, or any other field in the documents from the collection. When using find, MongoDB will return an array of all matches:

```
db.users.find({ fullName: 'Jane Doe' })
```

- An important thing to watch out for is that when querying an ObjectId, we need to wrap the ID string with an ObjectId() constructor, as follows:

```
db.users.findOne({ _id: ObjectId('67ebb4a94c948c737fb71237')})
```

Make sure to change the string passed to the ObjectId() constructor to a valid ObjectId returned from the previous commands.

- MongoDB also provides certain query operators, prefixed by $. For example, we can find everyone above the age of 30 in our collection by using the $gt operator, as follows:

```
db.users.find({ age: { $gt: 30 } })
```

You will notice that John Doe does not get returned, because his age is exactly 30. If we want to match ages greater than or equal to 30, we need to use the $gte operator.

- If we want to sort our results, we can use the .sort() method after .find(). For example, we can return all items from the users collection, sorted by age ascending (1 for ascending, -1 for descending):

```
db.users.find().sort({ age: 1 })
```

# Updating documents

To update a document in MongoDB, we combine the arguments from the query and insert operations into a single operation. We can use the same criteria to filter documents as we did for find(). To update a single field from the document, we use the $set operator:

- We can update the age field for the user with the username jane as follows:

```
db.users.updateOne({ username: 'jane' }, { $set: { age: 24 } })
```

Note: Just like findOne, updateOne only updates the first matching document. If we want to update all documents that match, we can use updateMany.

MongoDB will return an object with information about how many documents matched (matchedCount), how many were modified (modifiedCount), and how many were upserted (upsertedCount).

- The updateOne method accepts a third argument, which is an options object. One useful option here is the upsert option, which, if set to true, will insert a document if it does not exist yet, and update it if it does exist already. Let’s first try to update a non-existent user with upsert: false:

```
db.users.updateOne({ username: 'new' }, { $set: { fullName: 'New User' } })
```

Now we set upsert to true, which inserts the user:

```
db.users.updateOne({ username: 'new' }, { $set: { fullName: 'New User' } }, { upsert: true })
```

Note: If you want to remove a field from a document, use the $unset operator. If you want to replace the whole document with a new document, you can use the replaceOne method and pass a new document as the second argument to it.

# Deleting documents

To delete documents from the database, MongoDB provides the deleteOne and deleteMany methods, which have a similar API to the updateOne and updateMany methods. The first argument is, again, used to match documents.

Let’s say the user with the username new wants to delete their account. To do so, we need to remove them from the users collection. We can do so as follows:

```
db.users.deleteOne({ username: 'new' })
```

That’s all there is to it! As you can see, it is very simple to do CRUD operations in MongoDB if you already know how to work with JSON objects and JavaScript, making it the perfect database for a Node.js backend.

# Accessing the database via VS Code

Up until now, we have been using the Terminal to access the database. If you remember, in Chapter 1, Preparing for Full-stack Development, we installed a MongoDB extension for VS Code. We can now use this extension to access our database in a more visual way:

- Click on the MongoDB icon on the left sidebar (it should be a leaf icon) and click on the Add Connection button:
- A new MongoDB tab will open up. In this tab, click on Connect in the Connect with Connection String box:
- A popup should open at the top. In this popup, enter the following connection string to connect to your local database:

```
mongodb://localhost:27017/
```

- Press Return/Enter to confirm. A new connection will be listed in the MongoDB sidebar. You can browse the tree to view databases, collections, and documents. For example, click the first document to view it:

Tip: You can also directly edit a document by editing a field in VS Code and saving the file. The updated document will automatically be saved to the database.

The MongoDB extension is very useful for debugging our database, as it lets us visually spot problems and quickly make edits to documents. Additionally, we can right-click on Documents and Search for documents… to open a new window where we can run MongoDB queries, just like we did in the Terminal. The queries can be executed on the database by clicking on the Play button in the top right. You may need to confirm a dialog with Yes, and then the results will show in a new pane.

# Accessing the MongoDB database via Node.js

We are now going to create a new web server that, instead of returning users from a JSON file, returns the list of users from our previously created users collection:

- In the testDB folder, open a Terminal. Install the mongodb package, which contains the official MongoDB driver for Node.js:

```
npm install mongodb@6.15.0
```

- Create a new backend/mongodbweb.js file and open it. Import the following:

```
import { createServer } from 'node:http'
import { MongoClient } from 'mongodb'

const url = 'mongodb://localhost:27017/'
const dbName = 'testDB'
const client = new MongoClient(url)

try {
  await client.connect()
  console.log('Successfully connected to database!')
} catch (err) {
  console.error('Error connecting to database:', err)
}

const server = createServer(async (req, res) => {
  const db = client.db(dbName)
  const users = db.collection('users')
  const usersList = await users.find().toArray()
  res.statusCode = 200
  res.setHeader('Content-Type', 'application/json')
  res.end(JSON.stringify(usersList))
})

const host = 'localhost'
const port = 3000
server.listen(port, host, () => {
  console.log(`Server listening on http://${host}:${port}`)
})

```

- Run the server by executing the script:

```
node backend/mongodbweb.js
```
