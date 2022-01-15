# Distributed Find By ID To Update
This project is implemented by TDD to solve the issue of updating an object in a distributed system realm. 
The challenge is to guarantee the consistency of data after likelihood of **concurrent updates**.  
The solution is to provide added features to a simple and common ``findById`` function like **distributed lock mechanism**.

## Explain the Algorithm
### Domain
#### Membership
- ``membership_id``: (1 to 10 billions), not nullable
- ``membership_type``: {Gold, Silver, Bronze}, not nullable
- ``membership_credit``: (0 to 100_000), not nullable
- ``membership_start_date``: not nullable
- ``membership_expiration_date``: not nullable
### Function: Find Membership By ID to Update
The goal of the function is to provide a proper and secured ``Membership`` object in order to update it later.
#### Pre conditions
- The request should contain ``membership_id``
  - The ``membership_id`` is a long number (1 to 10 billions)
- The request should contain ``user_id`` and proper ``privileges``
  - The ``user_id`` is an integer number (1 - 10_000) 
  - The ``user_id`` should have ``findByIdToUpdate:membership`` privilege
  - The request should contain a JWT token which includes the ``user_id`` and ``user_access``
  - The JWT token should be verified
#### Processes
- If (``isLockedToUpdate(membership_id)`` and ``isMembershipLockedByUserId(membership_id, user_id)``) equals to ``true`` then
  - return membership object
- If ``isLockedToUpdate(membership_id)`` = ``true`` and ``isMembershipLockedByUserId(membership_id, user_id)`` = ``false`` then 
  - return ``not accessable error``.
- If ``isLockedToUpdate(membership_id)`` equals to ``false`` then 
  - do ``lockToUpdate(membership_id)`` and receive the ``transaction_id``
    - create a random uuid for ``transaction_id``
    - insert the (key, value) (``transaction_id``, ``{state(LOCKED, UN_LOCKED), user_id, object_id, expiration_time(Long)}`` into the ``cache``
    - insert the (key, value) (``membership_id``, ``transaction_id``) into the ``cache``
    - return ``transaction_id``

#### Post conditions
- return ``ObjectToUpdate`` json contains
  - ``transaction_id``
  - ``object_id``
  - ``{membership_type, membership_credit, membership_start_date, membership_expiration_date}``

### Function: Update Membership
The goal of the function is to update a membership object which is not locked.
#### Pre conditions
- The request should contain ``transaction_id``
  - should be ``uuid``
- The request should contain ``membership_id``
  - The ``membership_id`` is a long number (1 to 10 billions)
- The request should contain ``user_id`` and proper ``privileges``
  - The ``user_id`` is an integer number (1 - 10_000)
  - The ``user_id`` should have ``findByIdToUpdate:membership`` privilege
  - The request should contain a JWT token which includes the ``user_id`` and ``user_access``
  - The JWT token should be verified
- The request should contain the changed values of the object's fields
  - refer to domain
#### Processes
- If (``isLockedToUpdate(membership_id)`` and ``isMembershipLockedByUserId(membership_id, user_id)``) equals to ``true`` then
  - update the object
  - return ``succeed``
- If ``isTransactionUnLocked(transaction_id)`` then return ``succeed``
- If ``isLockedToUpdate(membership_id)`` equals to ``true`` then return ``not accessible error``
  - Note: never tell that why the object is not accessible? **because it is not secured**.
#### Pos conditions
- return ``UpdatedObject``
  - ``transaction_id``
  - ``object_id``
  - ``{membership_type, membership_credit, membership_start_date, membership_expiration_date}``

### Sub Functions:
##### function isLockedToUpdate(membership_id)
- is ``membership_id`` is in the cache?
  - (yes)
    - get the ``transaction_id`` of the ``membership_id``
      - is the ``expiration_time`` is valid?
        - (yes) return ``true``
        - (no) return ``false``
      - is the ``state`` is ``LOCKED``?
        - (yes) return ``true``
        - (no) return ``false``
  - (no) return false

##### function isMembershipLockedByUserId(membership_id, user_id)
- is ``membership_id`` is in the cache?
  - (yes)
    - get the ``transaction_id`` of the ``membership_id``
      - if ``expiration_time`` is valid and ``state`` is ``LOCKED`` and ``user_id`` is the same?
        - (yes) return ``true``
        - (no) return ``false``
  - (no)