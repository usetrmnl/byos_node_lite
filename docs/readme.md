# Image server for TRMNL built with Node.js, JSX and HTML
![Tests](https://github.com/usetrmnl/byos_node_lite/actions/workflows/tests.yml/badge.svg)

Create custom screen on your e-ink TRMNL device just like website: collect data with JavaScript and design layout with JSX(React) or HTML
with Liquid templates.

This repository is starter for Node.js server, that generates an image that you can use
via [Redirect](https://help.usetrmnl.com/en/articles/11035846-redirect-plugin)
or [Alias](https://help.usetrmnl.com/en/articles/10701448-alias-plugin) plugins, or run it as
own [BYOS](#bring-your-own-server-byos).

<img src="preview.png" alt="preview">

## Features

- Creating screens with JSX or HTML (Liquid)
- Design Framework from TRMNL
- Generating screen (image) with preview
- JSON Data API
- BYOS (auto-provisioning, screens, device logs)
- Migrating any Plugin (they are in Liquid)
- Playlists (set of screens)
- Proxying screens (plugins) from Core
- Docker
- no database required (everything in ENV and Config)
- no dashboard

## Quick Start

1. Press button `Use this template` on Github, or clone this repository
2. Copy .env.example to .env.local and change values to yours
3. (optional) setup local Node.js and NPM, for example via [nvm](https://github.com/nvm-sh/nvm)
4. Run `npm run watch`
5. See preview of the screen in browser http://127.0.0.1:3000/image?secret_key=...

Or run it via Docker:

```shell
docker build --no-cache -t trmnl . && docker run --env-file .env.local -p 3000:3000 trmnl
```

After run, you can change files in `src/Template` and `src/Data` to something that you want to display.

Later, to display screen on device you would need to [deploy](#your-server), provide [endpoints](#endpoints)
in plugin settings, or setup your device to [BYOS](#bring-your-own-server-byos).

--------

## Technologies used:

- [Headless Chrome](https://pptr.dev) for rendering HTML or JSX to image
- [Liquid](https://shopify.github.io/liquid/) for HTML templating (same used by usetrmnl.com)
- [React](https://react.dev/reference/react-dom/server/renderToString) if you want to use JSX components
- [Express.js](https://expressjs.com) as API server
- [TSX](https://tsx.is) for supporting JSX/TSX files

## Endpoints

**Image** https://yourserver.com/image?secret_key=... <br>
↑ can be used for preview and [Alias](https://help.usetrmnl.com/en/articles/10701448-alias-plugin) plugin

**JSON** https://yourserver.com/plugin/redirect?secret_key=... <br>
↑ can be used for [Redirect](https://help.usetrmnl.com/en/articles/11035846-redirect-plugin) plugin

For BYOS work routes are:

```code
/api/setup - auto-provisioning of new device
/api/display - JSON with link to Image endpoint
/api/log - receiving logs and displayind them app logs (stdout)
```

## Liquid templates

For putting variables into HTML we use Liquid templating library (same as TRMNL).<br>
Its a great option for moving custom plugins from usetrmnl.com to your own server.<br>
Liquid files are in `src/Template` directory with `.liquid` extension. Most of their contents is regular HTML.<br>
[Liquid 101 article](https://help.usetrmnl.com/en/articles/10671186-liquid-101) from TRMNL.<br>
See example [here](../src/Template/HackerNews.liquid)

## JSX components

- You can use regular JSX components (similar to React), but without hooks, as screen is rendering only once
- Starting point is [App.tsx](../src/Template/JSX/App.tsx)

After choosing between Liquid and JSX, you can disable another in [Screen.ts](../src/Screen/Screen.ts)

## Design Framework

You have full support of JavaScript and CSS, so you can use [Framework](https://usetrmnl.com/framework) by TRMNL:

```html

<html>
<head>
    <link rel="stylesheet" href="https://usetrmnl.com/css/latest/plugins.css">
    <script src="https://usetrmnl.com/js/latest/plugins.js"></script>
</head>
<body class="environment trmnl">
<div class="screen">
    ...
</div>
```

See [Header](../src/Template/Header.html) for adding styles, fonts, etc and screen (plugin)
example [HackerNews.liquid](../src/Template/HackerNews.liquid)

## Images and other static files

You use any static files in folder [assets](../assets) - at moment of screen rendering they will be matched with
relative links in HTML, that started with '/assets/...'

Example:

```html
<img src='/assets/images/wallpaper.jpeg'/>
```

## Data for Templates

It's easier to collect all variables and data in one place, before using it in templates.<br>
You can collect it in any methods or files and assemble results in in [PrepareData.ts](../src/Data/PrepareData.ts).<br>
Example: [getting HackerNews posts](../src/Data/HackerNewsData.ts).

## Your Server

To run your TRMNL server you need any form of server (VM, droplet, pod, instance) somewhere, for example AWS, Google
Cloud, Digital Ocean.

- it can be run via [Dockerfile](../Dockerfile) or command `npm run start`
- Better to pass ENV parameters from secure storage that your cloud provider has

## Bring your own server (BYOS)

You can skip using plugins and connect your device to your server directly by
using [BYOS](https://docs.usetrmnl.com/go/diy/byos) mode. After changing configuration to BYOS your device will stop
doing API requests to [usetrmnl.com](https://usetrmnl.com) servers. You can even close it in private network for data
security.

This repo implements basic BYOS server for one device.<br>

You can enable it with those steps:

1. Set `BYOS_ENABLED = true` in [Config](../src/Config.ts)
2. Put your device's MAC value to ENV key BYOS_DEVICE_MAC (it can be via .env.local). If you don't know it - you can do
   this step later.
3. Generate or use usetrmnl.com access token in ENV key BYOS_DEVICE_ACCESS_TOKEN
4. Hold round button on your device for more than 5 seconds - you should see connection instructions on screen.
5. Connect your phone to wifi called `TRMNL`
6. On setup choose `Custom server` (see screenshot below) and provide address of your server. In case of local computer
   in same WiFi network it would be something like http://192.168.0.26:3000

<img src="BYOS_setup.png" alt="BYOS setup" height="400">

Troubleshooting:

- If you see `[404] GET /api/display` in logs - then you might forgot to enable BYOS in [Config](../src/Config.ts)
- If you see `Failed to check image` - maybe you forgot to change `PUBLIC_URL_ORIGIN` in ENV
- To find out MAC address for step 1 you can check logs of server: it will be attempts to connect
- If you see error `wrong access-token value from device` - you may need to click button `Soft reset` on device setup
  stage

## Returning from BYOS to cloud or updating firmware
Switching requests back to usetrmnl.com
1. Hold round button on your device for more than 5 seconds - you should see connection instructions on screen.
2. Connect your phone to wifi called `TRMNL`
3. On setup screen press button “Soft reset”. Device should restart after that
4. Connect your phone to wifi called `TRMNL` again
5. On setup screen choose wifi and fill password only (no custom server)

## BYOS Proxy

You can use both own server with own screen and some plugins from usetrmnl.com (aka core) by using Proxy mode. In this
mode you will see your screen, one of screens from core after, and etc, one by one.

You can enable it with those steps:

1. Set `BYOS_PROXY = true` in [Config](../src/Config.ts)
2. Repeat setup process for `BYOS` with doing `soft reset`
3. Check `access-token` received from usetrmnl.com and change ENV key BYOS_DEVICE_ACCESS_TOKEN to it

## CONTRIBUTING

See [CONTRIBUTING.md](./CONTRIBUTING.md)
