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
### Find By ID to Update
The goal of the function is to provide a proper and secured ``Membership`` object in order to update it later.
#### Pre conditions
- The request should contain ``membership_id``
  - The ``membership_id`` is a long number (1 to 10 billions)
- The request should contain ``user_id``
  - The ``user_id`` is an integer number (1 - 10_000) 
  - The ``user_id`` should have ``findByIdToUpdate:membership`` privilege
  - The request should contain a JWT token which includes the ``user_id`` and ``user_access``
  - The JWT token should be verified
#### Processes
- If ``isLockedToUpdate(membership_id)`` equals to ``true`` then return an ``not accessable error``.
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