# Distributed Find By ID To Update
This project is implemented by TDD to solve the issue of updating an object in a distributed system realm. 
The challenge is to guarantee the consistency of data after likelihood of **concurrent updates**.  
The solution is to provide added features to a simple and common ``findById`` function like **distributed lock mechanism**.

## Draft of idea
- The ``findById`` function.
  - It is needed to check that if ``object_id`` in the request has a valid ``transaction_id`` in the cache or not.
    - If there is a valid ``transaction_id`` and the ``state`` is LOCK and ``user_id`` is not the same then return bad request error! 
    - If there is no valid ``transaction_id`` It is needed to generate a new ``transaction_id``, 
     then put the generated ``transaction_id`` as a key into the cache with values of ``transaction_state``(default=LOCKED), ``object_id``, ``user_id``, and ``expiration_time``, 
     and finally put the ``object_id`` as a key into the cache with value of ``transaction_id``.
  - Then return the object to user with ``transaction_id`` and ``update_tag`` 
- The ``update`` function
  - check the ``transaction_id`` of the request is not null.
  - check the ``transaction_id`` of request is correctly mapped to ``user_id`` of the request.
  - check the ``expiration_time`` of the ``transaction_id`` is valid.
  - check the state of ``transaction_id`` in the cache should be ``LOCK``:
      - if the state is ``Updated`` then return 200 status.
  - check the user privilege should be ``update:object`` by verifying ``JWT`` token and its' claim.
  - If every precondition is valid then go for an update.
  - After update the state of ``transaction_id`` should change to ``Updated``.
  - Finally, return 200 status.
      