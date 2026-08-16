# Sports Prediction Platform — Ecosystem

**2014–2018 · PHP, Python, MySQL, JavaScript, Ionic**

A custom-built sports prediction platform with its own internal 
framework — covering routing, session handling, scheduling, 
deployment, and security — built end-to-end without relying on an 
off-the-shelf PHP framework. Extended with a companion mobile app 
and Python-based backend.

> **Source code:** Not publicly available. This repository documents 
> the architecture and selected implementation details.

## Overview

The platform combines a public-facing web application (predictions, 
rankings, admin-managed content) with:

* a companion mobile app (Ionic/Android) backed by a dedicated Python API ([see separate repository](https://github.com/elast-net/2017-mobile-api))
* third-party integrations (social, messaging, payments)
* a custom task-scheduling mechanism working around shared-hosting cron limits
* an isolated deployment/testing pipeline
* a dedicated code-obfuscation toolchain for production builds ([see separate repository](https://github.com/elast-net/2014-php-obfuscator))

```text
                     ┌─────────────────┐
                     │   Web frontend  │
                     └────────┬────────┘
                              │
                    ┌─────────┴───────────┐
                    │   PHP core / own    │
                    │  routing framework  │
                    └──┬──────────────┬───┘
                       │              │
              ┌────────┘              └───────────┐
              ▼                                   ▼
     Internal PHP API                    Third-party integrations
              ▲                          (Twitter, Facebook, SMS,
              │                            payment provider)
    ┌─────────┴──────────┐
    │   Python API       │
    │  (mobile backend)  │
    └─────────┬──────────┘
              │
              ▼
        Mobile App (Ionic)
```

## Own Framework / Core Architecture

Rather than using an off-the-shelf PHP framework, the platform was 
built on a custom internal framework covering:

* request routing
* session management (see *Security & Hardening*)
* database access layer
* a lightweight templating approach
* a custom results calculator engine (see note below)

Code was organized into clearly separated directories: reusable 
components (`inc`), configuration (`cfg`), and the admin panel (`adm`).

## Third-Party Integrations

The platform integrated with:

* **Twitter API** and **Facebook API** — for content distribution
* **SMS gateway API** — for user notifications
* **Payment provider API** — for premium features

## Scheduling Under Hosting Constraints

Shared hosting only allowed cron execution every 5 minutes. Rather 
than accept that granularity, a custom task-distribution mechanism 
queued pending tasks and carried them across invocations, achieving 
effective per-minute execution within that constraint:

```php
// simplified illustration — not the original implementation
function runScheduledBatch() {
    $tasks = loadPendingTasks();

    foreach ($tasks as $task) {
        if (dueNow($task)) {
            executeTask($task);
            markDone($task);
        }
    }

    // tasks not yet due are simply left in the queue,
    // to be picked up by the next 5-minute cron trigger
    persistRemainingTasks($tasks);
}
```

The same mechanism also handled backups, log/database cleanup, and 
periodic maintenance jobs.

## Mobile App & Cross-Language API

A companion mobile app (Ionic/Android) is backed by a dedicated 
Python API ([see separate repository](https://github.com/elast-net/2017-mobile-api)). To avoid duplicating core 
business logic across two languages, the Python API communicates 
with a dedicated **internal PHP API**, which triggers the same 
operations otherwise handled natively by the web application's PHP 
codebase — allowing both platforms to share logic without 
reimplementing it.

## Results Calculator

The platform includes a custom-built results calculator, a distinct, 
well-known tool within this niche sports community.

> Implementation details are not published, as the calculator's 
> logic is specific to this niche and not intended for reuse by 
> competing services.

## Security & Hardening

Several defensive measures beyond standard practice for the platform's scale:

* **Honeypot admin panel** — a decoy admin path, with the real 
  administration panel located elsewhere
* **Isolated session storage** — sessions stored in a dedicated 
  directory rather than shared server tmp
* **IP/user-agent-independent session cookies**
* **Two-stage code obfuscation** for deployed production code — see 
  the [dedicated obfuscation toolchain repository](https://github.com/elast-net/2014-php-obfuscator)
* **Sanitized database exports** — structure exports with selected 
  content replaced, for safe sharing/backup
* **Automated installer** with three-stage setup (file structure, 
  SQL configuration, access control via index.html/htaccess/IP 
  whitelisting) and full test/production environment isolation

A simplified illustration of the installer's staged approach:

```php
// simplified illustration — not the original implementation
copyFileStructure($sourceDir, $targetDir);
copySqlConfiguration($sqlConfigPath, $targetDir);
setupAccessControl($targetDir); // index.html, .htaccess, IP whitelist
```

## Deployment Pipeline

Production code passed through a dedicated obfuscation toolchain 
before deployment — see: **[PHP Code Obfuscator with Real-Time Watch 
Mode](https://github.com/elast-net/2014-php-obfuscator)** for details on that system, which strips comments/
whitespace, renames identifiers, and packages the result for release, 
with an optional real-time watch mode for active development.

## Historical Context

Originally developed in **2014** and maintained until **2018**. The 
implementation reflects PHP web development practices of that period, 
predating the widespread adoption of frameworks like Laravel/Symfony 
in smaller/independent projects, and built before Docker-based 
deployment workflows became standard.

## What the Project Demonstrates

* End-to-end platform design without relying on an existing framework
* Third-party API integration (social, messaging, payments)
* Creative engineering under infrastructure constraints (cron scheduling)
* Defensive security design (honeypot, session isolation, obfuscation)
* Cross-language architecture (PHP + Python sharing business logic)
* Full deployment/testing pipeline automation

**Status:** Historical production project · source code not publicly 
available
