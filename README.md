# Redwood

> **WARNING:** RedwoodJS software has not reached a stable version 1.0 and should not be considered suitable for production use. In the "make it work; make it right; make it fast" paradigm, Redwood is in the later stages of the "make it work" phase.

## Getting Started
- [Tutorial](https://redwoodjs.com/tutorial/welcome-to-redwood): getting started and complete overview guide.
- [Docs](https://redwoodjs.com/docs/introduction): using the Redwood Router, handling assets and files, list of command-line tools, and more.
- [Redwood Community](https://community.redwoodjs.com): get help, share tips and tricks, and collaborate on everything about RedwoodJS.

### Setup

We use Yarn as our package manager. To get the dependencies installed, just do this in the root directory:

```terminal
yarn install
```

### Fire it up

```terminal
yarn redwood dev
```

Your browser should open automatically to `http://localhost:8910` to see the web app. Lambda functions run on `http://localhost:8911` and are also proxied to `http://localhost:8910/.redwood/functions/*`.

## Setup Database

You will need a Postges database, either running on Postgres.app, psql, Docker, or one of the few thousands startups currently hosting free production Postgres databases. If you've never used Postgres before [follow this guide](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-3-5ao5).

### Create `.env` file for DATABASE_URL

Do not commit this file. It is included in `.gitignore` by default.

```bash
touch .env
```

The `DATABASE_URL` is injected into `prisma.schema`.

```
DATABASE_URL=
```

### Run migration

```bash
yarn rw prisma migrate dev --name nailed-it
```

This will create a migration called `nailed-it`. This migration will generate the following `.sql` file.

```sql
CREATE TABLE "Post" (
    "id" SERIAL NOT NULL,
    "title" TEXT NOT NULL,
    "body" TEXT NOT NULL,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY ("id")
);
```

### Create test data

If you were already running your development server, restart it so your environment variable is injected. Open `localhost:8910/posts`.

![01-posts-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y9nxxjrxgrbwitr4x553.png)

We have a new page called Posts with a button to create a new post. If we click the new post button we are given an input form with fields for title and body.

![02-new-post-form](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/suglknt7f8jaema7ic1o.png)

We were taken to a new route, `/posts/new`. If we click the save button we are brought back to the posts page.

![03-new-created-post](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xu8dvrpksgfvgj0pvhvp.png)

Return to `localhost:8910`.

![04-home-page-with-post](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3x5sleicr037j2rbnyeb.png)

## Home page

Our home page displays the data fetched from our `BlogPostsCell`.

```javascript
import BlogPostsCell from 'src/components/BlogPostsCell'

const HomePage = () => {
  return (
    <>
      <h1>Redwood+Begin</h1>
      <p>Ship it</p>
      <BlogPostsCell />
    </>
  )
}

export default HomePage
```

## BlogPostsCell

Our `BlogPostsCell` fetches the blog posts with a GraphQL query to our `posts` service inside `services/post/posts.js`.

```javascript
export const QUERY = gql`
  query BlogPostsQuery {
    posts {
      id
      title
      body
      createdAt
    }
  }
`

export const Loading = () => <div>Loading...</div>

export const Empty = () => <div>Empty</div>

export const Failure = ({ error }) => <div>Error: {error.message}</div>

export const Success = ({ posts }) => {
  return posts.map((post) => (
    <article key={post.id}>
      <header>
        <h2>{post.title}</h2>
      </header>
      <p>{post.body}</p>
      <div>Posted at: {post.createdAt}</div>
    </article>
  ))
}
```

## posts.js

```javascript
import { db } from 'src/lib/db'

export const posts = () => {
  return db.post.findMany()
}
```

## schema.prisma

Our schema is set to a `postgres` database and has a single `Post` model.

```prisma
datasource DS {
  provider = "postgres"
  url      = env("DATABASE_URL")
}

generator client {
  provider      = "prisma-client-js"
  binaryTargets = "native"
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  body      String
  createdAt DateTime @default(now())
}
```

## App.js

The entry point to our web side.

```javascript
import { FatalErrorBoundary } from '@redwoodjs/web'
import { RedwoodApolloProvider } from '@redwoodjs/web/apollo'

import FatalErrorPage from 'src/pages/FatalErrorPage'
import Routes from 'src/Routes'

import './index.css'

const App = () => (
  <FatalErrorBoundary page={FatalErrorPage}>
    <RedwoodApolloProvider>
      <Routes />
    </RedwoodApolloProvider>
  </FatalErrorBoundary>
)

export default App
```

## Routes.js

```javascript
import { Router, Route } from '@redwoodjs/router'

const Routes = () => {
  return (
    <Router>
      <Route path="/" page={HomePage} name="home" />
      <Route notfound page={NotFoundPage} />
    </Router>
  )
}

export default Routes
```


Each route is specified with a `Route`. To create a route to a normal Page, you'll pass three props:
- `path`
- `page`
- `name`

The `path` prop specifies the URL path to match, starting with the beginning slash. The `page` prop specifies the Page component to render when the path is matched. The `name` prop is used to specify the name of the _named route function_.