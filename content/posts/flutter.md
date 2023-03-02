---
title: "Flutter"
date: 2023-03-01T23:19:26+09:00
draft: false
---

# Flutter

## Install

```
sudo snap install flutter --classic
flutter sdk-path
flutter config --no-analytics
```

## Serverpod

### Install

```
dart --disable-analytics
sudo apt update
sudo apt full-upgrade
sudo apt install net-tools
sudo apt autoremove
dart pub global activate serverpod_cli
serverpod
```

### Create a project

```
serverpod create mypod
cd mypod/mypod_server
docker compose up --build --detach
dart bin/main.dart
```

#### Postgres error

```
2023-03-01 15:31:37.227923Z Internal server error. Failed to connect to database in future call manager.
PostgreSQLSeverity.fatal 28P01: password authentication failed for user "postgres"
===== asynchronous gap ===========================
package:postgres_pool/postgres_pool.dart 351:18                 PgPool.run.<fn>
package:retry/retry.dart 131:16                                 RetryOptions.retry
package:postgres_pool/postgres_pool.dart 349:14                 PgPool.run
package:serverpod/src/database/database_connection.dart 443:20  DatabaseConnection.deleteAndReturn
package:serverpod/src/database/database.dart 166:12             Database.deleteAndReturn
package:serverpod/src/server/future_call_manager.dart 100:18    FutureCallManager._checkQueue

Local stacktrace:
#0      FutureCallManager._checkQueue (package:serverpod/src/server/future_call_manager.dart:136:36)
<asynchronous suspension>
```

Remove the Docker Container of Postgres.  
Remove the Docker Image of Postgres.  
Remove the Docker Volume of your project.  
