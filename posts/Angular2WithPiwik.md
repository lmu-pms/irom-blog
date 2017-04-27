# Adding Piwik Analytics to an Angular2 app

## 1. Install Piwik

### Server and database for Piwik

The first thing to do is to install Piwik on its own server. To get Piwik up and running we have to meet some <a href="https://piwik.org/docs/requirements/" target="_blank">requirements</a>:

* Webserver such as Apache, Nginx, IIS, etc.
* PHP version 5.5.9 or greater
* MySQL version 5.5 or greater, or MariaDB
* (enabled by default) PHP extension pdo and pdo_mysql, or the mysqli extension.

For development purposes we can use XAMPP (for Windows) as a server and MySQL provider. Then we have to <a href="https://piwik.org/faq/how-to-install/faq_23484/" target="_blank">create a database and user for Piwik</a>, either by manually doing it in the MySQL admin interface or by executing the following statements:

```sql
$ mysql> CREATE DATABASE piwik_db_name_here;
```
```sql
$ mysql> CREATE USER 'piwik'@'localhost' IDENTIFIED BY 'my-strong-password-here';
```
```sql 
$ mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER, CREATE TEMPORARY TABLES, LOCK TABLES ON piwik_db_name_here.* TO 'piwik'@'localhost';
```

### Piwik installation

Now we can install Piwik on our server. Detailed installation information can be found in Piwik's <a href="https://piwik.org/docs/installation/" target="_blank">installation guide</a>.

* Download the latest Piwik release <a href="https://builds.piwik.org/piwik.zip" target="_blank">from here</a>
* Unzip the file into a folder on our previously set up server, e.g. in `xampp/htdocs/analytics` to access the Piwik analytics at `localhost/analytics`
* Now open your browser and go to your Piwik analytics site. You should see the installation welcome screen where you have to click "Next" to start the installation. The installation wizard guides you over the next 7 short steps. We have to provide our database info and create a Piwik super user (for login to our analytics site), provide our website URL that we want to track and get the Javascript tracking code for inserting into our app. Save that code snippet for step 2 of this guide.

## 2. Add Piwik to your Angular2 app

To enable tracking on our Angular2 app, all we have to do now is to add the previosly obtained JS tracking code just before the closing `</head>` tag in the `index.html` of our app:

```html
<!-- index.html -->
<head>
    ...
    <!-- Piwik -->
    <script type="text/javascript">
      var _paq = _paq || [];
      /* tracker methods like "setCustomDimension" should be called before "trackPageView" */
      _paq.push(['trackPageView']);
      _paq.push(['enableLinkTracking']);
      (function() {
        var u="[YOUR_ANALYTICS_SITE_URL]";
        _paq.push(['setTrackerUrl', u+'piwik.php']);
        _paq.push(['setSiteId', '[YOUR_WEBSITE_ID]']);
        var d=document, g=d.createElement('script'), s=d.getElementsByTagName('script')[0];
        g.type='text/javascript'; g.async=true; g.defer=true; g.src=u+'piwik.js'; s.parentNode.insertBefore(g,s);
      })();
    </script>
    <!-- End Piwik Code -->
  </head>
  ```

That's it! At least the basic tracking should be working now. Visit your app in the browser and then check your analytics site. You should see that there is at least one visitor (you) on your app site right now.

But Piwik doesn't track the subpages yet. To enable tracking on Angular2 subpages we need to install a Piwik plugin for Angular2. There are some plugins out there for integration of Piwik in Angular2 apps, but many of them are pretty outdated. 

The most known and used plugin for Angular2 apps is <a href="https://github.com/angulartics/angulartics2" target="_blank">Angulartics2</a>. It works for a lot of analytics providers including Piwik. But we don't use it in this tutorial for two simple reasons: First, there's no documentation for Piwik yet and second, it's pretty overloaded for our purposes because we don't need all the other analytics providers, who are supported by this plugin. 

A simple Piwik wrapper that meets our needs is the recently published <a href="https://github.com/awronka/Angular2Piwik" target="_blank">Angular2Piwik plugin</a>. We install it in our app by running

```console
npm install --save angular2piwik
```

Next, we have to import it in our `NgModule`:

```javascript
// app.module.ts
// ...
import { Angular2PiwikModule } from 'Angular2Piwik';
// ...
@NgModule({
  imports: [
    // ...
    Angular2PiwikModule
  ],
  // ...
})
```

The last thing we have to do is to call the tracking function on every subpage (i.e. component) we want to track:

```javascript
// my.component.ts
// ...
import { UsePiwikTracker } from 'Angular2Piwik';
// ...
@Component({
  // ...
  providers: [ UsePiwikTracker ]
})
export class MyComponent {
  constructor(
    // ...
    private usePiwikTracker: UsePiwikTracker
  ) {
    // ...
    usePiwikTracker.trackPageView();
  }
  // ...
}
```

## 3. Have fun with analytics!

We're all set up now, every visit on one of our app pages is being registered by Piwik and shown on the analytics page. Of course there is much more we can track, the Angular2Piwik plugin provides all original Piwik tracking functions. 

For example, this is how to track a click event:

```javascript
// my.component.ts
// ...
import { UsePiwikTracker } from 'Angular2Piwik';
// ...
@Component({
  // ...
  providers: [ UsePiwikTracker ]
})
export class MyComponent {
  // ...
  constructor(
    // ...
    private usePiwikTracker: UsePiwikTracker
  ) { }

  doThisOnClick(someVal): void {
    this.usePiwikTracker.trackEvent('clickEvent', {category : 'myEvents', label: 'Clicks', value: someVal});
    // some event handling code...
  }
}
```

Check out your analytics site to see how events are tracked by Piwik!
