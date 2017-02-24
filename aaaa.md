<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [What's this project?](#whats-this-project)
- [What changes were made?](#what-changes-were-made)
  - [Adding in a service worker](#adding-in-a-service-worker)
  - [Adding in a Web App Manifest](#adding-in-a-web-app-manifest)
  - [Adding in a GitHub deployment step](#adding-in-a-github-deployment-step)
- [What additional changes might be needed?](#what-additional-changes-might-be-needed)
  - [I've added in React Router and now my URLs don't work offline](#ive-added-in-react-router-and-now-my-urls-dont-work-offline)
  - [I'm using cross-origin APIs or resources, and they aren't working while offline](#im-using-cross-origin-apis-or-resources-and-they-arent-working-while-offline)
- [How can I try out the Progressive Web App?](#how-can-i-try-out-the-progressive-web-app)
- [How can I report bugs?](#how-can-i-report-bugs)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## What's this project?

This is one approach to taking a project created with the
[`create-react-app`](https://github.com/facebookincubator/create-react-app) tool
and adding in some additional bits commonly found in
[Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/).

## What changes were made?

GitHub's visual diff shows
[all the changes needed](https://github.com/jeffposnick/create-react-pwa/compare/c-r-a-0.6.0...c-r-pwa-0.6.0)
to make a Progressive Web App. They include:

### 添加 service worker

The service worker that's generated will ensure that the local images,
JavaScript, CSS, and HTML for your web app will be cached and continue to work,
even when a user is offline. It also will save bandwidth and improve performance
while users are online, by only making network requests for those local
resources when there's actually an update that's been deployed.

service worker会被产生，用于保证你的web应用的本地图片、JavaScript, CSS, 和 HTML会被缓存病继续有效。
甚至当用户处于离线的情况。它在用户在线时，也会节省带宽和提高性能，当有实际的更新被部署时，仅仅会对那些本地的资源发送网络请求。

A [`sw-precache`](https://github.com/GoogleChrome/sw-precache) dependency was added to `package.json`, and the
`npm run build` command has been updated to call the `sw-precache` command-line
interface after the `webpack` build process is completed.

[`sw-precache`](https://github.com/GoogleChrome/sw-precache)作为依赖被添加到`package.json`里，`npm run build`命令已经更新`webpack`构建处理完成之后，去调用`sw-precache`命令.

`sw-precache` generates a `service-worker.js` file that will automatically
cache the other static files in the `build/` directory, and keep them up to
date when you deploy changes.

`sw-precache`会生产`service-worker.js`文件，它会自动换成其他的`build/`里的static文件，保证他们在你部署变更时是最新的。

Code to register the service worker was also added to `index.html`.

注册service worker的代码也被加到了`index.html`里。

The service worker is only generated as part of the production build, so the
development environment will continue to work as before. When running a
production server locally, make sure you use a different port than `3000`, to
ensure the service worker does not inadvertantly take control of the development
environment.

service worker仅仅会作为线上构建的异步而产生，所以开发环境下回和之前一样有效。当在本地运行线上server时，确保你使用的不是`3000`端口，用于保证service worker在开发环境不被显示控制。


### 添加 Web App Manifest

A [web app manifest](https://developers.google.com/web/updates/2014/11/Support-for-installable-web-apps-with-webapp-manifest-in-chrome-38-for-Android?hl=en)
provides metadata about your web app. Along with a service worker, your web
app needs a manifest in order to trigger the Add to Homescreen prompt (in
supported browsers).

[web app manifest](https://developers.google.com/web/updates/2014/11/Support-for-installable-web-apps-with-webapp-manifest-in-chrome-38-for-Android?hl=en)提供你的webapp的元数据。伴随service worker，你的web应用需要一个清单以便能够触发添加到主屏幕提示（在支持的浏览器里）

## What additional changes might be needed?

By following the [changes made](https://github.com/jeffposnick/create-react-pwa/compare/c-r-a-0.6.0...c-r-pwa-0.6.0),
you should end up with a Progressive Web App using React that's ready to be
deployed to any static hosting environment. However, if you add in additional
functionality to the starting point, you may need to update your `sw-precache`
settings to ensure the service worker behaves properly.

通过[修改模式](https://github.com/jeffposnick/create-react-pwa/compare/c-r-a-0.6.0...c-r-pwa-0.6.0)，你会结束使用React的已经不是到任意静态主机环境的PWA。然而，如果你添加额外功能给起始点，你可能需要更新你的`sw-precache`设置来保证service worker正确处理。

### I've added in React Router and now my URLs don't work offline

If you've followed the suggestions in the `create-react-app` documentation
and added in [React Router](https://github.com/reactjs/react-router) using the
History API to manage URLs, then you need to tell the service worker that
navigations to all the random URLs your web app now supports should actually
be fulfilled with the cached copy of your `index.html`. You can do this
with the [`navigateFallback`](https://github.com/GoogleChrome/sw-precache#navigatefallback-string)
option in `sw-precache`.

Assuming you're using the `sw-precache-config.js` configuration file, the
additional option would look like `navigateFallback: 'index.html'`.

如果你已经遵守了这些在`create-react-app`文档里的建议，并添加[React Router](https://github.com/reactjs/react-router)使用History API管理URL，你需要告诉service worker导航到所有你的webapp刺客支持的随机URL应该被填写为`index.html`.的换成副本。你可以使用`sw-precache`里的[`navigateFallback`](https://github.com/GoogleChrome/sw-precache#navigatefallback-string)来做这个。

### I'm using cross-origin APIs or resources, and they aren't working while offline

The service worker generated by default by `sw-precache` only handles requests
for local, static resources, like your images, JavaScript, CSS, and HTML.
Requests made at runtime for, e.g., APIs or images that live on other servers
won't be handled by the default `sw-precache` setup. This means they will stop
working offline.

The [`sw-toolbox`](https://github.com/GoogleChrome/sw-toolbox) allows you to
set up runtime caching strategies, using URL patterns to determine what
strategy and cache sizes to use. `sw-precache` provides an easy way to use
`sw-toolbox` via the [`runtimeCaching`](https://github.com/GoogleChrome/sw-precache#runtimecaching-arrayobject)
configuration option.

service worker默认是有`sw-precache`生产，仅仅处理本地的请求，静态资源，比如你的图片，JavaScript, CSS, and HTML.
二运行时的请求，比如API或位于其他服务器的图片，默认 `sw-precache`配置是不会处理他们的。这意味着他们会在离线模式下无法工作。

[`sw-toolbox`](https://github.com/GoogleChrome/sw-toolbox) 运行你设置运行时缓存策略，使用URL patterns来决定策略和要使用的缓存大小。`sw-precache`提供了一种简单的方式来使用`sw-toolbox`，通过[`runtimeCaching`](https://github.com/GoogleChrome/sw-precache#runtimecaching-arrayobject)配置选项即可。

## How can I try out the Progressive Web App?

The code from this project is deployed at https://jeffy.info/create-react-pwa/

## How can I report bugs?

If you've found a bug in the code output by `create-react-app`, please let the
project maintainers know in their [issue tracker](https://github.com/facebookincubator/create-react-app/issues).

If you've found an issue specific to the Progressive Web App bits (the service
worker, the Web App Manifest, etc.), then please let us know in this project's
[issue tracker](https://github.com/jeffposnick/create-react-pwa/issues).


Google PWA Study （2016年7月） https://developers.google.com/web/showcase/2016/service-worker-perf

Google developer pages https://developers.google.com/web/progressive-web-apps/

Develop your first progressive web app https://developers.google.com/web/fundamentals/getting-started/codelabs/your-first-pwapp/


如果想查看google的技术文档还请翻墙查看: https://developers.google.com/web/updates/2015/12/getting-started-pwa
这里还有一些关于PWA的资料收集: https://github.com/ljinkai/pwa-collection 

