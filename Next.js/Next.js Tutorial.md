# Next.js Tutorial

## Getting Strated

### Automatic Setup

```
npx create-next-app@latest
```

If you want to start with a TypeScript project you can use the `--typescript` flag:

```
npx create-next-app@latest --typescript
# or
yarn create next-app --typescript
```

After the installation is complete:

\- Run `npm run dev` or `yarn dev` to start the development server on `http://localhost:3000

### Manual Setup

Install `next`, `react` and `react-dom` in your project:

```
npm install next react react-dom
# or
yarn add next react react-dom
```

Open `package.json` and add the following `scripts`:

```
"scripts": {
	"dev": "next dev",
	"build": "next build",
	"start": "next start",
	"lint": "next lint"
}
```

These scripts refer to the different stages of developing an application:

\- `dev` - Runs `next dev` to start Next.js in development mode

\- `build` - Runs `next build` to build the application for production usage

\- `start` - Runs `next start` to start a Next.js production server

\- `lint` - Runs `next lint` to set up Next.js built-in ESLint configuration

Create two directories `pages` and `public` at the root of your application:

\- `pages` - Associated with a route based on their file name. For example, `pages/about.js` is mapped to `/about`

\- `public` - Stores static assets such as images, fonts, etc. Files inside `public` directory can then be referenced by your code starting from the base URL (/).

Next.js is built around the concept of pages. A page is a React Component exported from a `.js`, `.jsx`, `.ts`, or `.tsx` file in the `pages` directory. You can even add dynamic route parameters with the filename.

Inside the `pages` directory, add the `index.js` file to get started. This is the page that is rendered when the user visits the root of your application.

Populate `pages/index.js` with the following contents:

```
function HomePage() {
	return <div>Welcome to Next.js!</div>
}

export default HomePage
```

After the set up is complete:

\- Run `npm run dev` or `yarn dev` to start the development server on `http://localhost:3000`

So far, we get:

\- Automatic compilation and bundling

\- React Fast Refresh

\- Static generation and server-side rendering of `pages/`

\- Static file serving throught `public/` which is mapped to the base URL (/)

## Pages

In Next.js, a `page` is a React Component exported from a `.js`, `.jsx`, `.ts`, or `.tsx` file in the `pages` directory. Each page is associated with a route based on its file name.

Example: If you create `pages/about.js` that exports a React component like below, it will be accessible at `/about`.

```
export default function About() {
	return <div>About</div>
}
```

### Pages with Dynamic Routes

Next.js supports pages with dynamic routes. For example, if you create a file called `pages/posts/[id].js`, then it with be accessible at `posts/1`, `posts/2`, etc.

### Pre-rendering

### Two forms of Pre-rendering

Next.js has two forms of pre-rendering: `Static Generation` and `Server-side Rendering`. The difference is in when it generates the HTML for a page.

**\- Static Generation**: The HTML is generated at **build time** and will be reused on each request.

**\- Server-side Rendering**: The HTML is generated on **each request**.

### Static Generation without data

By default, Next.js pre-renders pages using Static Generation without fetching data. Here's an example:

```
function About() {
	return <div>About</div>
}

export default About
```

### Static Generation with data

Some pages require fetching external data for pre-rendering. There are two scenarios, and one or both might apply. In each case, you can use these functions that Next.js provides:

1. Your page **content** depends on external data: Use `getStaticProps`.
2. Your page **paths** depend on external data: Use `getStaticPaths` (usually in addition to `getStaticProps`).

Scenario 1: Your page content depends on external data

Example: Your blog page might need to fetch the list of blog posts from a CMS (content management system).

```
// TODO: Need to fetch `poss` (by calling some API endpoint)
// 		 before this page can be pre-rendered.
export default function Blog({ posts }) {
	return (
		<ul>
			{posts.map((post) => (
				<li>{post.title}</li>
			))}
		</ul>
	)
}
```

To fetch this data on pre-render, Next.js allows you to `export` an `async` function called `getStaticProps` from the same file. This function gets called at build time and lets you pass fetched data to the page's `props` on pre-render.

