# Gapp
<img src="https://user-images.githubusercontent.com/48908942/190998832-7b8d072a-c3fe-4676-ba26-9d3efe2895fb.png" alt="Gapp Logo" height="250px" width="250px" class="js-lazy-loaded">

[![Twitter](https://img.shields.io/badge/Twitter-TICgal-blue.svg?style=flat-square)](https://twitter.com/ticgalcom)
[![Web](https://img.shields.io/badge/Web-TICgal-blue.svg?style=flat-square)](https://tic.gal/)
[![Localazy](https://img.shields.io/badge/Translate-Localazy-cyan)](https://localazy.com/p/gapp-multiplatform)

**A GLPI Self-service mobile client.**

RTFM before opening an issue (Friendly reminder :)): https://tic.gal/gapp/

Bug reporting for Gapp.
Please report anything related with Gapp here.

Plugin v3.1.0 not working with GLPI 11.0.7 - pluginList endpoint returns ERROR_RESOURCE_NOT_FOUND_NOR_COMMONDBTM

## Environment

- **GLPI version**: 11.0.7
- **GappEssentials version**: 3.1.0 (main branch)
- **PHP**: 8.x
- **Web server**: Apache 2.4 (DocumentRoot: /var/www/glpi/public)
- **Gapp mobile app**: 3.0.0 (2.0678.00) — Self-Service edition
- **Deployment**: GLPI in ECS container, Apache routing all requests through Symfony (`public/index.php`)

## Problem description

After installing GappEssentials 3.1.0 from the main branch and activating it on GLPI 11.0.7, the Gapp mobile app fails to log in with the error:

> **"Gapp essentials must be installed and active. [Plugin]"**

The app login flow completes `initSession` and `getGlpiConfig` successfully, but fails at `pluginList`.

## Investigation

Calling the `pluginList` endpoint directly via curl using the standard GLPI 11 legacy API path:

```bash
curl -H "Session-Token: <valid_token>" \
  "https://<glpi_host>/api.php/v1/pluginList"
```

Returns:

```json
["ERROR_RESOURCE_NOT_FOUND_NOR_COMMONDBTM", "resource not found or not an instance of CommonDBTM..."]
```

The same behavior occurs with `apirest.php` path (which is rewritten by Symfony in GLPI 11).

When calling the plugin's controller directly via `/plugins/gappessentials/apirest.php/pluginList`, the controller IS reached (different error returned), but the `Session-Token` header is not parsed by the plugin:

```json
["ERROR_SESSION_TOKEN_MISSING", "session_token parameter is missing or empty..."]
```

This suggests the Symfony Route registered by `ApiRestController.php` (`#[Route("/apirest.php{request_parameters}")]`) is not being matched by requests that the mobile app sends to `/api.php/v1/pluginList`.

## Plugin state

- Plugin is activated (visible in Setup → Plugins, status = active, version = 3.1.0)
- `Configure → General → Gapp Essentials` tab is functional
- GappEssentials config (Origem da requisição = Helpdesk) saves successfully

## Server logs

`/var/glpi/log/api.log` shows the app reaching `getGlpiConfig` with user context, but `pluginList` calls do not include the user context, indicating the request is being rejected before reaching the plugin's controller.

## Question

Is the `ApiRestController` route definition (`/apirest.php{request_parameters}`) intended to also handle requests coming from `/api.php/v1/`? If so, is there an additional registration step required in GLPI 11.0.7 that may have changed?

If this is a known issue and a fix is in progress, is there an ETA for v3.1.0 release on the Marketplace?

Thanks for the great work!

