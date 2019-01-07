# Paper Question Sheets

For math and music quizes.

Uses the [ABC JS music transcription library](https://abcjs.net/).

## Things I Learned

Coming into this project I had never used a JS asset bundler. I was also unfamiliar with using NPM packages and package.json config.

I'm using [Parcel](https://parceljs.org) to build / bundle the app. Parcel is "zero config" if you don't count configing your npm `package.json` file. For example:

    "devDependencies": {
        "parcel-bundler": "^1.11.0"
    },

    "dependencies": {
        "abcjs": "^5.6.1"
    }

Dependencies are installed from the command line:

    npm install

The `abcjs` dependency can now be imported within the `index.js` Javascript:

    import ABCJS from 'abcjs';
    ABCJS.renderAbc('paper', 'ABC');

In development mode, view app on [http://localhost:1234](http://localhost:1234) by running:

    parcel index.html

To bundle-up a production distribution, run:

    parcel build index.html

Note that because `babel-core` is present as a dependency in `package.json` all [the JS will be transpiled automatically](https://parceljs.org/javascript.html#default-babel-transforms).

## [Heroku](https://heroku.com) Deployment

This was a battle, but I won. ⚔️ [musical-bears.herokuapp.com](https://musical-bears.herokuapp.com/) ⚔️

Our Heroku build requirements:

* Must recognized NPM package.json config and install required modules.
* Must run Parcel to bundle the app to the `dist` folder.
* Must deploy an HTTP server to host bundled app from the `dist` folder.

After much battle with Heroku's auto-recognized default NodeJS buildpack I switched to a custom buildpack pipeline. I used the official nodejs buildpack and the experimental [static HTML/JS app buildpack](https://github.com/heroku/heroku-buildpack-static); see the [static buildpack getting started guide](https://gist.github.com/hone/24b06869b4c1eca701f9) for details.

Here's how I deployed from the CLI. These instructions assume that you have the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) installed.

Added `heroku-postbuild` to the `package.json` scripts section:

    "scripts": {
        "dev": "parcel index.html",
        "build": "parcel build index.html",
        "heroku-postbuild": "parcel build index.html"
    },

From the command line:

    heroku login
    heroku create
    heroku buildpacks:clear
    heroku buildpacks:add heroku/nodejs
    heroku buildpacks:add https://github.com/hone/heroku-buildpack-static
    touch static.json

Open the `static.json` file and configure heroku-buildpack-static to host the `index.html` in Parcel's `dist` folder. The `static.json` file should look like this:

    {
        "root": "dist/"
    }

And back at the CLI:

    git add .
    git -m "Added parcel as a postbuild step to package.json plus a static.json config file for heroku-buildpack-static."
    git push heroku master
    heroku open

## [Surge](https://surge.sh) Deployment

This was not a battle, but app must be locally built.

🌊 [musical-bears.surge.sh](https://musical-bears.surge.sh/) 🌊

With surge you can deploy any folder. When using a JS bunlder you need to build the app locally first, and then deploy with surge.

I installed the surge CLI as a dev dependency to test it out:

    npm install --save-dev surge

And added an entry in the `package.json` scripts section:

    "scripts": {
        "dev": "parcel index.html",
        "build": "parcel build index.html",
        "surge": "surge"
    }

The version of the `cryptiles` package required by `surge` triggered a security notice, so included it in the dev dependencies with an explicit version in `package.json`:

    "devDependencies": {
      "parcel-bundler": "^1.11.0",
      "parcel-plugin-clean-dist": "0.0.6",
      "cryptiles": ">=4.1.2",
      "surge": "^0.20.1"
    }

I then ran parcel's build and the surge CLI:

    npm run build
    npm run surge

The first time through I was prompted to create an account.

**Important:** The surge CLI will prompt for the project folder. Add parcel's `dist` folder to the end of the default path to ensure you're deploying the bundled app.

## GitHub Pages Deployment

Easy-peasy once again. I had to deploy as `paper-questions` because that's the name of the associated GitHub repo:

📖 [stungeye.github.io/paper-questions/](https://stungeye.github.io/paper-questions/)  📖

Install the `gp-pages` package as a development dependency:

    npm install --save-dev gh-pages

Update `package.json` with new scripts:

    "scripts": {
        "dev": "parcel index.html",
        "build": "parcel build index.html",
        "build-gh-pages": "parcel build index.html --public-url /paper-questions",
        "deploy-gh-pages": "gh-pages -d dist"
    }

From the CLI:

    npm run build-gh-pages
    npm run deploy-gh-pages


## [Netlify](https://www.netlify.com) Deployment

Super easy continuous deploymenting using their web dashboard:

🔥 https://musical-bears.netlify.com/ 🔥

Steps:

* Login to Netlify dashboard.
* Click "New Site From Git" button.
* Auth your Github account and pick the repo you wish to deploy.
* Setup the build steps:
  * Build command: `npm run build`
  * Publish directory: `dist`

Once the site has been built I went into the Domain Management settings and changed the randomly assigned domain name to `musical-bears.netlify.com`.

Looks like all of this can also be done from their CLI.

## [Zeit Now](https://zeit.co/now) Deployment (With Local Build)

Problem free when deploying a local build.

🎪 [musical-bears.now.sh/](https://musical-bears.now.sh/) 🎪

Install the `now` package as a development dependency:

    npm install --save-dev now

Update `package.json` with `now` alias:

    "scripts": {
        "dev": "parcel index.html",
        "build": "parcel build index.html",
        "now-from-dist": "now --name musical-bears deploy ./dist",
        "now-alias": "now alias https://musical-bears.now.sh",
    }

Deploy from the CLI:

    npm run build
    npm run now-from-dist
    npm run now-alias

The default subdomain assigned by Zeit has a unique id in it; something like musical-bears-pvrgpxdmt.now.sh.

The final command sets up a nicer alias for the install subdomain.

## [Zeit Now](https://zeit.co/now) Deployment (With Remote Build)

Currently not possible due to [/zeit/now-builders/issues/49](https://github.com/zeit/now-builders/issues/49). The `abcjs` music library uses git for one of it's dependencies and `now-builders` currently doesn't support this.

Update `package.json` with `now` alias:

    "scripts": {
        "dev": "parcel index.html",
        "build": "parcel build index.html",
        "now": "now",
        "now-build": "parcel build index.html"
    }

Create a `now.json` file:

    {
        "version": 2,
        "name": "musical-bears",
        "builds": [
            { "src": "package.json", "use": "@now/static-build", "config": {"distDir": "dist"} }
        ]
    }

Deploy from CLI:

    rm -rf ./dist
    npm run now

The first time through I had to confirm my email address and click on a verification email. I had to rerun the above command after that to deploy.

The `rm -rf ./dist` was required. Otherwise the `now` CLI tries to upload this folder.

## UNLICENSE

This is free and unencumbered software released into the public domain. See UNLICENSE for details.
