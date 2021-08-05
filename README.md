# Two Model CRUD App - No relationship - First Model

For this activity we'll be creating a Blog App with Express. In our blog we'll be able to view different authors, and view the articles that they have written.

## Lesson Objectives

1. Init Directory
1. Start express
1. Create Home page
1. Create Authors Index
1. Create Authors New Page
1. Set up Author Model
1. Create Authors Post Route
1. Show Authors on Index Page
1. Create Authors Show Page
1. Create Authors Delete Route
1. Create Authors Edit Page
1. Create Authors Put Route

## Init Directory

1. `mkdir express-blog`
1. `cd express-blog`
1. `touch server.js`
1. `npm init`
    - make entry point `server.js`
1. `npm install express`
1. Add git to your project with `git init`
1. `touch .gitignore`
1. Add node_modules in your `.gitignore` file

## Start Express

server.js:

```javascript
const express = require('express');
const app = express();
const PORT = 4000;

app.listen(PORT, () => {
	console.log(`Server running on port ${PORT}! 👟`);
});
```

## Create Home page

We'll start by setting up the first page of our app, the homepage.

1. `npm install ejs`
1. `mkdir views`
1. `touch views/index.ejs`

views/index.ejs:

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
	</head>
	<body>
		<header>
			<h1>Welcome to the Blog</h1>
			<nav>
				<ul>
					<li>
						<a href="/authors">Authors</a>
					</li>
					<li>
						<a href="/articles">Articles</a>
					</li>
				</ul>
			</nav>
		</header>
	</body>
</html>
```

server.js:

```javascript
app.get('/', (req, res) => {
	res.render('index.ejs');
});
```

## Create Authors Index

We'll now setup the authors index page.

1. `mkdir views/authors`
1. `touch views/authors/index.ejs`

views/authors/index.ejs:

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
	</head>
	<body>
		<header>
			<h1>Authors</h1>
			<nav>
				<ul>
					<li>
						<a href="/">Home</a>
					</li>
					<li>
						<a href="/authors/new">Create a new Author</a>
					</li>
				</ul>
			</nav>
		</header>
	</body>
</html>
```

1. `mkdir controllers`
1. `touch controllers/authorsController.js`

controllers/authorsController.js:

```javascript
const express = require('express');
const router = express.Router();

router.get('/', (req, res)=>{
	res.render('authors/index.ejs');
});

module.exports = router;
```

Use the controller in server.js:

```javascript
const authorsController = require('./controllers/authorsController.js');
app.use('/authors', authorsController);
```

## Create Authors New Page

`touch views/authors/new.ejs`

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
	</head>
	<body>
		<header>
			<h1>Create an Author</h1>
			<nav>
				<ul>
					<li>
						<a href="/">Home</a>
					</li>
					<li>
						<a href="/authors">Authors Index</a>
					</li>
				</ul>
			</nav>
		</header>
		<main>
			<form action="/authors" method="POST">
				<label for="name">Name: </label>
				<input type="text" id="name" name="name" />
			</form>
		</main>
	</body>
</html>
```

create route in `controllers/authorsController.js`

```javascript
router.get('/new', (req, res)=>{
	res.render('authors/new.ejs');
});
```

## Connect to mongo

1. `npm install mongoose`
2. `mkdir models`
3. `touch models/index.js`

in models/index.js
```javascript
const mongoose = require('mongoose');

const connectionString = 'mongodb://localhost:27017/blogdb';

mongoose.connect(connectionString, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
  useFindAndModify: false
});


mongoose.connection.on('connected', () => {
  console.log(`Mongoose connected to ${connectionString}`);
});

module.exports = {
  Author: require('./Author.js')
}
```

## Set up Author Model

1. `touch models/Author.js`

```javascript
const mongoose = require('mongoose');

const authorSchema = mongoose.Schema({
	name: String
});

const Author = mongoose.model('Author', authorSchema);

