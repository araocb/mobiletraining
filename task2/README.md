## Hands on with the Sync Gateway Endpoints

Install Postman (if you haven’t already), then import the shared Postman collection into your Postman workspace.

## Background Changes

In Task 1, you created a bucket on your Couchbase Server instance called `mobilebucket`. Create a scope for this called `testscope`, and under this scope, create `testcollection`.

### Task

1. Check that Sync Gateway is running by calling the **SGW Working Endpoint** request in the shared Postman collection.

2. Create a Sync Gateway database on the `mobilebucket` bucket using the **Create Database** request in the shared Postman collection.

    You will likely see a connection error. This happens because the **Create Database** endpoint is part of the **Admin API**. Sync Gateway exposes three APIs: **Public**, **Admin**, and **Metrics**.

    By default, the Admin API only listens on `localhost`. Since you’re sending the request from outside the Docker container, the connection fails. To fix this, update the Sync Gateway configuration so the Admin API binds to `0.0.0.0` (making it accessible outside the container).

    Update your Sync Gateway configuration to the following and restart Sync Gateway for the change to take effect:

    ```json
    {
    "api": {
        "admin_interface": "0.0.0.0:4985"},
    "bootstrap": {
        "server": "couchbase://<ip address>",
        "username": "Administrator",
        "password": "password",
        "server_tls_skip_verify": true,
        "use_tls_server": false
    },
    "logging": {
        "console": {
        "enabled": true,
        "log_level": "info",
        "log_keys": ["*"]
        }
    }
    }
    ```

    After this change, you should be able to proceed with Task 2.

    ```json
    {
    "bucket": "mobilebucket",
    "num_index_replicas": 0,
    "name": "mydb",
    "scopes": {
        "testscope": {
            "collections": {
                "testcollection": {}
                }
            }
        }
    }
    ```

    Note: This uses a **non-default scope and collection**, so make sure they exist on the Couchbase Server side first.

    This is the minimal configuration for the exercise. For the full set of options, see the [Sync Gateway Admin REST API reference](https://docs.couchbase.com/sync-gateway/current/rest-api/rest_api_admin.html#tag/Database-Management/operation/put_db-).

    Now that we are done with the DB configuration, you can proceed with the next steps!

3. Create a document on the keyspace `mobiletraining`.`testscope`.`testcollection` using Sync Gateway. 

4. Get the document you created in the previous step using Sync Gateway endpoints.

5. Update the document using Sync Gateway endpoints.

6. Get the current DB configuration.

7. Try updating the current DB Configuration.

8. Get the new DB configuration.

9. Create a basic Sync Function to assign channels to a document.

10. Create a new Sync Gateway user who has access to the channel.

11. Using the newly created Sync Gateway user, fetch the previously created document via the **Public API** endpoint.

