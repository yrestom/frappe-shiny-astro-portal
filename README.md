## Shiny Astro Portal

Astro Powered Portal Pages in Frappe App.

Source code for sample Frappe app created in the blog: https://frappe.io/blog/engineering/astro-powered-web-portal-in-frappe-apps

## Introduction
------------

There are many ways to create web pages in Frappe. You can use the **Web Page** doctype, Framework also has the **Website Generator** feature to generate web views for a DocType or you can place your HTML/markdown pages in the `www` directory.

You can build very complex pages using the above. You get bootstrap styling and script support too. 

But while building my personal website, I wanted more control. I wanted to use modern frontend tools like **TailwindCSS**, **MDX**, and more. A few weeks ago, I came across [Astro](https://astro.build/), which I immediately found interesting. According to Astro's official website:

> Astro is an all-in-one web framework for building fast, content-focused websites.

Also, their tag line is: *Build faster websites.* And they have sufficient evidence that proves it. So, I decided to give it a spin alongside Frappe Framework. I wanted to replace all of my portal pages with astro powered pages. In short, the `www` directory will be powered by an astro project. This way I could also deploy it to [Frappe Cloud](https://frappecloud.com/).

In this tutorial, I will guide you in detail on how I integrated Astro in my Frappe app (and you can too!). We will also deploy our website to Frappe Cloud at the end, so stay tuned.

## Setting up our Frappe App
-------------------------

### Prerequisites

This tutorial assumes you know the basics of Frappe Framework. You will need a working Frappe bench installation on your local machine to follow along. You can learn more about setting up bench [here](https://frappeframework.com/docs/v14/user/en/installation).

### Create a new app

Let's create a new custom Frappe app using the below command (run inside bench directory):

```
bench new-app shiny_astro_portal

```

### Create a new site

We can create a site named `abc.localhost`:

```
bench new-site abc.localhost

```

### Install the app on our site

Let's install our custom app on our newly created site:

```
bench --site abc.localhost install-app shiny_astro_portal

```

We are ready to go!

## Setting Up our Astro project
----------------------------

We will follow astro's official automatic [installation guide](https://docs.astro.build/en/install/auto/) to setup a new astro project inside our app. Let's `cd` into our custom app's directory (`shiny_astro_portal`) and run:

```
yarn create astro

```

Follow through the setup wizard giving the project a name (here, **astro_pages**) and choosing your preferred configuration (typescript and stuff):

Once the setup is complete, open up the `shiny_astro_portal` app directory in your favorite code editor (VSCode, of course). Here is the directory structure of our app at present:

```
.
├── astro_pages
├── shiny_astro_portal
├── shiny_astro_portal.egg-info
├── MANIFEST.in
├── README.md
├── license.txt
├── requirements.txt
└── setup.py

```

The `astro_pages` directory contains our astro app. Let's go inside this directory and start the astro dev server:

```
$ cd astro_pages
$ yarn run dev

```

Now open up the displayed URL (http://127.0.0.1:3000) in your browser and if everything went well, you should see the below astro starter page:

![Astro Starter Page](https://frappe.io/files/astro_first_page.png)

Astro seems to be working now. Try playing around by creating more pages or head over to [astro.build](https://astro.build/) to learn more about what you can do with astro.

I will remove the unnecessary pages and create a new page inside the pages directory named `amazing.astro` (will be built to `amazing.html`). Here is my `astro_pages/src` directory now:

```
.
├── components
├── layouts
│   └── Layout.astro
└── pages
    └── amazing.astro

```

And here are the contents of the `amazing.astro` page:

```
---
import Layout from '../layouts/Layout.astro';
---

<Layout title="My Amazing Page">
    <main class="text-center my-3">
        <h1 class="font-bold text-6xl text-teal-600">Isn't it amazing?</h1>
    </main>
</Layout>

```

The content of the `Layout.astro` layout can be found [here](https://gist.github.com/7d7c88f604111b281c74b890444fe639).

### Adding TailwindCSS

Notice, in the above code snippet I am using utility classes from TailwindCSS, so let's add that to our astro project. It is as easy as running this:

```
yarn astro add tailwind

```

Astro has many other [integrations](https://astro.build/integrations/), which makes astro very powerful and easy to extend.

You should see your amazing page when you visit '/amazing' in you dev site (`localhost:3000/amazing` usually):

![amazing.html on local](https://frappe.io/files/amazing_page_on_local.png)

The Core Concept
----------------

As you may know, every custom app contains a `www` directory. When you add a new `html` file here, it is served as a web page on your Frappe site. For example, if you add a file named `about.html` in the `www` directory, a new web page will be available on your site at **/about** (file based routing at play here). All the pages here are treated as `jinja` templates and are rendered on the server side. You can learn more about this [here](https://frappeframework.com/docs/v14/user/en/guides/portal-development/adding-pages).

The second thing to know is about serving static assets like `css` and `js` files in your Frappe app. Every app has its own `public` folder which can be used to serve static assets.

Keeping this two concepts in mind, here is what we need to do when we build our astro project for production:

1.  Place all our generated `html` files inside the `www` directory after the build.
2.  Place all of the generated static assets inside the `public` directory after the build.

Configuring Astro build
-----------------------

We can start by configuring our astro project to place the output of the build in the `www` directory. For this, open up your `astro.config.mjs` file and add the below key-value pair to the config object:

```
outDir: '../shiny_astro_portal/www'

```

Now, when you run the build command, all the output will be placed inside the `www` directory, including the assets. But we want the assets to go inside the public directory (so Frappe can serve it) and also tell `astro` to create links based on that path. We can do this by modifying our `astro_pages/package.json` build script:

```
"scripts": {
    ...,
    "build": "astro build --base=/assets/shiny_astro_portal/astro_pages/ && yarn mv-assets",
    "mv-assets": "mv -f ../shiny_astro_portal/www/assets ../shiny_astro_portal/public/astro_pages/assets",
    ...
}

```

Here, we have done two things:

1.  Set the base path for assets, so astro generates links (in `html` pages) that point to the proper path (from which Frappe will serve the static assets).
2.  Moving the assets folder to `public` directory as discussed above.

You can run `yarn build` in the `astro_pages` directory and see if the pages are being placed in `www` and assets are inside `public/astro_pages` directory. For the `mv-assets` command to work, the `astro_pages` directory must be present inside the `public` directory, so, I will create the directory and add a `.gitkeep` file inside it (so, that is gets pushed to version control).

## Getting the `bench build` command working
-----------------------------------------

Only one last thing to do now, in order for the production build to happen when `bench build` is run. Create a new `package.json` file in the root of our app and set a build script there:

```
"scripts": {
    "build": "cd astro_pages && yarn run build",
    "postinstall": "cd astro_pages && yarn install"
}

```

Here, we are just doing a `cd` into our astro project and triggering the build command. One more **important** thing is to setup a `postinstall` script which will install the necessary dependencies for our `astro` project when our custom app will be added to a bench.

Phew! That is all! Now, you can try running the below command to check if the build works:

```
bench build --app shiny_astro_portal

```

After the build command is finished running, you can try visiting the '/amazing' route on your local Frappe site (`abc.localhost:8000/amazing` in my case).

## Deploying to Frappe Cloud
-------------------------

Let's deploy our astro-powered Frappe app to Frappe Cloud now. Frappe Cloud is a managed hosting platform for Frappe sites. You can create a free FC (yup, that's what we call it) account by signing up [here](https://frappecloud.com/dashboard/signup). Adding your card will get you **$25 free credits** which you can use to create Frappe sites!

> You can also self-host your site if you wish, because, FOSS you know 😉

### Pushing our app to GitHub

At the time of writing, FC only supports pulling apps from GitHub. So, we will push our app to GitHub. Here, I am going to use the GitHub CLI ([`gh`](https://cli.github.com/)) to do so.

Before committing any changes to `git`, we want to make sure we are ignoring build assets. We can do so by adding the following lines in our root `.gitignore` file:

```
shiny_astro_portal/www/*
astro_pages/.astro/*
shiny_astro_portal/public/astro_pages/assets

```

Now, commit all the changes by running these commands inside your app directory:

```
$ git add .
$ git commit -m "feat: add astro project"

```

Use the `gh` CLI to create a new repo and push our app source code to GitHub:

```
# in apps/shiny_astro_portal directory
gh repo create

```

Choose 'Push an existing local repository to GitHub' and '.' as the path.

### Creating a private bench

Follow the instructions (written by yours truly) [here](https://frappecloud.com/docs/benches/create-new) to create a new private bench on FC.

### Adding our app from GitHub

Once we have created the private bench, we can add our `shiny_astro_portal` app from GitHub. Follow [this](https://frappecloud.com/docs/benches/custom-app) guide to connect your GitHub account and add your custom app containing the astro project. After adding the app, we can deploy the bench:

### Creating a new site

After the bench is successfully deployed, we can create a new site on that bench using the `New Site` button on the top right corner of the bench dashboard:

Now, go through the setup wizard by choosing a subdomain, apps to install (make sure you select `shiny_astro_portal`) and selecting a plan. As soon as you are done with the setup wizard, the site creation will be initiated for you. Wait for the jobs to complete. You now have a Frappe site with `astro` powered web pages!

Try visiting the path `/amazing` on your FC site to check if it works:

![Amazing page on FC site](https://frappe.io/files/amazing_page_fc_site.png)

## Additional Resources
--------------------

You can find the source code for the `shiny_astro_portal` app [here](https://github.com/NagariaHussain/shiny_astro_portal). You can copy the build configurations and replace the names with your particular app name and portal name to get up and running quickly.

You can build docs, marketing pages, blogs and much more with astro. I would suggest you head over to [astro.build](https://astro.build/) to learn more.

Learn more about Frappe Framework [here](https://frappeframework.com/docs).

## Conclusion
----------

Since you are still with me, I probably think this guide was helpful. You can setup a variety of static site generation frameworks using the same principles. 

See you in the next one!

#### License

MIT