```
export default function Blog({ posts }) {
	// Render posts...
}

// This function gets called at build time
export async function getStaticProps() {
	// Call an external API endpoint to get posts
	const res = await fetch('https://.../posts')
	const posts = await res.json()
	
	// By returning { props: { posts } }, the Blog component
	// will receive `posts` as a prop at build time
	return {
		props: {
			posts,
		},
	}
}
```

### Serever-side Rendering

> Also referred to as "SSR" or "Dynamic Rendering"

If a page uses **Server-side Rendering**, the page HTML is generated on **each request**. 

To use Server-side Rendering for a page, you need to `export` an `async` function called `getServerSideProps`. This function will be called by the server on every request.

For example, suppose that your page needs to pre-render frequently updated data (fetched from an external API). You can write `getServerSideProps` which fetches this data and passes it to `Page` like below:

```
export default function Page({ data }) {
	// Render data...
}

// This gets called on every request
export async function getServerSideProps() {
	// Fetch data from external API
	const res = await fetch(`https://.../data`)
	const data = await res.json()
	
	// Pass data to the page via props
	return { props: { data } }
}
```

## Data Fetching Overview

### getServerSideProps

If you export a function called `getServerSideProps` (Server-Side Rendering) from a page, Next.js will pre-render this page on each request using the data returned by `getServerSideProps`.

```
export async function getServerSideProps(context) {
	return {
		props: {}, // will be passed to the page component as props
	}
}
```

When does getServerSideProps run

`getServerSideProps` only runs on server-side and never runs on the browser. If a page users `getServerSideProps`, then:

\- When you request this page directly, `getServerSideProps` runs at request time, and this page will be pre-rendered with the returned props

\- When you request this page on client-side page transitions through `next/link` or `next/router`, Next.js sends an API request to the server, which runs `getServerSideProps`

When should I use getServerSideProps

You should use `getServerSideProps` only if you need to render a page whose data must be fetched at request time. This could be due to the nature of the data or properties of the request (such as `authorization` headers or geo location). Pages using `getServerSideProps` will be server side rendered at request time and only be cached if cache-control headers are configured.

If you do not need to render the data during the request, then you should consider fetching data on the client side or `getStaticProps`.

getServerSideProps or API Routes

It can be tempting to reach for an API Route when you want to fetch data from the server, then call that API route from `getServerSideProps`. This is an unnecessary and inefficient approach, as it will cause an extra request to be made due to both `getServerSideProps` and API Routes running on the server.

Take the following example. An API route is used to fetch some data from a CMS. That API route is then called directly from `getServerSideProps`. This produces an additional caall, reducing performance. Instead, directly import the logic used your API Route into `getServerSideProps`. This could mean calling a CMS, database, or other API directly from inside `getServerrSideProps`.

getServerSideProps with Edge API Routes

`getServerSideProps` can be used with both Serverless and Edge Runtimes, and you can set props in both. However, currently in Edge Runtime, you do not have access to the response object. This means that you cannot -- for example -- add cookies in `getServerSideProps`. To have access to the response object, you should continue to use the Node.js runtime, which is the default runtime.

You can explicitly set the runtime on a per-page basis by modifying the `config`, for example:

```
export const config = {
	runtime: 'nodejs',
}
```

Fetching data on the client side

If your page contains frequently updating data, and you don't need to pre-render the data, you can fetch the data on the client side. An example of this is user-specific data:

\- First, immediately show the page without data. Parts of the page can be pre-rendered using Static Generation. You can show loading states for missing data

\- This approach works well for user dashboard pages, for example. Because a dashboard is a private, user-specific page, SEO is not relevant and the page doesn't need to be pre-rendered. The data is frequently updated, which requires request-time data fetching.

Using getServerSideProps to fetch data at request time

The following example shows how to fetch data at request time and pre-render the result.

```
function Page({ data }) {
	// Render data...
}

// This gets called on every request
export async function getServerSideProps() {
	// Fetch data from external API
	const res = await fetch(`https://.../data`)
	const data = await res.json()
	
	// Pass data to the page via props
	return { props: { data } }
}

