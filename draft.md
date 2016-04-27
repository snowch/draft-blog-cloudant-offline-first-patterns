<header>
Cloudant offline-first patterns
============
</header>

Offline-first applications provide a better, faster user experience — both offline and online — by storing and accessing data locally and then synchronizing this data with the cloud when an Internet connection is available.  

As an app developer, it’s tempting to let your mobile backend handle all of your data access. This is a clean and simple architecture, with less code to write. The problem is that networks are unreliable. When the network doesn’t work, neither does your app — and a broken app means unhappy, frustrated users.

Cloudant supports a model of offline-first where data can be synchronised between a Cloudant database and a device.  This article explores two high level patterns for offline-first data access with Cloudant:

- Database per user offline-first pattern
- Filtered replication offline-first pattern

# Sharding data for sync (aka Database per user)

The database per user offine-first pattern refers to having a seperate database on a remote Cloudant cluster for each user running the mobile application.  In this scenario, the mobile application running on the user's device uses Cloudant Sync to keep a local copy of the data and synchronises the data between the device and the Cloudant cluster.  

![single_db_per_user](https://github.com/snowch/draft-blog-cloudant-offline-first-patterns/blob/master/single_db_per_user.png)

# Filtered Replication

In the filtered replication offine-first pattern each user has access to a shared master database in Cloudant.  In this scenario, the mobile application running on the user's device uses replication to keep a local copy of the data and synchronise the data between the device and the Cloudant cluster.  The replication is usually filtered so that only a subset of the data is synchronised from the master database to the device.

![filtered_replication](https://github.com/snowch/draft-blog-cloudant-offline-first-patterns/blob/master/filtered_replication.png)

# Which pattern should I use?

This section is intended to be a rough guide only.  Read the whole blog post to understand the idiosyncrasies of each pattern.

## Access control

Permissions in Cloudant are at the database level.  A user that has access to a database has access to all data in the database.

The database per user pattern allows you to create a unique database on Cloudant for each users and create a unique API key for each user.  A user's API key is granted access to her user database, allowing only them to access their data on Cloudant.

In the filtered replication pattern, each user has access to a shared database on Cloudant and can access all data in that database.  If more granular access control is required, a custom proxy service will need to be implemented that intercepts all calls to Cloudant and performs custom authorisation.

## Number of active users

Each active database [see Definition of 'active database'] on Cloudant will have multiple processes to manage the activity for that database.  A lot of active databases will eventually starve the Cloudant cluster of resources.  Exactly how many will depend on factors such as the size of your cluster and the other workloads being served by your cluster.  As a rule of thumb, more than a few hundred active databases may result in too much load on your cluster.

| Definition of 'active database'     | 
| ------------- |
| An active database is ...    "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."  |

Developer's implementing the database per user pattern will need to consider how many active databases they have on Cloudant.  In filtered replication, data from user devices is replicated to a single database on Cloudant, resulting in much less load on the cluster.


