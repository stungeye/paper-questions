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

