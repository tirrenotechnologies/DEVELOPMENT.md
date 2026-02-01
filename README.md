# tirreno for developers

Crash course for new tirreno developers

---

## Welcome

Welcome and thank you for your interest in [tirreno safety platform](https://www.tirreno.com). The tirreno community is open and we welcome contributions of code and ideas.

tirreno is available in three editions:

- **Community Edition** (open-source): For developer teams that want to add a security layer to self-hosted applications. Get started today without getting into complex business relationships. Licensed under GNU Affero General Public License v3 (AGPL-3.0).

- **Application Edition**: Protects your organization's internal applications from account threats, ensures audit trails and field history for compliance, detects data exfiltration and insider threats.

- **Platform Edition**: Built for client portals, SaaS, public sector portals, and digital platforms. Multi-application support, fraud and abuse prevention, and dedicated assistance for your SOC, product, and risk teams.

For Application and Platform editions, contact team@tirreno.com.

```
     Community ──────────────► Application ────────────► Platform
       Edition                    Edition                  Edition
         │                         │                         │
         ▼                         ▼                         ▼
    Personal apps             Internal apps            External-facing
    + Basic security          + Compliance             + Multi-app
    + Development             + Audit trails           + Fraud/abuse
    + No support              + Insider threats        + Dedicated support
```

Here is some basic information for new developers to get up and running quickly:

* Here is an overview of the [tirreno system architecture](#tirreno-system-architecture)
* Here is an overview of the tirreno [coding standards](#contributing)
* Here is [administration guide](https://github.com/tirrenotechnologies/ADMIN.md) 

* Here is an overview of [how to customize tirreno](#risk-rules--customization) for your needs
* The easiest way to get started on development is documented in [local development setup](#local-development-setup)
* Contributed code should be released under the GNU Affero General Public License v3 (AGPL-3.0)
* To get started, feel free to introduce yourself at the [Mattermost community](https://chat.tirreno.com) or email the team at team@tirreno.com

---

## Table of contents

1. [System architecture](#tirreno-system-architecture)
   - [Introduction](#introduction)
   - [Overview](#overview)
   - [Technology stack](#technology-stack)
   - [System requirements](#system-requirements)
   - [Directory structure](#directory-structure)

2. [API integration](#api-integration)
   - [Official tracker libraries](#official-tracker-libraries)
   - [API reference](#api-reference)

3. [Integration guide](#integration-guide)
   - [Why send events to tirreno?](#why-send-events-to-tirreno)
   - [Integration planning](#integration-planning)
   - [Security considerations](#security-considerations)
   - [Quick start](#quick-start)
   - [Event tracking best practices](#event-tracking-best-practices)
   - [Send all logged-in user events](#send-all-logged-in-user-events)
   - [Protecting the registration](#protecting-the-registration)
   - [Protecting the login](#protecting-the-login)
   - [Auto-ban abusive IPs](#auto-ban-abusive-ips)
   - [Field audit trail](#field-audit-trail)
   - [Testing your integration](#testing-your-integration)

4. [Risk rules & customization](#risk-rules--customization)
   - [Built-in rules](#built-in-rules)
   - [Developing custom rules](#developing-custom-rules)
   - [Ruler operators reference](#ruler-operators-reference)
   - [Rule context attributes](#rule-context-attributes)
   - [Suspicious pattern lists](#suspicious-pattern-lists)
   - [UI constants](#ui-constants)

5. [Contributing](#contributing)
   - [Source code](#source-code)
   - [Contributor license agreement (CLA)](#contributor-license-agreement-cla)
   - [Git workflow](#git-workflow)
   - [Local development setup](#local-development-setup)
   - [Code quality tools](#code-quality-tools)
   - [PHP coding standards](#php-coding-standards)
   - [Template syntax](#template-syntax)
   - [Internationalization (i18n)](#internationalization-i18n)
   - [JavaScript coding standards](#javascript-coding-standards)
   - [File formatting](#file-formatting)
   - [Code comments](#code-comments)
   - [Commit messages](#commit-messages)
   - [Line endings](#line-endings)
   - [Testing](#testing)

6. [Resources](#resources)

7. [Found a mistake?](#found-a-mistake)

---

## tirreno system architecture

### Introduction

tirreno is a PHP/PostgreSQL application using Fat-Free Framework (F3). Lightweight MVC for safety analytics, security analytics and threat detection.

### Overview

```
 ┌──────────┐      request       ┌─────────────────┐      POST /sensor/       ┌─────────────────┐
 │   User   │ ─────────────────▶ │    Your App     │ ────────────────────────▶│    tirreno      │
 └──────────┘                    │  (allow/deny)   │◀──────────────────────── │  • Risk scoring │
                                 └─────────────────┘      response            │  • Rule engine  │
                                                                              │  • Blacklist    │
                                                                              └─────────────────┘
```

Your application sends user events (logins, registrations, page views, field changes) to tirreno. tirreno analyzes the events, calculates risk scores, and can automatically blacklist suspicious users. Your app can query the blacklist API to block bad actors in real-time.

### Technology stack

Core dependencies (`composer.json`):

| Dependency | What it does |
|------------|--------------|
| **bcosca/fatfree-core** | Fat-Free Framework (F3) |
| **matomo/device-detector** | Device/browser/OS detection |
| **ruler/ruler** | Rules engine |

Dev tools: **phpstan** (static analysis), **php_codesniffer** (style)

### System requirements

- **PHP** 8.0–8.3 with PDO_PGSQL, cURL, mbstring
- **PostgreSQL** 12+
- **Apache** with mod_rewrite

Hardware: 512 MB RAM for PostgreSQL (4 GB recommended), ~3 GB storage per 1M events.

### Directory structure

```
tirreno/
│
├── app/                        # Application code
│   ├── Assets/                 # Rule base classes
│   │   └── Rule.php            # Abstract Rule class
│   │
│   ├── Controllers/            # Request handlers
│   │   ├── Admin/              # Admin panel controllers
│   │   │   ├── Base/           # Base controller classes
│   │   │   ├── Events/         # Events module
│   │   │   ├── Rules/          # Rules module
│   │   │   ├── Users/          # Users module
│   │   │   └── ...             # Other admin modules
│   │   ├── Api/                # API controllers
│   │   │   ├── Blacklist.php   # Blacklist API
│   │   │   └── Endpoint.php    # API endpoint handler
│   │   ├── Pages/              # Page controllers
│   │   │   ├── Login.php
│   │   │   ├── Signup.php
│   │   │   └── ...
│   │   ├── Cron.php            # Cron controller
│   │   └── Navigation.php      # Navigation controller
│   │
│   ├── Crons/                  # Background job handlers
│   │   ├── Base.php            # Base cron class
│   │   ├── BatchedNewEvents.php
│   │   ├── EnrichmentQueueHandler.php
│   │   ├── RiskScoreQueueHandler.php
│   │   └── ...                 # Other cron jobs
│   │
│   ├── Dictionary/             # Internationalization (i18n)
│   │   └── en/                 # English translations
│   │       ├── Pages/          # Page-specific translations
│   │       ├── Parts/          # Component translations
│   │       └── All.php         # Combined translations
│   │
│   ├── Interfaces/             # PHP interfaces
│   │   ├── ApiKeyAccessAuthorizationInterface.php
│   │   ├── ApiKeyAccountAccessAuthorizationInterface.php
│   │   └── FraudFlagUpdaterInterface.php
│   │
│   ├── Models/                 # Database models (extend BaseSql)
│   │   ├── BaseSql.php         # Base class with execQuery()
│   │   ├── Device.php          # Device/user-agent model
│   │   ├── Grid/               # Grid data models
│   │   ├── Chart/              # Chart data models
│   │   ├── Enrichment/         # Enrichment models
│   │   └── ...                 # Other models
│   │
│   ├── Updates/                # Database migration handlers
│   │
│   ├── Utils/                  # Utility classes
│   │   ├── ApiKeys.php         # API key utilities
│   │   ├── Constants.php       # Application constants
│   │   ├── Logger.php          # Logging utilities
│   │   ├── Rules.php           # Rule utilities
│   │   └── ...                 # Other utilities
│   │
│   └── Views/                  # View helpers
│
├── assets/                     # Static assets and rules
│   ├── rules/                  # Rules engine
│   │   ├── core/               # Core rule definitions
│   │   └── custom/             # Custom rule definitions
│   ├── lists/                  # Suspicious pattern lists
│   │   ├── url.php             # URL attack patterns
│   │   ├── user-agent.php      # User agent patterns
│   │   ├── email.php           # Email patterns
│   │   └── file-extensions.php # File extension categories
│   ├── logs/                   # Application logs
│   └── ...                     # CSS, images
│
├── config/                     # Configuration files
│   ├── config.ini              # Main configuration
│   ├── routes.ini              # Route definitions
│   ├── apiEndpoints.ini        # API endpoint definitions
│   ├── crons.ini               # Cron job configuration
│   └── local/                  # Local overrides
│
├── install/                    # Web-based installation wizard
│   └── index.php               # DELETE AFTER INSTALLATION
│
├── libs/                       # Third-party libraries (vendor)
│
├── sensor/                     # API endpoint for event ingestion
│
├── tmp/                        # Temporary files, cache
│
├── ui/                         # Frontend UI
│   ├── css/                    # Stylesheets
│   ├── images/                 # Static images
│   │   ├── icons/
│   │   └── flags/
│   ├── js/                     # JavaScript files
│   │   ├── endpoints/          # Page entry points
│   │   ├── pages/              # Page controllers
│   │   │   ├── Base.js         # Base page class
│   │   │   ├── Ips.js          # IPs page
│   │   │   ├── Events.js       # Events page
│   │   │   └── ...             # Other pages
│   │   ├── parts/              # Reusable components
│   │   │   ├── grid/           # Data grid components
│   │   │   ├── chart/          # Chart components (uPlot)
│   │   │   ├── panel/          # Detail panel components
│   │   │   ├── choices/        # Filter components (Choices.js)
│   │   │   ├── utils/          # Utility modules
│   │   │   │   ├── Constants.js
│   │   │   │   ├── String.js
│   │   │   │   └── Date.js
│   │   │   └── ...             # Other components
│   │   └── vendor/             # Third-party JS libraries
│   │       ├── jquery-3.6.0/
│   │       ├── datatables-2.3.2/
│   │       ├── uPlot-1.6.18/
│   │       ├── choices-10.2.0/
│   │       ├── jvectormap-2.0.5/
│   │       └── tooltipster-master-4.2.8/
│   └── templates/              # HTML templates
│       ├── layout.html         # Base layout
│       ├── pages/              # Page templates
│       │   ├── admin/          # Admin page templates
│       │   │   ├── events.html
│       │   │   ├── ip.html
│       │   │   ├── users.html
│       │   │   └── ...
│       │   ├── login.html
│       │   ├── signup.html
│       │   └── ...
│       ├── parts/              # Component templates
│       │   ├── headerAdmin.html
│       │   ├── footerAdmin.html
│       │   ├── leftMenu.html
│       │   ├── notification.html
│       │   ├── forms/
│       │   ├── panel/
│       │   ├── tables/
│       │   ├── widgets/
│       │   └── choices/
│       └── snippets/           # Code snippets (PHP, Python, etc.)
│
├── index.php                   # Application entry point
├── .htaccess                   # Apache URL rewriting rules
├── .profile                    # Environment profile
├── composer.json               # PHP dependencies
├── composer.lock               # Locked dependency versions
├── cron.json                   # Cron job definitions
├── phpcs.xml                   # PHP CodeSniffer configuration
├── eslint.config.js            # JavaScript linting configuration
│
├── AUTHORS.md                  # Project contributors
├── CHANGELOG.md                # Version history
├── CODE_OF_CONDUCT.md          # Community guidelines
├── LICENSE                     # AGPL-3.0 license
├── LEGALNOTICE.md              # Legal notices
├── README.md                   # Project overview
├── RELEASE_NOTES.md            # Release notes
├── SECURITY.md                 # Security policy
├── FILE_ID.DIZ                 # BBS-style file description
└── robots.txt                  # Search engine directives
```

---

## API integration

### Official tracker libraries

Use one of these:

| Language | Install |
|----------|---------|
| **PHP** | `composer require tirreno/tirreno-tracker` |
| **Python** | `pip install tirreno_tracker` |
| **Node.js** | `npm install @tirreno/tirreno-tracker` |

Repos: [PHP](https://github.com/tirrenotechnologies/tirreno-php-tracker), [Python](https://github.com/tirrenotechnologies/tirreno-python-tracker), [Node.js](https://github.com/tirrenotechnologies/tirreno-nodejs-tracker)

### API reference

#### Endpoint

```
POST /sensor/
Content-Type: application/x-www-form-urlencoded
Api-Key: YOUR_API_KEY
```

**Note:** The API uses form-urlencoded format, not JSON.

#### Required parameters

| Parameter | Description |
|-----------|-------------|
| `userName` | Unique user ID (max 100 chars) |
| `ipAddress` | IPv4/IPv6 address (invalid IPs default to `0.0.0.0`) |
| `url` | URL path (max 2047 chars) |
| `eventTime` | Timestamp `Y-m-d H:i:s.v` (defaults to current UTC if invalid) |

#### Optional parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `emailAddress` | string | Email address (max 255 chars). Validated and converted to lowercase |
| `userAgent` | string | Browser UA (max 511 chars) |
| `firstName` | string | User's first name (max 100 chars) |
| `lastName` | string | User's last name (max 100 chars) |
| `fullName` | string | User's whole name (max 100 chars) |
| `pageTitle` | string | Title of visited page (max 255 chars) |
| `phoneNumber` | string | User's phone number (max 19 chars) |
| `httpReferer` | string | Referer HTTP header value (max 2047 chars) |
| `httpMethod` | string | HTTP method: `GET`, `POST`, `HEAD`, `PUT`, `DELETE`, `PATCH`, `TRACE`, `CONNECT`, `OPTIONS`, `LINK`, `UNLINK` |
| `httpCode` | string | HTTP response status code (must be numeric, defaults to `0`) |
| `browserLanguage` | string | Detected browser language (max 255 chars) |
| `eventType` | string | One of the event types listed below. Defaults to `page_view`, or `page_error` if httpCode >= 400 |
| `userCreated` | string | User creation timestamp (`Y-m-d H:i:s` or `Y-m-d H:i:s.v`) |
| `payload` | array | Event details for `page_search` events |
| `fieldHistory` | array | Field edit history for `field_edit` events |

**Note:** Maximum length for all other parameters is 100 characters unless specified above. Parameters exceeding max length are truncated.

#### Event types

Default: `page_view` (or `page_error` if `httpCode` >= 400)

| Type | Description |
|------|-------------|
| `page_view` | Page visit (default) |
| `page_edit` | Page modification |
| `page_delete` | Page deletion |
| `page_search` | Search query |
| `page_error` | Error page |
| `account_login` | User authentication |
| `account_logout` | Session end |
| `account_login_fail` | Failed login attempt |
| `account_registration` | New account creation |
| `account_email_change` | Email address change |
| `account_password_change` | Password change |
| `account_edit` | Account profile modification |
| `field_edit` | Data modification |

#### Payload parameter

For `page_search` events:
```json
{
    "eventType": "page_search",
    "payload": {
        "field_id": 179280,
        "value": "search query",
        "field_name": "Country"
    }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `field_id` | Yes | Unique identifier for the search field |
| `value` | Yes | The search query string |
| `field_name` | No | Human-readable field name |

#### Field history parameter

Required for `field_edit` events. Must be an array of field change objects:
```json
{
    "eventType": "field_edit",
    "fieldHistory": [
        {
            "field_id": 179283,
            "new_value": "Paris",
            "field_name": "User city",
            "old_value": "London",
            "parent_id": "",
            "parent_name": ""
        }
    ]
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `field_id` | Yes | Unique identifier for the field |
| `new_value` | Yes | The new field value |
| `field_name` | No | Human-readable field name |
| `old_value` | No | The previous field value |
| `parent_id` | No | Parent record ID (for nested data) |
| `parent_name` | No | Parent record name |

**Note:** Missing required fields default to `"unknown"`. All values are converted to strings.

#### Blacklist API

Check if a user or IP is blacklisted:

**Request:**
```
POST /api/v1/blacklist/search
Content-Type: application/json
Api-Key: YOUR_API_KEY

{
    "value": "username_or_ip"
}
```

**Response:**
```json
{
    "value": "username_or_ip",
    "blacklisted": false
}
```

#### Rate limiting

tirreno uses a leaky bucket algorithm to prevent API abuse. Default limits:

| Setting | Default | Description |
|---------|---------|-------------|
| `LEAKY_BUCKET_RPS` | `5` | Maximum requests per second |
| `LEAKY_BUCKET_WINDOW` | `5` | Time window in seconds |

When rate limit is exceeded, the API returns HTTP `429 Too Many Requests`. Configure limits in `config/config.ini` or via environment variables.

**Best practices:**
- Implement exponential backoff when receiving 429 responses
- Queue events and send in batches during high traffic
- Set appropriate timeouts (3-5 seconds recommended)

#### Error responses

HTTP status codes returned by the API:

| HTTP Code | Cause |
|-----------|-------|
| `200` | Success (no response body) |
| `400` | Required field missing or invalid format |
| `401` | `Api-Key` header missing or API key not found |
| `429` | Rate limit exceeded |
| `500` | Internal server error |
| `503` | Database unavailable |

**Validation error response (400):**
```
Validation error: "Required field is missing or empty" for key "ipAddress"
```

**Note:** Successful requests (200) return no response body.

#### Logbook error types

The Logbook page in tirreno dashboard tracks all API requests with these status codes:

| Status | Description |
|--------|-------------|
| Success | Event recorded successfully |
| Validation warning | Event recorded with field corrections (e.g., truncated values) |
| Critical validation error | Event rejected due to missing required fields |
| Critical error | Server error, event not recorded |
| Rate limit exceeded | Request rejected due to rate limiting |

---

## Integration guide

This section covers integrating tirreno into your application.

### Why send events to tirreno?

tirreno analyzes user events to detect threats and calculate risk scores. Use cases:

- **Security monitoring:** Detect account takeover, brute force attacks, suspicious behavior
- **Threat hunting:** Search for indicators of compromise across user activity
- **Insider threats:** Spot unusual employee behavior, potential data exfiltration
- **Compliance:** Activity logs and field audit trail for GDPR, SOC 2, PCI-DSS
- **Forensics:** Investigate incidents with full session history
- **Risk scoring:** Calculate user trust scores from behavior patterns
- **Fraud prevention:** Block malicious users before damage occurs
- **IP enrichment:** Add geolocation, ISP, VPN/proxy detection to IP data

tirreno tracks per-user metrics: devices per day, IPs per day, sessions, events per session.

> **IP Enrichment API:** tirreno provides an API for IP geolocation and threat intelligence. The open-source Community Edition includes an optional IP enrichment pack (2,000 free API requests/month). For high-volume needs, contact tirreno for Enterprise options. See [tirreno.com](https://www.tirreno.com) for pricing details.

### Integration planning

> **Application Edition:** For internal applications we recommend to use existing integrations. Check the list of available integrations or contact tirreno at team@tirreno.com for further details.

#### What to track

| Fraud Vector | How tirreno Detects | Events |
|--------------|---------------------|--------|
| Stolen credentials | Multiple IPs, unusual locations | `account_login`, `account_login_fail` |
| Account sharing | Concurrent sessions, device changes | `page_view`, `account_login` |
| Fake accounts | Disposable emails, VPN/proxy | `account_registration` |
| Data scraping | High volume, bot signatures | `page_view`, `page_search` |
| Privilege abuse | Off-hours, sensitive operations | `account_edit`, `field_edit` |

#### Where to integrate

**Minimum:**
- Login/logout
- Failed login attempts
- Registration
- Password/email changes

**Recommended:**
- Authenticated page views
- Search queries
- Data modifications
- File downloads/exports
- Admin actions

#### Data you need

| Data | Source | Required |
|------|--------|----------|
| User ID | Auth system | Yes |
| IP address | Request headers | Yes |
| URL | Request path | Yes |
| Timestamp | Server time (UTC) | Yes |
| Email | User profile | Recommended |
| User agent | Headers | Recommended |

#### Technical notes

**Performance:**
- Use async/non-blocking HTTP calls
- Set 3-5 second timeouts
- Queue events during high traffic
- Don't block user actions on tirreno response

**Reliability:**
- Implement retry with exponential backoff
- Fail open (allow user action if tirreno unavailable)
- Log failed events locally for debugging

**Privacy:**
- Never send passwords or tokens
- Hash sensitive IDs if needed
- Document data collection in privacy policy
- Consider GDPR requirements for EU users

**Scalability:**
- One event per significant user action
- Batch events where appropriate
- Monitor tirreno logbook for errors

### Security considerations

When integrating tirreno, follow these security best practices:

1. **Install in private environment** Deploy tirreno in a private network with controlled access
2. **Protect your API key** Store in environment variables, never in code
3. **Use HTTPS** Always send events over encrypted connections
4. **Don't log sensitive data** Never include passwords, tokens, or PII in event payloads
5. **Fail open on errors** Don't block users if tirreno is temporarily unavailable
6. **Set timeouts** Use 3-5 second timeouts to prevent login delays
7. **Validate on your side** tirreno is for monitoring, not input validation
8. **Send timestamps in UTC** All `eventTime` values must be in UTC. Configure your tirreno instance timezone during initial setup or in Settings → Time zone. The dashboard will display events in your configured timezone, but all data sent via the API must use UTC

### Quick start

> **Important:** tirreno must be integrated on the backend only. Never send events from frontend JavaScript or mobile apps. Client-side code can be inspected, modified, or bypassed entirely — attackers could disable tracking, forge events, or extract your API key. Backend integration ensures event data cannot be tampered with and your API credentials remain secure.

The fastest way to integrate tirreno is using an official tracker library:

- [PHP Tracker](https://github.com/tirrenotechnologies/tirreno-php-tracker)
- [Python Tracker](https://github.com/tirrenotechnologies/tirreno-python-tracker)
- [Node.js Tracker](https://github.com/tirrenotechnologies/tirreno-nodejs-tracker)

**cURL (raw API):**
```bash
curl -X POST https://your-tirreno-instance.com/sensor/ \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "key=your-api-key" \
  -d "userName=user123" \
  -d "emailAddress=user@example.com" \
  -d "ipAddress=192.168.1.100" \
  -d "url=/login" \
  -d "eventTime=2024-12-08 14:30:00.000" \
  -d "eventType=page_view"
```

**PHP:**

Requirements: `cURL` PHP extension

Installation:
```
composer require tirreno/tirreno-tracker
```

Or manually via file download:
```php
require_once("TirrenoTracker.php");
```

Usage:
```php
<?php

// Load object
require_once("TirrenoTracker.php");

$tirrenoUrl = "https://example.tld/sensor/"; // Sensor URL
$trackingId = "XXX"; // Tracking ID

// Create object
$tracker = new TirrenoTracker($tirrenoUrl, $trackingId);

// Override defaults of required params
$tracker->setUserName("johndoe42")
        ->setIpAddress("1.1.1.1")
        ->setUrl("/login")
        ->setUserAgent("Mozilla/5.0 (X11; Linux x86_64)")
        ->setEventTypeAccountLogin();

// Set optional params
$tracker->setFirstName("John")
        ->setBrowserLanguage("fr-FR,fr;q=0.9")
        ->setHttpMethod("POST");

// Track event
$tracker->track();
```

**Python:**
```python
pip install tirreno_tracker

from tirreno_tracker import Tracker

tracker = Tracker('https://your-tirreno-instance.com', 'your-api-key')

# Track a login
event = tracker.create_event()

event.set_user_name(user_id) \
     .set_email_address(user_email) \
     .set_ip_address(ip_address) \
     .set_url(url_path) \
     .set_user_agent(user_agent) \
     .set_event_type_account_login()

tracker.track(event)
```

**Node.js:**
```javascript
npm install @tirreno/tirreno-tracker

const Tracker = require('@tirreno/tirreno-tracker');

const tracker = new Tracker('https://your-tirreno-instance.com', 'your-api-key');

// Track a registration
const event = tracker.createEvent();

event.setUserName(userId)
     .setEmailAddress(userEmail)
     .setIpAddress(ipAddress)
     .setUrl(urlPath)
     .setUserAgent(userAgent)
     .setEventTypeAccountRegistration();

await tracker.track(event);
```

### Event tracking best practices

#### Which events to track

**Essential events (always track):**

| Event | When to Track | Why It Matters |
|-------|---------------|----------------|
| `account_login` | Successful authentication | Detect account takeover |
| `account_login_fail` | Failed login attempts | Detect brute force attacks |
| `account_registration` | New account creation | Detect fake account creation |
| `account_password_change` | Password updates | Detect account compromise |
| `account_email_change` | Email changes | Detect account hijacking |

**Recommended events:**

| Event | When to Track | Why It Matters |
|-------|---------------|----------------|
| `page_view` | Key page visits | Behavioral analysis |
| `page_edit` | Content modifications | Detect malicious edits |
| `page_search` | Search queries | Detect reconnaissance |
| `page_error` | 4xx/5xx errors | Detect scanning/attacks |
| `field_edit` | Data modification | Field audit trail |

#### Data quality guidelines

1. **Consistent user identifiers:**
```php
// Good - use permanent ID
$tracker->setUserName($user->id);

// Bad - don't use changing values
$tracker->setUserName($user->email);  // Emails can change
```

2. **Accurate timestamps:**

The tracker libraries automatically set `eventTime` to the current UTC timestamp with milliseconds when you call `track()`. For manual timestamp handling, use the format `Y-m-d H:i:s.v`:

```php
// PHP - include milliseconds
$eventTime = date('Y-m-d H:i:s.v');  // 2024-01-15 10:30:45.123
```

3. **Real IP addresses:**
```php
// Good - handle proxies correctly
function getRealIp(): string {
    $headers = ['HTTP_CF_CONNECTING_IP', 'HTTP_X_FORWARDED_FOR', 'REMOTE_ADDR'];
    foreach ($headers as $header) {
        if (!empty($_SERVER[$header])) {
            $ips = explode(',', $_SERVER[$header]);
            return trim($ips[0]);
        }
    }
    return $_SERVER['REMOTE_ADDR'];
}

$tracker->setIpAddress(getRealIp());
```

4. **Complete user agent:**
```php
// Good - full user agent
$tracker->setUserAgent($_SERVER['HTTP_USER_AGENT']);

// Bad - truncated
$tracker->setUserAgent(substr($_SERVER['HTTP_USER_AGENT'], 0, 50));
```

### Send all logged-in user events

Track page views and actions from authenticated users.

**PHP:**
```php
session_start();

if (isset($_SESSION['user_id'])) {
    $tracker->setUserName((string) $_SESSION['user_id'])
            ->setEmailAddress($_SESSION['user_email'])
            ->setIpAddress($_SERVER['REMOTE_ADDR'])
            ->setUrl($_SERVER['REQUEST_URI'])
            ->setUserAgent($_SERVER['HTTP_USER_AGENT'] ?? '')
            ->setHttpMethod($_SERVER['REQUEST_METHOD'])
            ->setEventTypePageView();

    $tracker->track();
}
```

**Node.js:**
```javascript
if (userId) {
    const event = tracker.createEvent();

    event.setUserName(userId)
         .setEmailAddress(userEmail)
         .setIpAddress(ipAddress)
         .setUrl(urlPath)
         .setUserAgent(userAgent)
         .setHttpMethod(httpMethod)
         .setHttpCode(httpCode.toString())
         .setEventTypePageView();

    await tracker.track(event);
}
```

**Python:**
```python
if user_id:
    event = tracker.create_event()

    event.set_user_name(str(user_id)) \
         .set_email_address(user_email) \
         .set_ip_address(ip_address) \
         .set_url(url_path) \
         .set_user_agent(user_agent) \
         .set_http_method(http_method) \
         .set_http_code(str(http_code)) \
         .set_event_type_page_view()

    tracker.track(event)
```

### Protecting the registration

Protect your registration flow from fake accounts, bots, and abuse.

#### Track registration events

**PHP:**
```php
$userId = createUser($_POST['email'], $_POST['password'], $_POST['name']);

$tracker->setUserName((string) $userId)
        ->setEmailAddress($_POST['email'])
        ->setFullName($_POST['name'])
        ->setIpAddress($_SERVER['REMOTE_ADDR'])
        ->setUrl('/register')
        ->setUserAgent($_SERVER['HTTP_USER_AGENT'] ?? '')
        ->setUserCreated(date('Y-m-d H:i:s'))
        ->setEventTypeAccountRegistration();

$tracker->track();

header('Location: /dashboard');
```

**Python:**
```python
user_id = create_user(email, password, name)

event = tracker.create_event()

event.set_user_name(str(user_id)) \
     .set_email_address(email) \
     .set_full_name(name) \
     .set_ip_address(ip_address) \
     .set_url('/register') \
     .set_user_agent(user_agent) \
     .set_event_type_account_registration()

tracker.track(event)
```

**Node.js:**
```javascript
const userId = await createUser(email, password, name);

const event = tracker.createEvent();

event.setUserName(userId.toString())
     .setEmailAddress(email)
     .setFullName(name)
     .setIpAddress(ipAddress)
     .setUrl('/register')
     .setUserAgent(userAgent)
     .setEventTypeAccountRegistration();

await tracker.track(event);
```

### Protecting the login

Secure your login flow against brute force attacks and credential stuffing.

#### Track login events

**PHP:**
```php
$email = $_POST['email'];
$password = $_POST['password'];

$user = authenticateUser($email, $password);

if (!$user) {
    // Track failed login
    $tracker->setUserName($email)
            ->setEmailAddress($email)
            ->setIpAddress($_SERVER['REMOTE_ADDR'])
            ->setUrl('/login')
            ->setUserAgent($_SERVER['HTTP_USER_AGENT'] ?? '')
            ->setEventTypeAccountLoginFail();

    $tracker->track();

    die('Invalid credentials');
}

// Track successful login
$tracker->setUserName((string) $user['id'])
        ->setEmailAddress($user['email'])
        ->setIpAddress($_SERVER['REMOTE_ADDR'])
        ->setUrl('/login')
        ->setUserAgent($_SERVER['HTTP_USER_AGENT'] ?? '')
        ->setEventTypeAccountLogin();

$tracker->track();

session_start();
$_SESSION['user_id'] = $user['id'];
header('Location: /dashboard');
```

**Python:**
```python
user = authenticate_user(email, password)

if not user:
    # Track failed login
    event = tracker.create_event()

    event.set_user_name(email) \
         .set_email_address(email) \
         .set_ip_address(ip_address) \
         .set_url('/login') \
         .set_user_agent(user_agent) \
         .set_event_type_account_login_fail()

    tracker.track(event)
    # Return error
else:
    # Track successful login
    event = tracker.create_event()

    event.set_user_name(str(user['id'])) \
         .set_email_address(user['email']) \
         .set_ip_address(ip_address) \
         .set_url('/login') \
         .set_user_agent(user_agent) \
         .set_event_type_account_login()

    tracker.track(event)
```

**Node.js:**
```javascript
const user = await authenticateUser(email, password);

if (!user) {
    // Track failed login
    const event = tracker.createEvent();

    event.setUserName(email)
         .setEmailAddress(email)
         .setIpAddress(ipAddress)
         .setUrl('/login')
         .setUserAgent(userAgent)
         .setEventTypeAccountLoginFail();

    await tracker.track(event);
    // Return error
} else {
    // Track successful login
    const event = tracker.createEvent();

    event.setUserName(user.id.toString())
         .setEmailAddress(user.email)
         .setIpAddress(ipAddress)
         .setUrl('/login')
         .setUserAgent(userAgent)
         .setEventTypeAccountLogin();

    await tracker.track(event);
}
```

#### Block blacklisted users

Check the blacklist API before allowing login:

```php
$email = $_POST['email'];
$password = $_POST['password'];

// Block known attackers before authentication
if ($blacklistService->isBlacklisted($email)) {
    die('Invalid credentials');
}

$user = authenticateUser($email, $password);

if (!$user) {
    $tracker->setUserName($email)
            ->setEmailAddress($email)
            ->setIpAddress($_SERVER['REMOTE_ADDR'])
            ->setUrl('/login')
            ->setUserAgent($_SERVER['HTTP_USER_AGENT'] ?? '')
            ->setEventTypeAccountLoginFail();

    $tracker->track();
    die('Invalid credentials');
}

// Also check authenticated user
if ($blacklistService->isBlacklisted((string) $user['id'])) {
    die('Invalid credentials');
}

// Track successful login
$tracker->setUserName((string) $user['id'])
        ->setEmailAddress($user['email'])
        ->setIpAddress($_SERVER['REMOTE_ADDR'])
        ->setUrl('/login')
        ->setUserAgent($_SERVER['HTTP_USER_AGENT'] ?? '')
        ->setEventTypeAccountLogin();

$tracker->track();

session_start();
$_SESSION['user_id'] = $user['id'];
header('Location: /dashboard');
```

### Auto-ban abusive IPs

Use tirreno's IP analysis combined with the blacklist API for automatic protection.

#### Configure threshold settings

Before implementing auto-ban, configure and test the threshold settings in tirreno:

1. Go to **Rules** page in tirreno dashboard
2. Set **Manual review** threshold (e.g., 33) users below this score appear in review queue
3. Set **Auto-blacklisting** threshold (e.g., 20) users below this score are automatically blacklisted
4. Click **Update** to save settings

#### Middleware for IP-based blocking

**PHP:**
```php
$ip = $_SERVER['REMOTE_ADDR'];

if ($blacklistService->isBlacklisted($ip)) {
    http_response_code(403);
    die('Access denied');
}
```

**Python:**
```python
if blacklist_service.is_blacklisted(ip_address):
    # Return 403 Access denied
    pass
```

**Node.js:**
```javascript
if (await blacklistService.isBlacklisted(ipAddress)) {
    // Return 403 Access denied
}
```

### Field audit trail

Track changes to important user fields for compliance, security, and regulatory requirements. The `fieldHistory` parameter allows you to send detailed change records.

#### Field history format

Each field change object has these properties:

| Property | Required | Type | Description |
|----------|----------|------|-------------|
| `field_id` | Yes | int/string | Unique identifier for the field |
| `new_value` | Yes | string | New value |
| `field_name` | No | string | Human-readable field name |
| `old_value` | No | string | Previous value |
| `parent_id` | No | string | Parent record ID (for nested data) |
| `parent_name` | No | string | Parent record name |

**Note:** Missing required fields default to `"unknown"`. All values are converted to strings.

**PHP:**
```php
function trackFieldChanges($userId, $userEmail, $oldData, $newData, $tracker) {
    $trackableFields = [
        'city' => 'User city',
        'phone' => 'Phone number',
        'address' => 'Address',
        'company' => 'Company name',
    ];

    $changes = [];
    foreach ($trackableFields as $field => $fieldName) {
        $oldValue = $oldData[$field] ?? '';
        $newValue = $newData[$field] ?? '';

        if ($oldValue !== $newValue) {
            $changes[] = [
                'field_id' => crc32($field),
                'field_name' => $fieldName,
                'old_value' => (string) $oldValue,
                'new_value' => (string) $newValue,
                'parent_id' => '',
                'parent_name' => '',
            ];
        }
    }

    if (!empty($changes)) {
        $tracker->setUserName((string) $userId)
                ->setEmailAddress($userEmail)
                ->setIpAddress($_SERVER['REMOTE_ADDR'])
                ->setUrl($_SERVER['REQUEST_URI'])
                ->setUserAgent($_SERVER['HTTP_USER_AGENT'] ?? '')
                ->setEventTypeFieldEdit()
                ->setFieldHistory($changes);

        $tracker->track();
    }
}

// Usage
$oldData = getUserById($userId);
updateUser($userId, $_POST);
trackFieldChanges($userId, $userEmail, $oldData, $_POST, $tracker);
```

**Python:**
```python
def track_field_changes(user_id, user_email, old_data, new_data, tracker):
    trackable_fields = {
        'city': 'User city',
        'phone': 'Phone number',
        'address': 'Address',
        'company': 'Company name',
    }

    changes = []
    for field, field_name in trackable_fields.items():
        old_value = old_data.get(field, '')
        new_value = new_data.get(field, '')

        if old_value != new_value:
            changes.append({
                'field_id': hash(field) & 0xffffffff,
                'field_name': field_name,
                'old_value': str(old_value),
                'new_value': str(new_value),
                'parent_id': '',
                'parent_name': '',
            })

    if changes:
        event = tracker.create_event()

        event.set_user_name(str(user_id)) \
             .set_email_address(user_email) \
             .set_ip_address(ip_address) \
             .set_url(url_path) \
             .set_user_agent(user_agent) \
             .set_event_type_field_edit() \
             .set_field_history(changes)

        tracker.track(event)

# Usage
old_data = get_user_by_id(user_id)
update_user(user_id, new_data)
track_field_changes(user_id, user_email, old_data, new_data, tracker)
```

**Node.js:**
```javascript
async function trackFieldChanges(userId, userEmail, oldData, newData, tracker) {
    const trackableFields = {
        city: 'User city',
        phone: 'Phone number',
        address: 'Address',
        company: 'Company name',
    };

    const changes = [];
    for (const [field, fieldName] of Object.entries(trackableFields)) {
        const oldValue = oldData[field] ?? '';
        const newValue = newData[field] ?? '';

        if (oldValue !== newValue) {
            changes.push({
                field_id: hashCode(field),
                field_name: fieldName,
                old_value: String(oldValue),
                new_value: String(newValue),
                parent_id: '',
                parent_name: ''
            });
        }
    }

    if (changes.length > 0) {
        const event = tracker.createEvent();

        event.setUserName(userId.toString())
             .setEmailAddress(userEmail)
             .setIpAddress(ipAddress)
             .setUrl(urlPath)
             .setUserAgent(userAgent)
             .setEventTypeFieldEdit()
             .setFieldHistory(changes);

        await tracker.track(event);
    }
}

// Usage
const oldData = await getUserById(userId);
await updateUser(userId, newData);
await trackFieldChanges(userId, userEmail, oldData, newData, tracker);
```

**Tracking nested/related data:**
```php
// For related records (e.g., user addresses)
$changes = [];

foreach ($updatedAddresses as $address) {
    $original = $originalAddresses->find($address->id);

    foreach (['street', 'city', 'zip'] as $field) {
        if ($original->$field !== $address->$field) {
            $changes[] = [
                'field_id' => crc32($field),
                'field_name' => ucfirst($field),
                'old_value' => $original->$field,
                'new_value' => $address->$field,
                'parent_id' => (string) $address->id,      // Link to address record
                'parent_name' => "Address #{$address->id}", // Human-readable reference
            ];
        }
    }
}
```

### Testing your integration

#### Manual testing checklist

1. **Verify API connectivity:**
```bash
curl -X POST https://your-tirreno.com/sensor/ \
  -H "Api-Key: your-api-key" \
  -d "userName=test-user-123" \
  -d "emailAddress=test@example.com" \
  -d "ipAddress=203.0.113.50" \
  -d "url=/test" \
  -d "userAgent=Mozilla/5.0 Test" \
  -d "eventTime=2024-12-08 01:01:00.000" \
  -d "eventType=page_view"
```

2. **Check the Logbook:**
   - Log in to your tirreno instance
   - Navigate to **Logbook** in the left menu
   - View real-time API requests with Source IP, Timestamp, Endpoint, and Status
   - Filter by endpoint, IP, or error messages using the search box
   - The chart shows request volume over time to identify traffic patterns

3. **Check the Users page:**
   - Navigate to **Users** to see the tracked user
   - Verify the user details and events are correctly recorded

4. **Verify event types:**
   - Test each event type you plan to use
   - Confirm events appear in the correct user timeline

---

## Risk rules & customization

tirreno is designed to be customized for your specific security needs. No CLA or pull request is required for local modifications.

The three main customization points are:

1. **Custom rules** Create detection rules based on user behavior
2. **Suspicious pattern lists** Adjust URL, user agent, and email pattern detection
3. **UI constants** Configure display thresholds and visual settings

### Built-in rules

tirreno includes standard detection rules organized by category:

#### Account takeover (A01-A08)
| Rule | Name | Description |
|------|------|-------------|
| A01 | Multiple login fail | User failed to login multiple times in a short term |
| A02 | Login failed on new device | User failed to login with new device |
| A03 | New device and new country | User logged in with new device from new location |
| A04 | New device and new subnet | User logged in with new device from new subnet |
| A05 | Password change on new device | User changed their password on new device |
| A06 | Password change in new country | User changed their password in new country |
| A07 | Password change in new subnet | User changed their password in new subnet |
| A08 | Browser language changed | User accessed the account with new browser language |

#### Behaviour (B01-B26)
| Rule | Name | Description |
|------|------|-------------|
| B01 | Multiple countries | IP addresses are located in diverse countries |
| B02 | User has changed a password | The user has changed their password |
| B03 | User has changed an email | The user has changed their email |
| B04 | Multiple 5xx errors | User made multiple requests which evoked internal server error |
| B05 | Multiple 4xx errors | User made multiple requests which cannot be fulfilled |
| B06 | Potentially vulnerable URL | User made a request to suspicious URL |
| B07 | User's full name contains digits | Full name contains digits |
| B08 | Dormant account (30 days) | Account has been inactive for 30 days |
| B09 | Dormant account (90 days) | Account has been inactive for 90 days |
| B10 | Dormant account (1 year) | Account has been inactive for a year |
| B11 | New account (1 day) | Account has been created today |
| B12 | New account (1 week) | Account has been created this week |
| B13 | New account (1 month) | Account has been created this month |
| B14 | Aged account (>30 days) | Account has been created over 30 days ago |
| B15 | Aged account (>90 days) | Account has been created over 90 days ago |
| B16 | Aged account (>180 days) | Account has been created over 180 days ago |
| B17 | Single country | IP addresses are located in a single country |
| B18 | HEAD request | HTTP request HEAD method is often used by bots |
| B19 | Night time requests | User was active from midnight till 5 a.m. |
| B20 | Multiple countries in one session | User's country changed in less than 30 minutes |
| B21 | Multiple devices in one session | User's device changed in less than 30 minutes |
| B22 | Multiple IP addresses in one session | User's IP changed in less than 30 minutes |
| B23 | User's full name contains space or hyphen | Full name contains space or hyphen |
| B24 | Empty referer | User made a request without a referer |
| B25 | Unauthorized request | User made a successful request without authorization |
| B26 | Single event sessions | User had sessions with only one event |

#### Country (C01-C16)
| Rule | Name | Description |
|------|------|-------------|
| C01 | Nigeria IP address | IP address located in Nigeria |
| C02 | India IP address | IP address located in India |
| C03 | China IP address | IP address located in China |
| C04 | Brazil IP address | IP address located in Brazil |
| C05 | Pakistan IP address | IP address located in Pakistan |
| C06 | Indonesia IP address | IP address located in Indonesia |
| C07 | Venezuela IP address | IP address located in Venezuela |
| C08 | South Africa IP address | IP address located in South Africa |
| C09 | Philippines IP address | IP address located in Philippines |
| C10 | Romania IP address | IP address located in Romania |
| C11 | Russia IP address | IP address located in Russia |
| C12 | European IP address | IP address located in European Union |
| C13 | North America IP address | IP address located in Canada or USA |
| C14 | Australia IP address | IP address located in Australia |
| C15 | UAE IP address | IP address located in United Arab Emirates |
| C16 | Japan IP address | IP address located in Japan |

#### Device (D01-D10)
| Rule | Name | Description |
|------|------|-------------|
| D01 | Device is unknown | User has manipulated device information |
| D02 | Device is Linux | Linux OS, increased risk of crawler bot |
| D03 | Device is bot | User agent identified as a bot |
| D04 | Rare browser device | User operates device with uncommon browser |
| D05 | Rare OS device | User operates device with uncommon OS |
| D06 | Multiple devices per user | User accesses account using multiple devices |
| D07 | Several desktop devices | User accesses account using different OS desktop devices |
| D08 | Two or more phone devices | User accesses account using numerous phone devices |
| D09 | Old browser | User accesses account using an old browser version |
| D10 | Potentially vulnerable User-Agent | User made a request with suspicious User-Agent |

#### Email (E01-E30)
| Rule | Name | Description |
|------|------|-------------|
| E01 | Invalid email format | Invalid email format |
| E02 | New domain and no breaches | Email belongs to recently created domain with no breach history |
| E03 | Suspicious words in email | Email contains auto-generated mailbox patterns |
| E04 | Numeric email name | Email username consists entirely of numbers |
| E05 | Special characters in email | Email has unusually high number of special characters |
| E06 | Consecutive digits in email | Email includes at least two consecutive digits |
| E07 | Long email username | Email username exceeds average length |
| E08 | Long domain name | Email domain name is too long |
| E09 | Free email provider | Email belongs to free provider |
| E10 | The website is unavailable | Domain's website seems to be inactive |
| E11 | Disposable email | Disposable email addresses are temporary |
| E12 | Free email and no breaches | Email belongs to free provider with no breach history |
| E13 | New domain | Domain name was registered recently |
| E14 | No MX record | Email's domain has no MX record |
| E15 | No breaches for email | Email was not involved in any data breaches |
| E16 | Domain appears in spam lists | Email appears in spam lists |
| E17 | Free email and spam | Email appears in spam lists and is from free provider |
| E19 | Multiple emails changed | User has changed their email |
| E20 | Established domain (> 3 year old) | Email belongs to domain registered at least 3 years ago |
| E21 | No vowels in email | Email username does not contain any vowels |
| E22 | No consonants in email | Email username does not contain any consonants |
| E23 | Educational domain (.edu) | Email belongs to educational domain |
| E24 | Government domain (.gov) | Email belongs to government domain |
| E25 | Military domain (.mil) | Email belongs to military domain |
| E26 | iCloud mailbox | Email belongs to Apple domains (icloud.com, me.com, mac.com) |
| E27 | Email breaches | Email appears in data breaches |
| E28 | No digits in email | Email address does not include digits |
| E29 | Old breach (>3 years) | Earliest data breach appeared more than 3 years ago |
| E30 | Domain with average rank | Email domain has Tranco rank between 100,000 and 4,000,000 |

**Note:** E18 is reserved for future use.

#### IP (I01-I12)
| Rule | Name | Description |
|------|------|-------------|
| I01 | IP belongs to TOR | IP assigned to The Onion Router network |
| I02 | IP hosting domain | Higher risk of crawler bot |
| I03 | IP appears in spam list | User may have exhibited unwanted activity before |
| I04 | Shared IP | Multiple users detected on same IP address |
| I05 | IP belongs to commercial VPN | User tries to hide real location |
| I06 | IP belongs to datacenter | User is utilizing an ISP datacenter |
| I07 | IP belongs to Apple Relay | IP belongs to iCloud Private Relay |
| I08 | IP belongs to Starlink | IP belongs to SpaceX satellite network |
| I09 | Numerous IPs | User accesses account with numerous IP addresses |
| I10 | Only residential IPs | User uses only residential IP addresses |
| I11 | Single network | IP addresses belong to one network |
| I12 | IP belongs to LAN | IP address belongs to local access network |

#### Phone (P01-P04)
| Rule | Name | Description |
|------|------|-------------|
| P01 | Invalid phone format | User provided incorrect phone number |
| P02 | Phone country mismatch | Phone number country is not among user's login countries |
| P03 | Shared phone number | User provided a phone number shared with another user |
| P04 | Valid phone | User provided correct phone number |

#### Reuse/blacklist (R01-R03)
| Rule | Name | Description |
|------|------|-------------|
| R01 | IP in blacklist | This IP address appears in the blacklist |
| R02 | Email in blacklist | This email address appears in the blacklist |
| R03 | Phone in blacklist | This phone number appears in the blacklist |

### Developing custom rules

Custom rules are placed in `assets/rules/custom/` with filenames `X01.php`, `X02.php`, etc.

Each rule must:
* Use namespace `Tirreno\Rules\Custom`
* Extend `\Tirreno\Assets\Rule`
* Define constants: `NAME`, `DESCRIPTION`, `ATTRIBUTES`
* Implement `defineCondition()` method

#### Testing rules

1. **Refresh rules:** After creating or modifying rules, go to the Rules page and click **Refresh** at the bottom of the page to apply your changes
2. **Test a rule:** Select a rule and click the **Play** button (▷) to see how many users are triggered by the rule
3. **Match rate:** The percentage shown indicates how many users match the rule (e.g., "22%" means 22% of users trigger this rule)

### Ruler operators reference

The rules engine uses ruler/ruler for condition evaluation. Available operators in `defineCondition()`:

| Operator | Description | Example |
|----------|-------------|---------|
| `equalTo` | Exact match | `$this->rb['ea_total_country']->equalTo(1)` |
| `notEqualTo` | Not equal | `$this->rb['eip_tor']->notEqualTo(true)` |
| `greaterThan` | Greater than | `$this->rb['ea_total_ip']->greaterThan(9)` |
| `greaterThanOrEqualTo` | Greater or equal | `$this->rb['ea_days_since_last_visit']->greaterThanOrEqualTo(30)` |
| `lessThan` | Less than | `$this->rb['ea_days_since_account_creation']->lessThan(7)` |
| `lessThanOrEqualTo` | Less or equal | `$this->rb['eup_device_count']->lessThanOrEqualTo(1)` |
| `stringContains` | Substring match | `$this->rb['le_email']->stringContains('test')` |
| `stringContainsInsensitive` | Case-insensitive substring | `$this->rb['le_domain_part']->stringContainsInsensitive('mail')` |
| `startsWith` | Prefix match | `$this->rb['event_url_string']->startsWith('/api/')` |
| `endsWith` | Suffix match | `$this->rb['le_email']->endsWith('.edu')` |
| `sameAs` | Variable comparison | `$this->rb['lp_country_code']->sameAs($this->rb['eip_country_id'])` |

**Logical operators:**

```php
// AND - all conditions must be true
$this->rb->logicalAnd(
    $this->rb['eip_tor']->equalTo(true),
    $this->rb['ea_days_since_account_creation']->lessThan(7)
);

// OR - at least one condition must be true
$this->rb->logicalOr(
    $this->rb['eip_vpn']->equalTo(true),
    $this->rb['eip_tor']->equalTo(true)
);

// NOT - negate a condition
$this->rb->logicalNot(
    $this->rb['le_has_no_data_breaches']->equalTo(true)
);
```

### Rule context attributes

When writing custom rules, the following attributes are available in the `defineCondition()` method. Access them via `$this->rb['attribute_name']`.

#### Event attributes (event_)

From Event context:
| Attribute | Type | Description |
|-----------|------|-------------|
| `event_ip` | array | IP IDs per event |
| `event_url_string` | array | URLs per event |
| `event_empty_referer` | array | Empty referer status per event |
| `event_device` | array | Device IDs per event |
| `event_type` | array | Event types |
| `event_http_code` | array | HTTP response codes |
| `event_http_method` | array | HTTP methods |
| `event_device_created` | array | Device creation timestamps |
| `event_device_lastseen` | array | Device last seen timestamps |

Derived event attributes:
| Attribute | Type | Description |
|-----------|------|-------------|
| `event_email_changed` | bool | User changed email in recent events |
| `event_password_changed` | bool | User changed password in recent events |
| `event_http_method_head` | bool | HEAD request detected |
| `event_empty_referer` | bool | Request had empty referer |
| `event_multiple_5xx_http` | int | Count of 5xx server errors |
| `event_multiple_4xx_http` | int | Count of 4xx client errors |
| `event_2xx_http` | bool | Successful requests exist |
| `event_vulnerable_url` | bool | URL matches suspicious patterns |

#### Account attributes (ea_)

Raw account data from User context:
| Attribute | Type | Description |
|-----------|------|-------------|
| `ea_userid` | string | User identifier |
| `ea_created` | string | Account creation timestamp |
| `ea_lastseen` | string | Last activity timestamp |
| `ea_total_visit` | int | Total visits |
| `ea_total_country` | int | Total countries |
| `ea_total_ip` | int | Total IP addresses |
| `ea_total_device` | int | Total devices |
| `ea_firstname` | string | First name |
| `ea_lastname` | string | Last name |

Derived account attributes:
| Attribute | Type | Description |
|-----------|------|-------------|
| `ea_days_since_account_creation` | int | Days since account was created (-1 if unknown) |
| `ea_days_since_last_visit` | int | Days since user's last activity (-1 if unknown) |
| `ea_fullname_has_numbers` | bool | Full name contains digits |
| `ea_fullname_has_spaces_hyphens` | bool | Full name contains spaces or hyphens |

#### IP attributes (eip_)

From Ip context:
| Attribute | Type | Description |
|-----------|------|-------------|
| `eip_cidr_count` | array | Count of IPs per CIDR |
| `eip_country_count` | array | Count of IPs per country |
| `eip_country_id` | array | Country IDs |
| `eip_data_center` | bool | IP belongs to datacenter |
| `eip_tor` | bool | IP belongs to TOR network |
| `eip_vpn` | bool | IP belongs to commercial VPN |
| `eip_starlink` | bool | IP belongs to Starlink |
| `eip_blocklist` | bool | IP appears in spam/blocklist |
| `eip_has_fraud` | bool | Fraud detected for IP |
| `eip_lan` | bool | IP belongs to LAN |
| `eip_shared` | int | Number of users sharing this IP |
| `eip_domains_count_len` | int | Number of domains on IP |
| `eip_unique_cidrs` | int | Number of unique network ranges |
| `eip_only_residential` | bool | All IPs are residential (derived) |

#### Device attributes (eup_)

From Device context:
| Attribute | Type | Description |
|-----------|------|-------------|
| `eup_device` | array | Device types (desktop, smartphone, tablet, etc.) |
| `eup_browser_name` | array | Browser names |
| `eup_browser_version` | array | Browser versions |
| `eup_os_name` | array | Operating system names |
| `eup_lang` | array | Browser languages |
| `eup_ua` | array | Raw user agent strings |

Derived device attributes:
| Attribute | Type | Description |
|-----------|------|-------------|
| `eup_device_count` | int | Number of devices used |
| `eup_has_rare_browser` | bool | User has uncommon browser |
| `eup_has_rare_os` | bool | User has uncommon OS |
| `eup_vulnerable_ua` | bool | User-Agent matches suspicious patterns |

#### Session attributes (event_session_)

From Session context:
| Attribute | Type | Description |
|-----------|------|-------------|
| `event_session_single_event` | bool | Session had only one event |
| `event_session_multiple_country` | bool | Country changed within 30 min |
| `event_session_multiple_ip` | bool | IP changed within 30 min |
| `event_session_multiple_device` | bool | Device changed within 30 min |
| `event_session_night_time` | bool | Activity between midnight and 5 AM |

#### Email attributes (Platform Edition only)

Last Email Attributes (le_):
| Attribute | Type | Description |
|-----------|------|-------------|
| `le_email` | string | Email address |
| `le_local_part` | string | Email username (before @) |
| `le_domain_part` | string | Email domain (after @) |
| `le_blockemails` | bool | Email is in blocklist |
| `le_data_breach` | bool | Known data breaches |
| `le_checked` | bool | Email has been verified |
| `le_fraud_detected` | bool | Fraud detected for email |
| `le_alert_list` | bool | Email on alert list |

Derived last email attributes:
| Attribute | Type | Description |
|-----------|------|-------------|
| `le_exists` | bool | Email address exists |
| `le_is_invalid` | bool | Email format is invalid |
| `le_has_suspicious_str` | bool | Email contains suspicious patterns |
| `le_has_numeric_only_local_part` | bool | Email username is all numbers |
| `le_email_has_consec_s_chars` | bool | Email has consecutive special characters |
| `le_email_has_consec_nums` | bool | Email has consecutive digits |
| `le_email_has_no_digits` | bool | Email has no digits |
| `le_email_has_vowels` | bool | Email username contains vowels |
| `le_email_has_consonants` | bool | Email username contains consonants |
| `le_with_long_local_part_length` | bool | Email username exceeds max length |
| `le_with_long_domain_length` | bool | Email domain exceeds max length |
| `le_email_in_blockemails` | bool | Email is in blocklist |
| `le_has_no_data_breaches` | bool | No known data breaches |
| `le_appears_on_alert_list` | bool | Email on alert list |
| `le_local_part_len` | int | Length of email username |

Email Attributes (ee_):
| Attribute | Type | Description |
|-----------|------|-------------|
| `ee_email` | array | All email addresses for user |
| `ee_earliest_breach` | array | Earliest breach dates per email |
| `ee_days_since_first_breach` | int | Days since earliest known breach (-1 if none) |

#### Domain attributes (Platform Edition only)

Last Domain Attributes (ld_):
| Attribute | Type | Description |
|-----------|------|-------------|
| `ld_disposable_domains` | bool | Domain is disposable email provider |
| `ld_free_email_provider` | bool | Domain is free email provider |
| `ld_blockdomains` | bool | Domain is in blocklist |
| `ld_mx_record` | bool | Domain has MX record |
| `ld_disabled` | bool | Domain website is disabled |
| `ld_creation_date` | string | Domain creation date |
| `ld_tranco_rank` | int | Tranco ranking (-1 if not ranked) |

Derived last domain attributes:
| Attribute | Type | Description |
|-----------|------|-------------|
| `ld_is_disposable` | bool | Domain is disposable email provider |
| `ld_days_since_domain_creation` | int | Days since domain registration |
| `ld_domain_free_email_provider` | bool | Domain is free email provider |
| `ld_from_blockdomains` | bool | Domain is in blocklist |
| `ld_domain_without_mx_record` | bool | Domain has no MX record |
| `ld_website_is_disabled` | bool | Domain website is disabled |

#### Phone attributes (Platform Edition only)

From Phone context (ep_):
| Attribute | Type | Description |
|-----------|------|-------------|
| `ep_phone_number` | array | Phone numbers |
| `ep_shared` | array | Shared status per phone |
| `ep_type` | array | Phone types |

Last phone from User context (lp_):
| Attribute | Type | Description |
|-----------|------|-------------|
| `lp_phone_number` | string | Last phone number |
| `lp_country_code` | string | Phone country code |
| `lp_invalid` | bool | Phone number is invalid |
| `lp_fraud_detected` | bool | Fraud detected for phone |
| `lp_alert_list` | bool | Phone on alert list |

Derived phone attributes:
| Attribute | Type | Description |
|-----------|------|-------------|
| `lp_invalid_phone` | bool | Phone number is invalid |
| `ep_shared_phone` | bool | Phone is shared with other users |

### Suspicious pattern lists

tirreno maintains lists of suspicious patterns in `assets/lists/`:

| File | Purpose |
|------|---------|
| `url.php` | URL attack patterns (SQL injection, path traversal, etc.) |
| `user-agent.php` | Suspicious user agent strings |
| `email.php` | Suspicious email patterns |
| `file-extensions.php` | File extension categories |

Each file returns a PHP array:

```php
<?php
return [
    '.env',
    '.git',
    '/wp-admin',
    'phpmyadmin',
    '<script>',
    // ...
];
```

To add patterns:
1. Open the appropriate file in `assets/lists/`
2. Add your pattern string to the array
3. Patterns are case-sensitive substring matches

**Example patterns by type:**

| List | Example Patterns |
|------|------------------|
| `url.php` | `'.env'`, `'../'`, `'/wp-admin'`, `'phpmyadmin'`, `'<script>'` |
| `user-agent.php` | Bot signatures, scanner identifiers, SQL injection attempts |
| `email.php` | `'spam'`, `'test'`, `'dummy'`, `'123'`, `'000'` |

### UI constants

Configure display thresholds, colors, and string lengths in `ui/js/parts/utils/Constants.js`.

**Trust score thresholds:**
```javascript
const USER_LOW_TRUST_SCORE_INF    = 0;
const USER_LOW_TRUST_SCORE_SUP    = 33;
const USER_MEDIUM_TRUST_SCORE_INF = 33;
const USER_MEDIUM_TRUST_SCORE_SUP = 67;
const USER_HIGH_TRUST_SCORE_INF   = 67;
```

**Critical value thresholds:**
```javascript
const USER_IPS_CRITICAL_VALUE       = 9;       // Flag users with 9+ IPs
const USER_EVENTS_CRITICAL_VALUE    = Infinity; // No limit for events
const USER_DEVICES_CRITICAL_VALUE   = 4;       // Flag users with 4+ devices
const USER_COUNTRIES_CRITICAL_VALUE = 3;       // Flag users from 3+ countries
```

**Color scheme:**
```javascript
const COLOR_RED    = '#FB6E88';  // High risk
const COLOR_GREEN  = '#01EE99';  // Low risk
const COLOR_YELLOW = '#F5B944';  // Medium risk
const COLOR_PURPLE = '#BE95EB';  // Special status

// Light variants (for backgrounds)
const COLOR_LIGHT_GREEN  = 'rgba(64,220,97,0.03)';
const COLOR_LIGHT_YELLOW = 'rgba(225,224,137,0.03)';
const COLOR_LIGHT_RED    = 'rgba(255,51,102,0.03)';
const COLOR_LIGHT_PURPLE = 'rgba(190,149,235,0.03)';
```

**String length limits:**
```javascript
const MAX_STRING_LENGTH_IN_TABLE = 18;
const MAX_STRING_LENGTH_URL = 32;
const MAX_TOOLTIP_LENGTH = 121;
const MAX_STRING_LENGTH_FOR_EMAIL = 14;
const MAX_STRING_LENGTH_FOR_PHONE = 17;
```

---

## Contributing

This section is for developers who want to contribute code to the tirreno project. If you only want to customize tirreno for your own use (custom rules, pattern lists, UI constants), see the [Risk rules & customization](#risk-rules--customization) section above.

> **Notice:** Submissions using generative AI will be rejected. Submissions from AI chatbots will result in the account being banned.

### Source code

The source code is maintained at: https://github.com/tirrenotechnologies/tirreno

### Contributor license agreement (CLA)

Before your contributions can be accepted, you must sign the tirreno Contributor License Agreement (CLA). All contributed code is dual-licensed: AGPL-3.0 for open source use and a separate enterprise license for commercial use. Contact team@tirreno.com for the CLA document.

### Git workflow

1. Fork the repository on GitHub
2. Clone your fork: `git clone https://github.com/YOUR_USERNAME/tirreno.git`
3. Create a branch: `git checkout -b feature/your-feature`
4. Make changes following coding standards
5. Commit, push, and open a Pull Request

### Local development setup

#### Prerequisites

- PHP 8.0 to 8.3 with extensions: PDO_PGSQL, cURL, mbstring
- PostgreSQL 12 or greater
- Apache with mod_rewrite
- Composer
- Git

#### Local setup

```bash
# 1. Fork and clone
git clone https://github.com/YOUR_USERNAME/tirreno.git
cd tirreno

# 2. Install dependencies
composer install

# 3. Create PostgreSQL database
createdb tirreno_dev

# 4. Configure database
# Edit config/ files with your database credentials

# 5. Run web installer
# Point Apache to project root, visit: http://localhost/install/

# 6. Delete install directory (important!)
rm -rf install/

# 7. Setup cron job
crontab -e
# Add: */10 * * * * /usr/bin/php /path/to/tirreno/index.php /cron

# 8. Create admin account at /signup/
```

#### Docker setup

`docker-compose.yml`:

```yaml
services:
  tirreno-app:
    image: tirreno/tirreno:latest
    ports:
      - "8585:80"
    volumes:
      - tirreno:/var/www/html
    networks:
      - tirreno-network
    depends_on:
      - tirreno-db

  tirreno-db:
    image: postgres:15
    environment:
      POSTGRES_DB: tirreno
      POSTGRES_USER: tirreno
      POSTGRES_PASSWORD: secret
    volumes:
      - ./db:/var/lib/postgresql/data
    networks:
      - tirreno-network

networks:
  tirreno-network:

volumes:
  tirreno:
```

Run: `docker compose up -d`

Access http://localhost:8585/install/, use `tirreno-db` as host, `tirreno`/`secret` for credentials.

**Manual Docker:**

```bash
# Create network
docker network create tirreno-network

# Start PostgreSQL
docker run -d \
  --name tirreno-db \
  --network tirreno-network \
  -e POSTGRES_DB=tirreno \
  -e POSTGRES_USER=tirreno \
  -e POSTGRES_PASSWORD=secret \
  -v ./db:/var/lib/postgresql/data \
  postgres:15

# Start tirreno
docker run -d \
  --name tirreno-app \
  --network tirreno-network \
  -p 8585:80 \
  -v tirreno:/var/www/html \
  tirreno/tirreno:latest
```

### Code quality tools

tirreno uses the following tools for code quality:
* **PHP_CodeSniffer** (`phpcs.xml`) for PHP style enforcement
* **PHPStan** for static analysis
* **ESLint** (`eslint.config.js`) for JavaScript

```bash
# PHP CodeSniffer - check style
./vendor/bin/phpcs --standard=phpcs.xml app/

# PHP CodeSniffer - auto-fix
./vendor/bin/phpcbf --standard=phpcs.xml app/

# PHPStan - static analysis
./vendor/bin/phpstan analyse

# ESLint - JavaScript
npx eslint ui/js/
npx eslint ui/js/ --fix
```

### PHP coding standards

#### Class structure

Follow the tirreno Model pattern:

```php
<?php
declare(strict_types=1);
namespace Tirreno\Models;

class Device extends \Tirreno\Models\BaseSql {
    protected $DB_TABLE_NAME = 'event_device';

    public function getFullDeviceInfoById(int $deviceId, int $apiKey): array {
        // ...
    }
}
```

Use fully-qualified class names and type declarations for all parameters and return types.

#### Naming conventions

| Element | Convention | Example |
|---------|------------|---------|
| Classes/Namespaces | PascalCase | `Device`, `BaseSql` |
| Methods/Variables | camelCase | `getDeviceInfo()`, `$apiKey` |
| Constants | UPPER_SNAKE_CASE | `DB_TABLE_NAME` |
| Tables/Columns | snake_case | `event_device`, `api_key` |
| Query params | :snake_case | `:api_key`, `:device_id` |

#### Query string style

Use parentheses for multiline SQL queries:

```php
$query = (
    'SELECT id, lang, created
    FROM event_device
    WHERE key = :api_key'
);
```

#### SQL security

Always use named PDO parameters:

```php
// Good
$params = [':api_key' => $apiKey, ':device_id' => $subjectId];
$query = ('SELECT id FROM event_device WHERE id = :device_id AND key = :api_key');
$results = $this->execQuery($query, $params);

// Bad - never concatenate user input
$query = "SELECT * FROM event_device WHERE id = $deviceId";
```

#### Database best practices

Extend `\Models\BaseSql`, use `execQuery()`. Never raw PDO.

#### XSS prevention

Templates auto-escape with `{{ @var }}`. Use `htmlspecialchars()` at output time in PHP:

```php
echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');
```

### Template syntax

tirreno uses the Fat-Free Framework's template engine with includes, variables, and inline PHP:

```html
<include href="templates/parts/headerAdmin.html" />
<div id="wrap">
    <include href="templates/parts/panel/eventPanel.html" />
    <include href="templates/parts/panel/devicePanel.html" />
    <include href="templates/parts/leftMenu.html" />
    <div class="main">
        <include href="templates/parts/forms/globalSearchForm.html" />
        <include href="templates/parts/systemNotification.html" />
        <include href="templates/parts/notification.html" />

        {~
            $country = ['iso' => $IP['country_iso']];
            $subtitle = array();
            if(isset($IP['name']) && !empty($IP['name'])) {
                $subtitle[] = $IP['name'];
            }
            $subtitle = join(', ', $subtitle);
        ~}

        <include href="templates/parts/infoHeader.html" with="title={{@IP.ip}}, country={{@country}}, id={{@IP.id}}"/>
        <include href="templates/parts/widgets/ip.html" />
        <include href="templates/parts/tables/users.html" />
        <include href="templates/parts/tables/events.html" with="showChart=1"/>
    </div>
</div>
<include href="templates/parts/footerAdmin.html" />
```

**Template conventions:**

| Syntax | Purpose | Example |
|--------|---------|---------|
| `{{ @var }}` | Output escaped variable | `{{ @IP.ip }}` |
| `{{ @var \| raw }}` | Output unescaped (careful!) | `{{ @htmlContent \| raw }}` |
| `{{ @arr.key }}` | Access array element | `{{ @IP.country_iso }}` |
| `{~ ... ~}` | Inline PHP code block | `{~ $x = 1 + 2; ~}` |
| `<include href="..." />` | Include template file | `<include href="templates/parts/header.html" />` |
| `<include ... with="..." />` | Include with parameters | `<include href="..." with="title={{@IP.ip}}, id={{@IP.id}}"/>` |
| `{** ... **}` | Template comment (not rendered) | `{**<include href="..." />**}` |

**Template directory structure:**
```
ui/templates/
├── layout.html             # Base layout
├── pages/                  # Page templates
│   ├── admin/              # Admin page templates
│   │   ├── events.html
│   │   ├── ip.html
│   │   ├── users.html
│   │   └── ...
│   ├── login.html
│   ├── signup.html
│   └── ...
├── parts/                  # Reusable components
│   ├── headerAdmin.html    # Common header
│   ├── footerAdmin.html    # Common footer
│   ├── leftMenu.html       # Navigation menu
│   ├── notification.html   # Alert messages
│   ├── forms/              # Form components
│   ├── panel/              # Side panels
│   ├── tables/             # Data tables
│   ├── widgets/            # Dashboard widgets
│   └── choices/            # Filter dropdowns
└── snippets/               # Code snippets
    ├── php.html
    ├── python.html
    └── nodejs.html
```

**Key patterns:**
- Use `<include>` for reusable components (DRY principle)
- Pass data with `with="param1={{@var1}}, param2={{@var2}}"`
- Use `{~ ... ~}` for template logic (preprocessing data before display)
- Comment out unused includes with `{** ... **}`
- Access nested array data with dot notation: `@IP.country_iso`

### Internationalization (i18n)

tirreno uses the framework's built-in internationalization support. Language strings are stored in dictionary files under `app/Dictionary/`.

**Using translations in templates:**
```html
<h1>{{ @DICT.dashboard_title }}</h1>
<button>{{ @DICT.save_button }}</button>
```

**Using translations in PHP:**
```php
$f3 = \Base::instance();

// Get translated string
$message = $f3->get('DICT.welcome_message');

// With variables
$greeting = sprintf($f3->get('DICT.hello_user'), $userName);
```

**Best practices:**
- Never hardcode user-visible strings, use dictionary keys
- Keep dictionary keys descriptive: `dashboard_title`, not `dt1`
- Group related strings with prefixes: `error_invalid_email`, `error_login_failed`
- Don't concatenate translated strings, word order varies by language

### JavaScript coding standards

Follow the ESLint configuration in `eslint.config.js`:

```javascript
// Use const/let, not var
const API_ENDPOINT = '/sensor/';
let eventCount = 0;

// Use arrow functions
const trackEvent = async (userId, eventType) => {
    const response = await fetch(API_ENDPOINT, {
        method: 'POST',
        body: new URLSearchParams({ userName: userId, eventType }),
    });
    return response.ok;
};

// Use template literals
const message = `User ${userId} logged in at ${timestamp}`;
```

#### Page architecture

tirreno uses ES6 modules with a class-based page structure:

```javascript
import {BasePage} from './Base.js';

import {DatesFilter} from '../parts/DatesFilter.js?v=2';
import {SearchFilter} from '../parts/SearchFilter.js?v=2';
import {IpTypeFilter} from '../parts/choices/IpTypeFilter.js?v=2';
import {IpsChart} from '../parts/chart/Ips.js?v=2';
import {IpsGrid} from '../parts/grid/Ips.js?v=2';

export class IpsPage extends BasePage {

    constructor() {
        super('ips');
        this.initUi();
    }

    initUi() {
        const datesFilter  = new DatesFilter();
        const searchFilter = new SearchFilter();
        const ipTypeFilter = new IpTypeFilter();

        this.filters = {
            dateRange:      datesFilter,
            searchValue:    searchFilter,
            ipTypeIds:      ipTypeFilter,
        };

        const gridParams = {
            url:        `${window.app_base}/admin/loadIps`,
            tileId:     'totalIps',
            tableId:    'ips-table',

            dateRangeGrid:      true,
            calculateTotals:    true,
            totals: {
                type: 'ip',
                columns: ['total_visit'],
            },

            isSortable:         true,
            orderByLastseen:    false,

            choicesFilterEvents: [ipTypeFilter.getEventType()],
            getParams: this.getParamsSection,
        };

        const chartParams = this.getChartParams(datesFilter, searchFilter);

        new IpsChart(chartParams);
        new IpsGrid(gridParams);
    }
}
```

**JavaScript conventions:**

| Pattern | Description | Example |
|---------|-------------|---------|
| ES6 modules | Use `import`/`export` | `import {BasePage} from './Base.js';` |
| Class inheritance | Pages extend `BasePage` | `class IpsPage extends BasePage` |
| Version cache-busting | Append `?v=N` to imports | `'../parts/DatesFilter.js?v=2'` |
| Constructor pattern | Call `super()`, then `initUi()` | `super('ips'); this.initUi();` |
| Filters object | Store filter instances | `this.filters = { dateRange, searchValue }` |
| Global app base | Use `window.app_base` for URLs | `` `${window.app_base}/admin/loadIps` `` |

**JavaScript directory structure:**
```
ui/js/
├── endpoints/                  # Page entry points
│   ├── admin_ips.js
│   ├── admin_events.js
│   └── ...
├── pages/                      # Page controllers
│   ├── Base.js                 # Base page class
│   ├── Ips.js                  # IPs page (IpsPage)
│   ├── Events.js               # Events page
│   └── ...
├── parts/                      # Reusable components
│   ├── DatesFilter.js          # Date range filter
│   ├── SearchFilter.js         # Search input filter
│   ├── DataRenderers.js        # Column rendering functions
│   ├── choices/                # Dropdown filters (Choices.js)
│   │   └── IpTypeFilter.js
│   ├── chart/                  # Chart components (uPlot)
│   │   └── Ips.js
│   ├── grid/                   # Data grid components (DataTables)
│   │   └── Ips.js
│   ├── panel/                  # Detail panels
│   └── utils/                  # Utility modules
│       ├── Constants.js
│       ├── String.js
│       └── Date.js
└── vendor/                     # Third-party libraries
```

### File formatting

- **Indentation**: 4 spaces (no tabs), check `.editorconfig` if present
- **Line endings**: Unix (LF)
- **File encoding**: UTF-8
- **Trailing newline**: Yes

### Code comments

tirreno uses a **minimal documentation style**. Write self-documenting code with descriptive names and type declarations. Add comments only to explain "why", not "what":

```php
// Good - explains why
// Skip devices that haven't been updated since last sync
if ($device->lastseen < $lastSync) {
    continue;
}

// Bad - states the obvious
// Check if lastseen is less than lastSync
if ($device->lastseen < $lastSync) {
    continue;
}
```

### Commit messages

Write [good commit messages](https://chris.beams.io/posts/git-commit/). Follow these guidelines:

Format: `<type>: <subject>`. Types: Add, Fix, Update, Remove, Refactor, Docs

```
Add: user session timeout configuration

Allow admins to configure timeout. Default 30 min.
Closes #123
```

### Line endings

All text files should use Unix-style line endings (LF, not CRLF). Windows developers should configure Git: `git config --global core.autocrlf input`

### Testing

Before submitting a pull request:
1. Test your changes on Chrome and Firefox
2. Run code quality checks: phpcs, phpstan, eslint
3. Verify database changes work with PostgreSQL 12+

---

## Resources

| Resource | URL |
|----------|-----|
| Live Demo | [play.tirreno.com](https://play.tirreno.com) (admin/tirreno) |
| Documentation | [docs.tirreno.com](https://docs.tirreno.com) |
| Administration guide | [github.com/tirrenotechnologies/DEVELOPMENT.md/ADMIN.md](https://github.com/tirrenotechnologies/ADMIN.md) |
| GitHub | [github.com/tirrenotechnologies/tirreno](https://github.com/tirrenotechnologies/tirreno) |
| GitLab Mirror | [gitlab.com/tirreno/tirreno](https://gitlab.com/tirreno/tirreno) |
| Docker Hub | [hub.docker.com/r/tirreno/tirreno](https://hub.docker.com/r/tirreno/tirreno) |
| Docker Repo | [github.com/tirrenotechnologies/docker](https://github.com/tirrenotechnologies/docker) |
| Packagist | [packagist.org/packages/tirreno/tirreno](https://packagist.org/packages/tirreno/tirreno) |
| PHP Tracker | [github.com/tirrenotechnologies/tirreno-php-tracker](https://github.com/tirrenotechnologies/tirreno-php-tracker) |
| Python Tracker | [github.com/tirrenotechnologies/tirreno-python-tracker](https://github.com/tirrenotechnologies/tirreno-python-tracker) |
| Node.js Tracker | [github.com/tirrenotechnologies/tirreno-nodejs-tracker](https://github.com/tirrenotechnologies/tirreno-nodejs-tracker) |
| Community Chat | [chat.tirreno.com](https://chat.tirreno.com) |
| Support Email | ping@tirreno.com |
| Security Email | security@tirreno.com |

---

## Found a mistake?

If you have found a mistake in the documentation, no matter how large or small, please let us know by [creating a new issue](https://github.com/tirrenotechnologies/tirreno/issues) in the tirreno repository.

---

## License

tirreno is licensed under the **GNU Affero General Public License v3 (AGPL-3.0)**.

The name "tirreno" is a registered trademark of tirreno technologies sàrl.

---

*tirreno Copyright (C) 2025 tirreno technologies sàrl, Vaud, Switzerland.*

't'
