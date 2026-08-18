---
title: "Increasing the Nextcloud AIO User Limit"
date: 2026-08-18 12:37:00 +0330
categories: [Nextcloud, AIO]
tags:
  - linux
  - devops
  - nextcloud
  - docker
author: arman
#description: >
#  Nextcloud AIO has a default user limit of **100 users** for the one-click installation. For testing, lab environments, or other situations where this limitation is not suitable, the configured limit can be increased.
toc: true
comments: false
math: false
mermaid: true
pin: false
published: true
---
Nextcloud AIO has a default **100-user limit**. This guide shows how to increase it, for example, to **1000 users**, by modifying `aio.config.php`.

> **Important:** This is a technical configuration change. Nextcloud AIO is licensed under **AGPL-3.0**, while Nextcloud also offers commercial Enterprise services. Review the applicable terms before using this in production.

## Check the Current Limit

```bash
docker exec nextcloud-aio-nextcloud \
  grep "one-click-instance.user-limit" \
  /var/www/html/config/aio.config.php
```

Expected:

```php
'one-click-instance.user-limit' => 100,
```

## Increase the Limit

Change it to 1000:

```bash
docker exec nextcloud-aio-nextcloud \
  sed -i "s/'one-click-instance.user-limit' => 100/'one-click-instance.user-limit' => 1000/" \
  /var/www/html/config/aio.config.php
```

Verify:

```bash
docker exec nextcloud-aio-nextcloud \
  grep "one-click-instance.user-limit" \
  /var/www/html/config/aio.config.php
```

Expected:

```php
'one-click-instance.user-limit' => 1000,
```

This **increases the limit**; it does not completely remove it.

## AIO Updates

AIO updates or container recreation may restore the default value. After updates, verify the setting again:

```bash
docker exec nextcloud-aio-nextcloud \
  grep "one-click-instance.user-limit" \
  /var/www/html/config/aio.config.php
```

If it has returned to `100`, apply the modification again.
