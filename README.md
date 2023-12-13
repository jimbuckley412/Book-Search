# Book Search Engine

This is a MERN stack application that allows users to search for books and save them to their profile. This application uses the Google Books API to search for books and save them to a MongoDB database. This application was refactored from a RESTful API to a GraphQL API. The application was also refactored from using the Context API to using Apollo Client for state management. The application was also refactored from using a RESTful API to a GraphQL API. The application was also refactored from using the Context API to using Apollo Client for state management.

## Table of Contents

- [Installation](#installation)
- [Usage](#usage)
- [Technologies Used](#technologies-used)
- [Folder Structure](#folder-structure)
- [Database Configuration](#database-configuration)
- [Authentication](#authentication)
- [Routes](#routes)
- [Client-Side](#client-side)
- [Contributing](#contributing)
- [License](#license)

## Installation

1. Clone the repository:

```bash
git clone https://github.com/your-username/your-repository.git
cd your-repository
npm install
npm start

## Usage

![Book Search Engine](./assets/book-search-engine.gif)

## Technologies Used

- [MongoDB](https://www.mongodb.com/)
- [Express.js](https://expressjs.com/)
- [React](https://reactjs.org/)
- [Node.js](https://nodejs.org/en/)
- [GraphQL](https://graphql.org/)
- [Apollo Server](https://www.apollographql.com/docs/apollo-server/)
- [Apollo Client](https://www.apollographql.com/docs/react/)
- [JSON Web Tokens](https://jwt.io/)
- [bcrypt](https://www.npmjs.com/package/bcrypt)
- [Bootstrap](https://getbootstrap.com/)
- [React Bootstrap](https://react-bootstrap.github.io/)
- [React Router](https://reactrouter.com/)
- [React Hook Form](https://react-hook-form.com/)

## Folder Structure

```bash
.
├── client
│   ├── public
│   │   └── index.html
│   ├── src
│   │   ├── components
│   │   │   ├── BookList.js
│   │   │   ├── LoginForm.js
│   │   │   ├── Nav.js
│   │   │   ├── SavedBooks.js
│   │   │   ├── SearchBooks.js
│   │   │   ├── SignupForm.js
│   │   │   └── styles.css
│   │   ├── pages
│   │   │   ├── Home.js
│   │   │   ├── NoMatch.js
│   │   │   └── Profile.js
│   │   ├── utils
│   │   │   ├── auth.js
│   │   │   ├── queries.js
│   │   │   └── mutations.js
│   │   ├── App.js
│   │   ├── index.js
│   │   └── App.test.js
│   ├── .gitignore
│   ├── package-lock.json
│   ├── package.json
│   └── README.md
├── models
│   ├── Book.js
│   ├── index.js
│   └── User.js
├── node_modules
├── routes
│   ├── api
│   │   ├── books.js
│   │   └── index.js
│   └── index.js
├── .gitignore
├── package-lock.json
├── package.json
└── server.js
```

## Database Configuration

The application uses MongoDB for the database. The database is configured in the `server.js` file:

```javascript
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost/googlebooks', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
  useFindAndModify: false
});
```

## Authentication

The application uses JSON Web Tokens for authentication. The JWT is generated in the `Auth.js` file in the `utils` folder:

```javascript
const token = signToken(user);
```

The JWT is verified in the `typeDefs.js` file in the `server` folder:

```javascript
const { AuthenticationError } = require('apollo-server-express');

const { User } = require('../models');
const { signToken } = require('../utils/auth');

const resolvers = {
  Query: {
    me: async (parent, args, context) => {
      if (context.user) {
        const userData = await User.findOne({ _id: context.user._id })
          .select('-__v -password')
          .populate('books');

        return userData;
      }

      throw new AuthenticationError('Not logged in');
    },
  },
  Mutation: {
    login: async (parent, { email, password }) => {
      const user = await User.findOne({ email });

      if (!user) {
        throw new AuthenticationError('Incorrect credentials');
      }

      const correctPw = await user.isCorrectPassword(password);

      if (!correctPw) {
        throw new AuthenticationError('Incorrect credentials');
      }

      const token = signToken(user);
      return { token, user };
    },
    addUser: async (parent, args) => {
      const user = await User.create(args);
      const token = signToken(user);

      return { token, user };
    },
    saveBook: async (parent, { bookData }, context) => {
      if (context.user) {
        const updatedUser = await User.findByIdAndUpdate(
          { _id: context.user._id },
          { $push: { savedBooks: bookData } },
          { new: true }
        );

        return updatedUser;
      }

      throw new AuthenticationError('You need to be logged in!');
    },
    removeBook: async (parent, { bookId }, context) => {
      if (context.user) {
        const updatedUser = await User.findOneAndUpdate(
          { _id: context.user._id },
          { $pull: { savedBooks: { bookId: bookId } } },
          { new: true }
        );

        return updatedUser;
      }

      throw new AuthenticationError('You need to be logged in!');
    },
  },
};

module.exports = resolvers;
```

## Routes

The application uses GraphQL for the API routes. The GraphQL routes are configured in the `server.js` file:

```javascript
const { ApolloServer } = require('apollo-server-express');
const { authMiddleware } = require('./utils/auth');
const { typeDefs, resolvers } = require('./schemas');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: authMiddleware,
});

server.applyMiddleware({ app });

app.use(express.urlencoded({ extended: true }));
app.use(express.json());

if (process.env.NODE_ENV === 'production') {
  app.use(express.static(path.join(__dirname, '../client/build')));
}

app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, '../client/build/index.html'));
});

mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost/googlebooks', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
  useFindAndModify: false
});

app.listen(PORT, () => {
  console.log(`🌍 Now listening on localhost:${PORT}`);
  console.log(`Use GraphQL at http://localhost:${PORT}${server.graphqlPath}`);
});
```

## Client-Side

The application uses React for the client-side. The React components are located in the `client/src/components` folder. The React Router is used to navigate between the different pages of the application. The React Router is configured in the `client/src/App.js` file:

```javascript
import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import { ApolloProvider } from '@apollo/react-hooks';
import ApolloClient from 'apollo-boost';

import Home from './pages/Home';
import Profile from './pages/Profile';
import NoMatch from './pages/NoMatch';

import 'bootstrap/dist/css/bootstrap.min.css';

const client = new ApolloClient({
  request: operation => {
    const token = localStorage.getItem('id_token');

    operation.setContext({
      headers: {
        authorization: token ? `Bearer ${token}` : ''
      }
    });
  },
  uri: '/graphql'
});

function App() {
  return (
    <ApolloProvider client={client}>
      <Router>
        <div>
          <Switch>
            <Route exact path="/" component={Home} />
            <Route exact path="/profile" component={Profile} />
            <Route component={NoMatch} />
          </Switch>
        </div>
      </Router>
    </ApolloProvider>
  );
}

export default App;
```

## Contributing

Jim Buckley

## License

MIT License
```
