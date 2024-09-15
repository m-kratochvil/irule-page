## Simple maintenance page

#### 1. Create the website page
* SSH into the F5 device
* Create the actual webpage file:
  * `vi /var/tmp/splash-page.html`
* Add the following html code into the page:
```
<!doctype html>
<title>Site Maintenance</title>
<style>
  body { text-align: center; padding: 150px; }
  h1 { font-size: 50px; }
  body { font: 20px Helvetica, sans-serif; color: #333; }
  article { display: block; text-align: left; width: 650px; margin: 0 auto; }
  a { color: #dc8100; text-decoration: none; }
  a:hover { color: #333; text-decoration: none; }
</style>
<article>
    <h1>We&rsquo;ll be back soon!</h1>
    <div>
        <p>Sorry for the inconvenience but we&rsquo;re performing some maintenance at the moment. If you need to you can always <a href="mailto:#">contact us</a>, otherwise we&rsquo;ll be back online shortly!</p>
        <p>&mdash; The Team</p>
    </div>
</article>
```

#### 2. Create the system-level iFile
`# tmsh create sys file ifile maintenance-splash-page.html source-path file:/var/tmp/splash-page.html`

#### 3. Create the LTM-level iFile
`# tmsh create ltm ifile maintenance-splash-page.html file-name maintenance-splash-page.html`

#### 4. Create the irule
* Go to "Local Traffic" --> "iRules", click on "Create"
* Name: maint-page-irule
* Paste the following irule into the Definition section and click "Update":

```
when HTTP_REQUEST {
  set vsPool [LB::server pool]
  if { [active_members $vsPool] < 1 } {
   # log local0. "Client [IP::client_addr] requested [HTTP::uri] no active nodes available..."
      HTTP::respond 200 content "[ifile get  maintenance-splash-page.html]" "Content-Type" "text/html" noserver "Cache-Control" "no-store, no-cache"
      }
  }
```

#### 5. Apply the iRule to the virtual server
* Go to "Local Traffic" --> "Virtual Servers" and select the your virtual server
* Click on the "Resources" tab
* Under the "iRules" section, click "Manage"
* Select the "maint-page-irule" and click "Finished"
