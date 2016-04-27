
Cloudant Offline-first Patterns - Part 1
============

*Note: view the raw document source for review comments.*

---

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

<start-comment-marker>In the filtered replication pattern, each user has access to a shared database on Cloudant and can access all data in that database<end-comment-marker>.  If more granular access control is required, a custom proxy service will need to be implemented that intercepts all calls to Cloudant and performs custom authorisation.

[comment]: # (Consider being more explicit – even if only data relevant to each user is sync’d to a device, they will require read and write permissions across the whole database for the sync to work.)

## Number of active users

Each active database [see Definition of 'active database'] on Cloudant will have multiple processes to manage the activity for that database.  A lot of active databases will eventually starve the Cloudant <start-comment-marker>cluster<end-comment-marker> of resources.  Exactly how many will depend on factors such as the size of your cluster and the other workloads being served by your cluster.  As a rule of thumb, more than a few hundred active databases may result in too much load on your cluster.

[comment]: # (Worth considering how this might be phrased when customers don’t have a cluster to themselves e.g. MT or cloud consumption model.)

| Definition of 'active database'     | 
| ------------- |
| An active database is ...    "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."  |

Developer's implementing the database per user pattern will need to consider how many active databases they have on Cloudant.  In filtered replication, data from user devices is replicated to a single database on Cloudant, resulting in much less load on the cluster.

## Backend queries across all of the user data

A query in Cloudant can only be performed on a single database.  However, usually there will be a requirement to perform backend queries across all of the user data.

With the database per user pattern, cross database queries could be implemented by writing application logic to make individual queries across to all of the user databases on the cluster and then aggregate the results of those queries.  However, the most common way to implement this requirement is to create a master database that will hold all of the data and then set up a replication for each user database on Cloudant to continuously synchronise data from that database a master database.  The backend administrator can then query the master database rather than having to query the individual user databases.  Below is a high level diagram showing this approach:

![single_db_per_user_with_master](https://github.com/snowch/draft-blog-cloudant-offline-first-patterns/blob/master/single_db_per_user_with_master.png)

Note that similar to the issue with the number of active databases, continuous replications also place a burden on the cluster and there will be a limit to the number of continuous replications.  There is an alternative to having continous replications by developing a custom service.  See the section "Replacing continuous replications with _db_updates watcher" in part 2 for more information.

The filtered replication pattern has a single database on Cloudant, so natively supports querying all of the data.

## Movement of data (documents) between user databases

Often there will be a requirement for data to move from one database to another.  For example, consider a basic application that tracks tasks being worked on by a team.  In this simple example, a task can only be worked on by one member of the team at a time so will only exist in one user database at a time.  How would you manage allocating a task from one team member to another team member?

In the database per user pattern where you do not have a master database, you could achieve the movement of data using a backend service that:

- selects the document(s) from the current user's database
- inserts the document(s) into the target user's database
- deletes the document(s) from the current user's database.

<start-comment-marker>If you are using the *database per user pattern* with a master database, and making the assumption that the backend administrator will allocate a task from one user to another by making changes in the master database this process gets considerably more complex.  One approach is described in detail in part 2 of the blog, "Database per user pattern / moving data using document state".

The *filtered replication pattern* has similar complexity as the database per user pattern with a master database for moving documents between users.  One approach is described in detail in part 2 of the blog, "Filtered replication pattern / moving data using document state".<end-comment-marker>

[comment]: # (This seems like a long-winded redirect to another section.)

## Sync speed

Cloudant Sync (and other clients that implement CouchDB synchronization) use a _changes feed that exists on each database to discover which documents have changed between sync operations. The initial sync begins from the start of the changes feed, and subsequent syncs will continue from the change that they last processed.

When you are using the database per user pattern, each user database on Cloudant contains the user's data so iterating through the changes should be quick. Subsequent syncs will only need to iterate changes relevant to that user so, again, this number will normally be a relatively small proportion of the total number of changes.

The filtered replication patterns uses a master database from which all user database retrieve their data.  If the master database is large, the first sync may take a long time as it will have a lot of changes to run through the filter.  If there are lots of changes in the master database (e.g. from other users), each subsequent sync will need to iterate though all of those changes to determine which documents should be sync’d based on the filter.

# Summary


|                                        |	Db per user pattern |	Filtered replication pattern |
|----------------------------------------|---------------------|------------------------------|
| Access control	                       | Each user has their own database that only they have access to. |	Each user has access to all of the data in the master database.  Some controls can be implemented for writes.  A proxy service could be created for finer access control but may be complex. |
| Number of active users	               | The number of active databases supported by the Cloudant cluster will be constrained by available resources. |	No impact. |
| Backend queries across all data	       | Usually requires a master database and replications from user databases.  The number of continous replications will be limited by available server resources. Custom more efficient replication orchestration can be implemented but maybe complex.| This pattern supports querying across all data by default. |
| Movement of data between user databases	| Can be complex depending on the use case.	Can be complex depending on the use case.
| Sync speed                              |	Initial and subsequent replications are usually quick.	| Initial replication may be slow if the master database is large.  Subsequent replications may be slow if their are lots of changes in the master database. |

See [part 2](./draft_part2.md) for further considerations.

