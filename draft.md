<header>
Cloudant offline-first patterns
============
</header>

Offline-first applications provide a better, faster user experience — both offline and online — by storing and accessing data locally and then synchronizing this data with the cloud when an Internet connection is available.  

As an app developer, it’s tempting to let your mobile backend handle all of your data access. This is a clean and simple architecture, with less code to write. The problem is that networks are unreliable. When the network doesn’t work, neither does your app — and a broken app means unhappy, frustrated users.

Cloudant supports a model of offline-first where data can be synchronised between a Cloudant database and a device.  This article explores two high level patterns for offline-first data access with Cloudant:

- Database per user offline-first pattern
- Filtered replication offline-first pattern

# sharding data for sync (aka Database per user)

The database per user offine-first pattern refers to having a seperate database on a remote Cloudant cluster for each user running the mobile application.  In this scenario, the mobile application running on the user's device uses Cloudant Sync to keep a local copy of the data and synchronises the data between the device and the Cloudant cluster.  



