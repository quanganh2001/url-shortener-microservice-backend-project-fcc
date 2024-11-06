# Setup MongoDB and config database
I do myself, so I do not write instructions for MongoDB. Please following the instructions in the Internet.

Store the password in an environment variable like this:
```env
PORT=
URI=
```

Type: `npm i mongodb`
# Tests
1. You can POST a URL to `/api/shorturl` and get a JSON response with `original_url` and `short_url` properties.
```js
const { MongoClient } = require('mongodb');
const dns = require('dns');
const urlparser = require('url');

const client = new MongoClient(process.env.URI);
const db = client.db("urlshortener");
const urls = db.collection("urls");

app.use(express.json());
app.use(express.urlencoded({extended: true}));

// Your first API endpoint
app.post('/api/shorturl', function(req, res) {
  console.log(req.body);
  const url = req.body.url;
  const dnslookup = dns.lookup(urlparser.parse(url).hostname, async (err, address) => {
    if (!address) {
      res.json({error: "Invalid URL"});
    } else {

      const urlCount = await urls.countDocuments({});
      const urlDoc = {
        url,
        short_url: urlCount
      };

      const result = await urls.insertOne(urlDoc);
      console.log(result);
      res.json({ original_url: url, short_url: urlCount });

    }
  });
});
```
2. When you visit `/api/shorturl/<short_url>`, you will be redirected to the original URL.
```js
app.get("/api/shorturl/:short_url", async (req, res) => {
  const shorturl = req.params.short_url;
  const urlDoc = await urls.findOne({ short_url: +shorturl });
  res.redirect(urlDoc.url);
});
```
Full source code:
```js
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const app = express();
const { MongoClient } = require('mongodb');
const dns = require('dns');
const urlparser = require('url');

const client = new MongoClient(process.env.URI);
const db = client.db("urlshortener");
const urls = db.collection("urls");

// Basic Configuration
const port = process.env.PORT || 3000;

app.use(cors());
app.use(express.json());
app.use(express.urlencoded({extended: true}));

app.use('/public', express.static(`${process.cwd()}/public`));

app.get('/', function(req, res) {
  res.sendFile(process.cwd() + '/views/index.html');
});

// Your first API endpoint
app.post('/api/shorturl', function(req, res) {
  console.log(req.body);
  const url = req.body.url;
  const dnslookup = dns.lookup(urlparser.parse(url).hostname, async (err, address) => {
    if (!address) {
      res.json({error: "Invalid URL"});
    } else {

      const urlCount = await urls.countDocuments({});
      const urlDoc = {
        url,
        short_url: urlCount
      };

      const result = await urls.insertOne(urlDoc);
      console.log(result);
      res.json({ original_url: url, short_url: urlCount });

    }
  });
});

app.get("/api/shorturl/:short_url", async (req, res) => {
  const shorturl = req.params.short_url;
  const urlDoc = await urls.findOne({ short_url: +shorturl });
  res.redirect(urlDoc.url);
});

app.listen(port, function() {
  console.log(`Listening on port ${port}`);
});

```