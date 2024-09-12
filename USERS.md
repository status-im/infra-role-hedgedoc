# Description

This file describes how users can be managed in CodiMD.

# Types

There are three types of user antries in CodiMD PostgreSQL database `Users` table.

* Registered users - When registration is open these update `email` field.
* GitHub OAuth users - When logged in they fill `profile` with JSON from GH.
* Google OAuth users - When logged in they also fill `profile`

We used to have open registrations, but those were closed.
Currently we only accept GitHub and Google logins.

# Queries

```sql
SELECT
  u.id,
  u.provider,
  u.json->'username' AS username,
  u.json->'displayName' AS name,
  (CASE WHEN u.provider::text = '"github"'
   THEN (u.json->'_json'->'email')::text
   ELSE u.email END) AS email
FROM (
    SELECT
        profileid AS id,
        email AS email,
        profile::json AS json,
        profile::json->'provider' AS provider
    FROM "Users") AS u;
```
```
          id           | provider |   username   |           name            |             email
-----------------------+----------+--------------+---------------------------+------------------------
 19521990              | "github" | "jlokier"    | "Jamie Lokier"            | "jamie@shareable.org"
 2212681               | "github" | "jakubgs"    | "Jakub"                   | "jakub@gsokolowski.pl"
 116095778576207530385 | "google" |              | "Jamie Lokier"            |
 5702426               | "github" | "Samyoul"    | "Samuel Hawksby-Robinson" | "samuel@samyoul.com"
 5483559               | "github" | "sachayves"  | "Sacha Saint-Leger"       | "sacha@ethereum.org"
 ...
```
