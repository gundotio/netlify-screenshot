Netlify Screenshot
==================

Screenshot webpages to render social media cards on-the-fly using Puppeteer; largely based on [how Pieter generates shareable pictures](https://levels.io/phantomjs-social-media-share-pictures) for [Nomad List](https://nomadlist.com), and how I did them for [Coworkations](https://coworkations.com) with [cardserver](https://github.com/stevelacey/cardserver).

| [![Gun.io: Hire the best talent in software development](https://gunio.netlify.app/screenshot/home.png)](https://gunio.netlify.app/screenshot/home.png) [📄 HTML](https://gun.io) [🖼 PNG](https://gunio.netlify.app/screenshot/home.png) | [![Gun.io Blog](https://gunio.netlify.app/screenshot/blog.png)](https://gunio.netlify.app/screenshot/blog.png) [📄 HTML](https://gun.io/blog) [🖼 PNG](https://gunio.netlify.app/screenshot/blog.png) |
| --: | --: |
| **[![Steaming MVP: Job Card](https://gunio-guild.netlify.app/screenshot/cards/jobs/c4521383-5dba-4d2b-ad24-816a28c9df32.png)](https://gunio-guild.netlify.app/screenshot/cards/jobs/c4521383-5dba-4d2b-ad24-816a28c9df32.png) [📄 HTML](https://app.gun.io/cards/jobs/c4521383-5dba-4d2b-ad24-816a28c9df32) [🖼 PNG](https://gunio-guild.netlify.app/screenshot/cards/jobs/c4521383-5dba-4d2b-ad24-816a28c9df32.png)** | **[![Steve L: Freelancer Card](https://gunio-guild.netlify.app/screenshot/cards/freelancer/579af90e-d141-41b9-b7a8-2c580b00f044.png)](https://gunio-guild.netlify.app/screenshot/cards/freelancer/579af90e-d141-41b9-b7a8-2c580b00f044.png) [📄 HTML](https://app.gun.io/cards/freelancer/579af90e-d141-41b9-b7a8-2c580b00f044) [🖼 PNG](https://gunio-guild.netlify.app/screenshot/cards/freelancer/579af90e-d141-41b9-b7a8-2c580b00f044.png)** |


Setup
-----

After deploying to Netlify and setting `BASE_URL` the site should be good to go, try visiting `https://{site-name}.netlify.app/screenshot` for a capture of your homepage.

Any additional path that comes after `/screenshot` is used in the request to your webserver:

```
https://{site-name}.netlify.app/screenshot/**/* -> {BASE_URL}/**/*
```

If you want to change the `BASE_URL` edit the site's environment variables from `Site Settings > Build & deploy > Environment`, the value should be the root domain of your website e.g. `https://example.com`.


Usage
-----

Netlify Screenshot performs HTML requests based on PNG requests like so:

| 🖼 PNG (Netlify request) | 📄 HTML (webserver request) |
| :--------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------- |
| https://gunio.netlify.app/screenshot/home.png                                                        | https://gun.io/home                                                      |
| https://gunio.netlify.app/screenshot/blog.png                                                        | https://gun.io/blog                                                      |
| https://gunio-guild.netlify.app/screenshot/cards/freelancer/579af90e-d141-41b9-b7a8-2c580b00f044.png | https://app.gun.io/cards/freelancer/579af90e-d141-41b9-b7a8-2c580b00f044 |
| https://gunio-guild.netlify.app/screenshot/cards/jobs/c4521383-5dba-4d2b-ad24-816a28c9df32.png       | https://app.gun.io/cards/jobs/c4521383-5dba-4d2b-ad24-816a28c9df32       |

If you want to serve your cards on the same domain as your app, you can route PNG traffic to Netlify via NGINX:

```
location ~ ^/(cards/.*\.png)$ {
  proxy_pass http://{site-name}.netlify.app/screenshot/$1;
}
```

Then your URLs can look something like this:

| 🖼 PNG (Netlify request) | 📄 HTML (webserver request) |
| :--------------------------------------------------------------------------- | :----------------------------------------------------------------------- |
| https://gun.io/home.png                                                      | https://gun.io/home                                                      |
| https://gun.io/blog.png                                                      | https://gun.io/blog                                                      |
| https://app.gun.io/cards/freelancer/579af90e-d141-41b9-b7a8-2c580b00f044.png | https://app.gun.io/cards/freelancer/579af90e-d141-41b9-b7a8-2c580b00f044 |
| https://app.gun.io/cards/jobs/c4521383-5dba-4d2b-ad24-816a28c9df32.png       | https://app.gun.io/cards/jobs/c4521383-5dba-4d2b-ad24-816a28c9df32       |

Alternatively, you can route all `/screenshots/` traffic to Netlify, then prefix any path on your site with `/screenshots/` to capture a screenshot.

```
location /screenshots/ {
  proxy_pass http://{site-name}.netlify.app/screenshot/;
}
```

Note that query params are passed through to your backend, which can be useful if you want to toggle features like Cookiebot or Intercom on/off.


Caching
-------

Netlify Screenshot serves sensible cache headers, and allows you to suffix requests with `.png` which helps reverse proxies like Cloudflare recognize the content as cacheable.

In addition to that, you might want to cache results locally so they don't hit Netlify every time.

To do that, add something like this to your `nginx.conf`:

```
proxy_cache_path /tmp/nginx levels=1:2 keys_zone=static:10m max_size=10g inactive=7d use_temp_path=off;
proxy_cache_valid 7d;
```

And update your location block like so:

```
location /screenshots/ {
  proxy_cache static;
  proxy_pass http://{site-name}.netlify.app/screenshot/;
}
```


Markup
------

You’ll want meta tags something like these:

```html
<meta itemprop="image" content="https://gunio.netlify.app/screenshot/blog.png">
<meta property="og:image" content="https://gunio.netlify.app/screenshot/blog.png">
<meta name="twitter:image" content="https://gunio.netlify.app/screenshot/blog.png">
```


Debugging
---------

- [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug)
- [Twitter Card Validator](https://cards-dev.twitter.com/validator)