export default Page
```

Caching with Server-Side Rendering (SSR)

You can use caching headers (Cache-Control) inside `getServerSideProps` to cache dynamic responses. For example, using <span style="color:blue; font-weight: bold;">stale-while-revalidate.</span>

```
export async function getServerSideProps({ req, res }) {
	res.setHeader(
		'Cache-Control',
		'public, s-maxage=10, stale-while-revalidate=59'
	)
	
	return {
		props: {},
	}
}
```

Does getServerSideProps render an error page

If an error is thrown inside `getServerSideProps`, it will show the `pages/500.js` file. Check out the documentation for 500 page to learn more on how to create it. During development this file will not be used and the dev overlay will be shown instead.

### getStaticProps

If you export a function called `getStaticProps` (Static Site Generation) from a page, Next.js will pre-render this page at build time using the props returned by `getStaticProps`.

```
export async function getStaticProps(context) {
	return {
		props: {}, // will be passed to the page component as props
	}
}
```

When should I use getStaticProps?

You should use `getStaticProps` if:

\- The data required to render the page is available at build time ahead of a user's request

\- The data comes from a headless CMS

\- The page must be pre-rendered (for SEO) and be very fast -- `getStaticProps` generates `HTML` and `JSON` files, both of which can be cached by a CDN for performance

\- The data can be publicly cached (not user-specific). This condition can be bypassed in certain specific situation by using a Middleware to rewrite the path.

When does getStaticProps run

`getStaticProps` always runs on the server and never on the client. You can validate code written inside `getStaticProps` is removed from the client-side bundle with this tool.

\- `getStaticProps` always runs during `next build`

\- `getStaticProps` runs in the backround when using `fallback: true`

\- `getStaticProps` is called before initial render when using `fallback: blocking`

\- `getStaticProps` runs in the backgrouond when using `revalidate`

\- `getStaticProps` runs on-demand in the background when using `revalidate()`

When combined with Incremental Static Regeneration, `getStaticProps` will run in the background while the stale page is being revalidated, and the fresh page served to the browser.

`getStaticProps` does not have access to the incoming request (such as query parameters or HTTP headers) as it generates static HTML. If you need access to the request for your page, consider using Middleware in addition to `getStaticProps`.

Using getStaticProps to fetch data from a CMS

The following example shows how you can fetch a list of blog posts from a CMS.

```
// posts will be populated at build time by getStaticProps()
function Blog({ posts }) {
	return (
		<ul>
			{posts.map((post) => (
				<li>{post.title}</li>
			))}
		</ul>
	)
}

// This function gets called at build time on server-side.
// It won't be called on cliend-side, so you can even do
// direct database queries
export async function getStaticProps() {
	// Call an external API endpoint to get posts.
	// You can use any data fetching library
	const res = await fetch('https://.../posts')
	const posts = await res.json()
	
	// By returning { props: { posts} }, the Blog component
	// will receive `posts` as a props at build time
	return {
		props: {
			posts,
		},
	}
}

export default Blog
```

Write server-side code directly

As `getStaticProps` runs only on the server-side, it will never run on the client-side. It won't even be included in the JS bundle for the browser, so you can write direct database queries without them being sent to browsers.

This means that instead of fetching an API route from `getStaticProps` (that itself fetches data from an external source), you can write the server-side code directly in `getStaticProps`.

Take the following example. An API route is used to fetch some data from a CMS. That API route is then called directly from `getStaticProps`. This produces an additional call, reducing performance. Instead, the logic for fetching the data from the CMS can be shared by using a `lib/` directory. Then it can be shared with `getStaticProps`. 

```
// The following function is shared
// with getStaticProps and API routes
// from a `lib/` directory
export async function loadPosts() {
	// Call an external API endpoint to get posts
	const res = await fetch('https://.../posts/')
	const data = await res.json()
	
	return data
}

// pages/blog.js
import { loadPosts } from '../lib/load-posts'

// This function runs only on the server side
export async function getStaticProps() {
	// Instead of fetching your `/api` route you can call the same 
	// function directly in `getStaticProps`
	const posts = await loadPosts()
	
	// Props returned will be passed to the page component
	return { props: { posts } }
}
```