module.exports = Author;
```

## Create Authors Create Route

Place this by your other require statements in your `authorsController.js` file.
```javascript
const db = require('../models/index.js');
```

We'll want to use Express's native body parser. This will allow us to parse form data and attach it to the request body.

We'll place this code just above our other `app.use` statements in the `server.js` file.
```js
app.use(express.urlencoded({extended:false}));
```

controllers/authorsController.js

```javascript
//...
//...farther down the page
router.post('/', (req, res) => {
	db.Author.create(req.body, (err, createdAuthor) => {
		if (err) return console.log(err);

		res.redirect('/authors');
	});
});
```

## Show Authors on Index Page

Let's now update the Author index Route to query the database, finding all Author, and send them to our Author index template.

controllers/authorsController.js:
```javascript
router.get('/', (req, res) => {
	db.Author.find({}, (err, foundAuthors) => {
		if (err) return console.log(err);

		res.render('authors/index.ejs', { authors: foundAuthors }
	});
});
```

In the Authors index template we'll add a main tag under neath our header tag. Here we will loop through the authors data and render a link showing the name for each Author.

views/authors/index.ejs:
```html
<main>
    <h2>List of Authors</h2>
    <ul>
        <% for(let i = 0; i < authors.length; i++){ %>
            <li>
                <a href="/authors/<%=authors[i]._id%>">
									<%=authors[i].name%>
								</a>
            </li>
        <% } %>
    </ul>
</main>
```

## Create Authors Show Page

`touch views/authors/show.ejs`

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
	</head>
	<body>
		<header>
			<h1>Show Page for <%= author.name %></h1>
			<nav>
				<ul>
					<li>
						<a href="/">Home</a>
					</li>
					<li>
						<a href="/authors">Author Index</a>
					</li>
				</ul>
			</nav>
		</header>
		<main>
			<section>
				<h2>Author Attributes:</h2>
				<ul>
					<li>Name: <%= author.name %></li>
				</ul>
			</section>
		</main>
	</body>
</html>
```

towards the bottom controllers/authorsController.js:

```javascript
// Place this route under the /new route to overriding that route.
router.get('/:id', (req, res) => {
	db.Author.findById(req.params.id, (err, foundAuthor) => {
		if (err) return console.log(err);

		res.render('authors/show.ejs', { author: foundAuthor });
	});
});
```

## Create Authors Delete Route

1. `npm install method-override`
1. use method-override in server.js:

Place this at the top of the file next your other require statements.
```javascript
const methodOverride = require('method-override')
```

Place this just **above** your other middleware calls (where you are calling `app.use`)
```js
app.use(methodOverride('_method'));
```

controllers/authorsController.js:

```javascript
router.delete('/:id', (req, res) => {
	db.Author.findByIdAndRemove(req.params.id, (err) => {
		if (err) return console.log(err);

		res.redirect('/authors');
	});
});
```

In the `show.ejs` template, add this just below the section where you are rendering the Author name.

views/authors/show.ejs

```html
<section>
    <form action="/authors/<%= author._id %>?_method=DELETE" method="post">
        <input type="submit" value="Delete Author"/>
    </form>
</section>
```

<!-- ## Create Authors Edit Page

In the `show.ejs` template add this just below the section for your delete.

Create a link on views/authors/show.ejs:

```html
<section>
    <a href="/authors/<%= author._id %>/edit">Edit</a>
</section>
```

controllers/authorsController.js

```javascript
router.get('/:id/edit', (req, res)=>{
	db.Author.findById(req.params.id, (err, foundAuthor)=>{
		res.render('authors/edit.ejs', {
			author: foundAuthor
		});
	});
});
```

`touch views/authors/edit.ejs`

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
	</head>
	<body>
		<header>
			<h1>Edit <%=author.name%>'s Info</h1>
			<nav>
				<ul>
					<li>
						<a href="/">Home</a>
					</li>
					<li>
						<a href="/authors">Authors Index</a>
					</li>
				</ul>
			</nav>
		</header>
		<main>
			<h2>Author Attributes:</h2>
			<form action="/authors/<%=author._id%>?_method=PUT" method="post">
				<input type="text" name="name" value="<%=author.name%>"/><br/>
				<input type="submit" value="Update Author"/>
			</form>
		</main>
	</body>
</html>
```

## Create Authors Put Route

controllers/authorsController.js:

```javascript
router.put('/:id', (req, res)=>{
	db.Author.findByIdAndUpdate(req.params.id, req.body, ()=>{
		res.redirect('/authors');
	});
});
``` -->
