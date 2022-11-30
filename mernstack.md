# MERN Stack on Ubuntu 20.04 

The MERN stack is an acronym for MongoDB, Express, React / Redux, and Node.js. The MERN stack is one of the most popular JavaScript stacks for building modern single-page web applications (SPA).

In this tutorial, we'll build a simple ToDo application that uses a RESTful API.

### Prerequisites
1. An Ubuntu VM (In this tutorial we're using an Ubuntu 20.04 on AWS VM Instance
   
## STEP 1 – BACKEND CONFIGURATION

Update ubuntu

`$sudo apt update`

Upgrade ubuntu

`$sudo apt upgrade`

Let’s get the location of Node.js software from Ubuntu repositories.

`$curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`

Install Node.js on the server

`$sudo apt-get install -y nodejs`

>Note: The command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.

Verify the node installation with the command below

`$node -v`

```
Output
v18.12.1
```
Verify the node installation with the command below

`$npm -v `

```
Output
8.19.2
```

### Application Code Setup

Create a new directory for your To-Do project:

`$mkdir Todo`

Run the command below to verify that the Todo directory is created with ls command

`$ls`

>You can learn more about different useful keys for ls command with ls --help.

Now change your current directory to the newly created one:

`$cd Todo`

 Initialise your project, so that a new file named **package.json** will be created. This file will normally contain information about your application and the dependencies that it needs to run.

 `$npm init`
 
  Follow the prompts after running the command. You can press **Enter** several times to accept default values, then accept to write out the package.json file by typing & **yes**.

 To confirm that you have package.json file created.

 `$ls`

`$cat package.json`

```
Output
{
  "name": "todo",
  "version": "1.0.0",
  "description": "A simple To-Do App",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Phoenix",
  "license": "ISC"
}
```

### Install ExpressJS

Express is a framework for Node.js, therefore a lot of things developers would have programmed is already taken care of out of the box. Therefore, it simplifies development, and abstracts a lot of low-level details. 
>For example, Express helps to define routes of your application based on HTTP methods and URLs.

To use express, install it using npm:

`$npm install express`

Now create a file index.js with the command below

`$touch index.js`

Run `ls` to confirm that your index.js file is successfully created

```
Output
index.js  node_modules  package-lock.json  package.json
```

Install the dotenv module

`$npm install dotenv`

Open the index.js file with the command below

`$vim index.js`

Type the code below into it and save. Do not get overwhelmed by the code you see. For now, simply paste the code into the file.

```
const express = require('express');
require('dotenv').config();
 
const app = express();
 
const port = process.env.PORT || 5000;
 
app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});
 
app.use((req, res, next) => {
res.send('Welcome to Express');
});
 
app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

```

Notice that we have specified to use **port 5000** in the code. This will be required later when we go on the browser.
Use: **w** to save in vim and use: **qa** to exit vim

Now it is time to start our server to see if it works. Open your terminal in the same directory as your index.js file and type:

`$node index.js`

If everything goes well, you should see Server running on port 5000 in your terminal.

![](assets/1.PNG)

Now we need to open this port in EC2 Security Groups.

![](assets/3.png)

Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000:

>http://PublicIP-or-PublicDNS:5000>


![](assets/4.png)

## Routes
There are three actions that our To-Do application needs to be able to do:
1. Create a new task
2. Display list of all tasks
3. Delete a completed task

>Each task will be associated with some particular endpoint and will use different standard **HTTP request methods**: ***POST, GET, DELETE.***

For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes:

`$mkdir routes`

>Tip: You can open multiple shells in Putty or Linux/Mac to connect to the same EC2

Change directory to routes folder.

`$cd routes`

Now, create a file api.js with the command below

`$touch api.js`

Open the file with the command below

`$vim api.js`

Copy below code in the file.

```
const express = require ('express');
const router = express.Router();
 
router.get('/todos', (req, res, next) => {
 
});
 
router.post('/todos', (req, res, next) => {
 
});
 
router.delete('/todos/:id', (req, res, next) => {
 
})
 
module.exports = router;
```
Moving forward let create Models directory.

### MODELS
Now comes the interesting part, since the app is going to make use of Mongodb which is a NoSQL database, we need to create a model.
>A model is at the heart of JavaScript based applications, and it is what makes it interactive.

We will also use models to define the **database schema**.
This is important so that we will be able to define the fields stored in each Mongodb document. 

>To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.

Change directory back Todo folder with **cd ..** and install Mongoose

`$npm install mongoose`

Create a new folder models :

`$mkdir models`

Change directory into the newly created ‘models’ folder with

`$cd models`

Inside the models folder, create a file 

`$touch todo.js`

>Tip: All three commands above can be defined in one line to be executed consequently with help of && operator, like this:
`mkdir models && cd models && touch todo.js`

Open the file created with vim todo.js then paste the code below in the file:

```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
 
//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})
 
//create model for todo
const Todo = mongoose.model('todo', TodoSchema);
 
module.exports = Todo;
```
Now we need to update our routes from the file **api.js** in **‘routes’** directory to make use of the new model.
In Routes directory, open **api.js** with 

`$vim api.js`

>delete the code inside with ***:%d*** command and paste there code below into it then save and exit

```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');
 
router.get('/todos', (req, res, next) => {
 
//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});
 
router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});
 
router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})
 
module.exports = router;
```

The next piece of our application will be the MongoDB Database



