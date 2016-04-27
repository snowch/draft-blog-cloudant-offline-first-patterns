Cloudant Offline-first Patterns - Part 2
============

Part 1 is available [here](./draft_part1.md)

This section should add more pertinent information...

# Database per user pattern

## Best practice

- Use server side user dbs just for sync - you don’t want to be building/having to manage design docs/indexes
- Use a third party application like [superlogin](https://www.npmjs.com/package/superlogin)  to orchestrate creating database and API keys for adding new users.

## Moving data using document state

## Using _db_updates watcher

# Filtered replication pattern

## Best practice

## Moving data using document state

## Write access control using VDU

# Future improvements

Stefan's project ...

