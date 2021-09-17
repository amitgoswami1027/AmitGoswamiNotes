# CHUBBY - DISTRIBUTED LOCKING SERVICE
# Optimistic vs. Pessimistic Locking

## Pessimistic Locking - Pessimistic Lock is where you assume that all the users are trying to access the same record and it literally locks the record exclusively for the first started transaction until it is completed successfully or failed. Then the lock is released and the next transaction on the record is handled in the same way.Pessimistic Lock is where you assume that all the users are trying to access the same record and it literally locks the record exclusively for the first started transaction until it is completed successfully or failed. Then the lock is released and the next transaction on the record is handled in the same way.
* In our case, if we apply Pessimistic Lock, the first user to come and buy the last item in the stock will click on “Purchase”.
* This will lock the object until the payment is completed or failed.
* Let’s say User A was able to pay for it and the stock value for that item is set to 0 now.
* All the other users have to wait during this process.
* Now all the other users will see that the item went out of stocks, and cannot do anything with the item.

## Optimistic Lock - Optimistic Lock is where you manage your data by checking a special value in the database — it is often a version number, timestamp, date, etc.— before you read/write to the data to make sure that the data you are dealing with is not stale/old/changed since you’ve last viewed. If the data is stale, the transaction is not completed successfully and an error is thrown to indicate that. Something like: “The record you attempted to edit was modified by another user after you got the original value”.
* In our case, if we apply Optimistic Lock, the first user to come and buy the last item in the stock will click on “Purchase”.
* Let’s say User A was able to pay for it and before the payment step the stock value is checked before committing to change it from 1 to 0.
* If the version numbers match the operation is committed and the item goes out of stock.
* Now all the other users that try to purchase that the item will be warned about that the item is no longer available, right at the moment they try to buy, pay or add it to their baskets.

Please note that:
* Optimistic locking is just a mechanism to prevent processes from overwriting changes by another process. Optimistic locking is not a magic wand to manage or auto-merge any conflicting changes. It can only allow users to alert or notify about such conflicting changes.
* Optimistic locking works by just comparing the value of the “version” column. Thus, optimistic locking is not a real database lock.

## SUMMARY
Both pessimistic and optimistic locking mechanisms have use cases that they fit in, however, for systems with high loads of updates which hold locks for a significant time interval, the optimistic shows better efficiency. Also, it must be noted that the higher RPS doesn’t necessarily means a faster algorithm, since an important amount of requests are failed because of the lock-wait errors in the pessimistic mechanism.

# DISTRIBUTED LOCKING MECHANISM (CHUBBY) 
Design a highly available and consistent service that can store small objects and provide a locking mechanism on those objects. Chubby is a service that provides a distributed locking mechanism and also stores small files.

Primarily Chubby was developed to provide a reliable locking service. Over time, some interesting uses of Chubby have evolved. Following are the top use cases where Chubby is practically being used:
* Leader/master election
* Naming service (like DNS)
* Storage (small objects that rarely change)
* Distributed locking mechanism



