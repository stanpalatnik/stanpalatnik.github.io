---
layout: post
title: "Using Express.js and Dust.js With Node.js"
date: 2012-03-22T00:00:00-04:00
---

Express.js is the most full-featured and popular Node.js open source framework. However, its default template library of choice - Jade, is far from impressive. Not only does it use it's own syntax(you cannot use plain HTML), but it is limited to server-side rendering. In this post, we will setup Dust.js with Express.js for server-side rendering, as well as discuss a possible implementation with client-side rendering.

<strong>Server-side rendering</strong>

Dust.js can easily be integrated into the latest Express.js version using the Consolidate.js module. This integration could easily be completed without this module, however, Consolidate.js performs the exact functionality we need, including support for caching.

For this example, we are using the latest Express 3.0 alpha source from Github.

[gist id="1968391"]

Note: Make sure to install the latest Consolidate.js from the Github source, since Dust.js support has been added only recently, and is not available in the node package manager just yet.

This is my code for app.js for my sample Node.js application. First we include all the required modules. We set our default template engine with the app.engine command. We pass in the default extension of all the template file-names, as well as the default engine that will be used to render them.

We can set the default file-name extension by setting the 'view engine' property. This is a very convenient feature. Let's say our file-names end with .dust. Instead of specifying 'filename.dust', we can just pass in 'filename'.

Here is my 'index' page:

[gist id="1967577"]

After this index page gets rendered through dust.render(), it will replace all the keys and conditional statements with plain HTML code. For example, let's say I added username: 'Stan' to the response object. Dust.js would detect that the username token is present, and would show my username stead of the login form!

<strong>Dust.js Crash Course</strong>

You might not recognize some symbols or tags in that HTML code. That's because they are used by the Dust.js parser to add additional functionality(conditionals, includes). Let's start with the title. <em>{+title}7th-Degree{/title}. </em>First let's ask ourselves what problem we are trying to solve. We want to have a standardized header and footer. We want to dynamically change the title in each page, without having to change it manually in the layout.

By surrounding the title with {+title} tags, we can dynamically change the title by passing the variable 'title' during the request. If you look back at the previous Gist, we are calling res.render, and passing in  title: 'Testing out dust.js server-side rendering'. Dust.js will see this and dynamically replace the title within those tags on the server. When the client(or crawler) loads this page, they will only see the ladder title. Similarly, the {+http_header/} tag serves the same purpose. But this case, we don't have a default value, so we open and close the tag to create a placeholder to add additional more information in the header, like additional JavaScripts of stylesheets.

Conditional tags:

[gist id="1968397"]

Another great feature of Dust.js is support for conditionals. Putting ? in front of a tag is similar to checking if the variable the tag represents has been set. If not, we render the content in after the {:else}.

&nbsp;

<strong>Client-side rendering:</strong>

A recent case study by LinkedIn shows the true benefits of client-side rendering: <a title="How LinkedIn Leverages Dust.js" href="http://engineering.linkedin.com/frontend/leaving-jsps-dust-moving-linkedin-dustjs-client-side-templates" target="_blank">How LinkedIn leverages Dust.js</a>.They make a strong case in support of Dust.js. One of the major advantages is removing load from your servers. Client-side templates can be served from your CDN network, while the data that populates them can come straight from your server. In this scenario, you get the best of both worlds. Offloading bandwidth to CDN servers while constantly serving the most updated content. Additionally, you are offloading the computational power to parse and render the templates on the client's browser. This results in reduced server latency. The major downside is that you have to include the entire Dust.js framework inside the webpage, which adds considerable overhead for slow internet connections and under-powered devices.

Let's take our previous example and port it to client-side rendering. In this case, we do not need to load the consolidate module on the server. We do not need to parse any variables on the server. So what's the catch? In order to enable client-side rendering, we need to compile the dust templates into JavaScript files before we serve them to the client.That means that every time we make a change to the template, we need to compile it into JavaScript. Compiling can be done with the dust.compile() command. The alternative would be to compile the HTML source each time on the client. This is most efficient approach for the server, it does not have to worry about tracking changes to the template. However, compiling a complicated template on the client will create substantial overhead for the browser.

[gist id="1968827"]

In this simple example, we are compiling and rendering the template all inside the client. First we compile the template into JavaScript that Dust.js can understand. Then we render the template using Dust.js. In this case, the engine performs a variable substitution to replace {name} with David. In this case, we maximized the amount of work the client does, which minimizes the work for us on the server. However, there are some issues to pure client-side rendering:
<ol>
	<li><strong>Increased Bandwidth</strong> - The entire Dust.js framework is 26 kb. If we remove client-side rendering functionality, the size is drastically reduced to 6 kb. That extra 20 kb will substantially increase bandwidth usage.</li>
	<li><strong>Search Engine Optimization</strong> - Search engine crawlers will not be able to use the template because they cannot execute JavaScript. However, in the LinkedIn article mentioned earlier in the article, they address this issue by falling back to server-side rendering if the client does not support JavaScript. This solves the crawler issue, as well as client that does not support JavaScript these days.</li>
</ol>
<a href="https://github.com/akdubya/dustjs/blob/master/docs/api.md" target="_blank">Click here to check out more information about the Dust.js API</a>

The major takeaways from this post is that Dust.js is a great templating engine. It's powerful, versatile, and efficient. And with Express 3.0, it's a breeze to setup.