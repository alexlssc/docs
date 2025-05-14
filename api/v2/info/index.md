## Authentification
The authentication system on Partoo API is using API Key that should be put in the header of the request (the name
of the header is `x-APIKey`).
An api_key is linked to a user. This user's role will give you different access level to the API features.

|  Security Scheme Type |  API Key |
|-----------------------|----------|
| Header parameter name | x-APIKey |

## Environments
We provide two public environments, **Sandbox** and **Production**.
- **Sandbox** is a playground environment for you to test your API integration without having to worry about side effects.
- **Production** is where your "real" data should live. Changes made there are propagated to publishers.

## First steps
First, it's a good idea to start building and testing your integration in the Sandbox environment.

Ask your Partoo Account Manager to send you an API Key for Sandbox. You'll need to pass this API Key in all your API calls in order
to be authentified, in the header `x-APIKey`. More details in the <a href="#section/Authentication">Authentication section</a>.

From there, you can call any route that your user has access to. To know the exact API endpoint to use, you can click
on the route url above the "Response samples", and this will give you the urls for Production and Sandbox.

If you need a UI access to Sandbox, ask your Partoo Account Manager for one. Be aware, Sandbox environment is empty by default,
and has limited functionalities.

Once you're confident that your integration is production ready, ask your Partoo Account Manager for a user access in Production,
and to give you access to the API Key Manager tool. From there, you'll be able to generate and manage your own API Keys inside the Partoo Web Application.

You can now use the Production API Key to start calling the Production API.

## Error codes
Partoo uses conventional HTTP response codes to indicate the success or failure of an API request.
In general: Codes in the 2xx range indicate success. Codes in the 4xx range indicate an error that failed given the information provided.
Codes in the 5xx range indicate an error with Partoo servers (these are rare).
List of potential error code

### 400 - Bad request
```json
"error": {
    "json": { }
}
```
### 401 - No valid API key provided
```json
"error": {
    "authentication": "User not authenticated"
}
```
### 403 - The API key doesn't have permissions to perform the request
```json
"error": {
    "authorization": "Operation not allowed"
}
```
### 404 - The requested resource doesn't exist
```json
"error": {
    "json": "Resource not found"
}
```
### 415 - Invalid media type
Our API framework requires `Content-Type` header to be provided as `application/json` on `POST`, `PUT` and `PATCH` operations.

If your API integration doesn't properly send this header, an error may appear
```json
{
    "errors": {
        "json": "Unsupported media type. Please use application/json"
    }
}
```

### 429 - Too Many Requests
See [Rate Limiting](#tag/Getting-started/Rate-Limiting)


## Security Disclaimer
**The Partoo Rest API is meant to be used only from a server that you control.** You'll need an API Key in order to make API calls,
and you should treat this key as secret as a password. The API Key should **never** be transmitted to or included into the front-end.
If you need to make calls from a front-end, you should proxy them by a server that you control, that will add your API Key.

Some functionalities are disabled in Sandbox, to avoid having a production impact:
- Business data will not be sent to the different publishers such as Google, Facebook, etc.
- It's not possible to import Google and Facebook account listings.
- There's no real reviews data, or real analytics data. The account starts empty, but can be filled by fake data if needed. For this, contact you Partoo Account Manager.

## Rate Limiting
In order to keep service available for everyone, our APIs are protected using a Rate Limit.

Each organization is limited to **300 calls per minute** by default.
If your integration requires more calls for any reason, get in touch with your customer success to update this limit, with review from our Tech team.

Any call beyond the reached limit will result in an error with [status code 429](https://developer.mozilla.org/fr/docs/Web/HTTP/Status/429).
A http header `Retry-After` is added to the response to hint when a query can be retried (60s later).


## Resources ans roles


### Resources structure
Partoo data is organised around 5 main resources:
1. **Provider:** A provider represents the entity that signs the contract with the client using Partoo solution & products.
An obvious example of provider is Partoo itself but a provider can also be a reseller of Partoo solutions.
A provider owns `organizations`, `businesses`, `users` and `groups`. If you are a Partoo reseller there will be a provider resource representing
you inside Partoo app.
2. **Organization:** An organization represents the legal entity, most likely a commercial company, that owns `businesses` (or listings).
If you are a Partoo client there are one or several organizations representing your companies.
An organization belongs to a `provider`.
3. **User:** A user can be a Partoo app user or Partoo API user.
A user belongs to an `organization` and has a `role` which gives him different levels of access on the different resources on Partoo (see section below).
4. **Business:** A business represents a listing. It belongs to an `organization`
5. **Group:** A group contains businesses, each organization can have several groups of businesses.

### Roles
To use Partoo Rest API, you need an `api_key` (see authentication section). An `api_key` authenticates a user and each user has a `role`.

A `role` defines for each resource (for instance `user`) a`READ` and/or `WRITE` access with the scope on which this access can be used.

For instance a user with `BUSINESS_MANAGER` role has `WRITE` access on its own user and `READ` acces to all the users of its organization.

For now there are 5 roles available:
- `PROVIDER` role is meant for reseller admin user. It can manage organizations, users and businesses of a provider
- `ORG_ADMIN` role is meant for client admin user. It can manage the user and businesses of its organization
- `GROUP_MANAGER` role is meant for client group manager. It can manage several businesses and users that belong to the group he managed
- `BUSINESS_MANAGER` role is meant for client business manager. It can manage several businesses, only if these businesses are in the same group.
- `PUBLISHER` role is meant for publisher wanting to use Partoo as a data source. It can read Partoo businesses subscribed to presence management product

#### PROVIDER
`PROVIDER` role is meant for reseller admin user. It can manage its provider organizations, users and businesses.

#### Read access
| Resource     |	Scope	| Details                                              |
| ------------ | -------- | ---------------------------------------------------- |
| User	     | Provider | Can access the users that shares its provider        |
| Organization | Provider | Can access the organizations that shares its provider|
| Group        | Provider | Can access the groups that share its provider        |
| Business	 | Provider | Can access the businesses that share its provider    |
| Category	 | All	    | Can access all categories                            |

#### Write access
| Resource     |	Scope	    | Details                                          |
| ------------ | ------------ | ------------------------------------------------ |
| User	     | Provider     | - Can create user, it will share its provider <br> - Can update user that shares its provider <br> - Can give role `ORG_ADMIN` and `BUSINESS_MANAGER` to user |
| Organization | Provider     | - Can create organization, it will share its provider <br> - Can update org that shares its provider |
| Group        | Provider     | - Can create group, it will share its provider <br> - Can update group that shares its provider |
| Business     | Provider     | - Can create business, they will share its provider (and its org_id if no org_id given) <br> - Can update businesses that shares its provider |
| Category	 | not writable	|                                                  |

### ORG_ADMIN
`ORG_ADMIN` role is meant for client admin user. It can manage its organization users and businesses.

#### Read access
| Resource     |	Scope	    | Details                                           |
| ------------ | ------------ | ------------------------------------------------- |
| User	     | Organization | Can access the users that shares its org_id       |
| Organization | Organization | Can access only its own org                       |
| Group        | Organization | Can access the group that shares its org_id       |
| Business	 | Organization | Can access the businesses that shares its org_id  |
| Category	 | All	        | Can access all categories                         |

#### Write access
| Resource     |	Scope	    | Details                                           |
| ------------ | ------------ | ------------------------------------------------- |
| User	     | Organization | - Can create user, it will share its provider and its `org_id`. <br> - Can update user that shares its `org_id` <br> - Can give the role `GROUP_MANAGER` and `BUSINESS_MANAGER` to its user |
| Organization | Organization | - Can update itself <br> - Cannot create new org. |
| Group        | Organization | - Can create group, it will share its provider and its `org_id` <br> - Can update group that shares its `org_id`
| Business	 | Organization | - Can create business, it will share its provider and its `org_id` <br> - Can update businesses that shares its `org_id`|
| Category	 | not writable |                                                   |

### GROUP MANAGER
`GROUP_MANAGER` role is meant for client group manager. It can manage several businesses and users that belong to the group he managed.

#### Read access
| Resource     |	Scope	    | Details                                            |
| ------------ | ------------ | -------------------------------------------------- |
| User	     | Organization | Can access the `ORG_ADMIN` that shares its `org_id` and the `GROUP_MANAGER`and `BUSINESS_MANAGER` that belong to its group |
| Organization | Organization | Can access only its own org                        |
| Group        | Group        | Can access only its group                          |
| Business	 | Group        | Can access the businesses that belong to its group |
| Category	 | All	        | Can access all categories                          |

#### Write access
| Resource     |	Scope	    | Details                                            |
| ------------ | ------------ | -------------------------------------------------- |
| User	     | Group        | - Can create user, it will share its provider, its `org_id` and its `group_id`. <br> - Can update user that shares its `org_id` <br> - Can only give the role `BUSINESS_MANAGER` to its user |
| Organization | No access    |                                                    |
| Group        | Group        | - Can update its own group <br> - Cannot create group |
| Business     | Group        | - Can update businesses that belong to its group <br> - Cannot create business |
| Category	 | not writable |                                                    |

### BUSINESS_MANAGER
`BUSINESS_MANAGER` role is meant for client business manager. It can manage several businesses, only if these businesses are in the same group.

#### Read access
| Resource     |	Scope	    | Details                                           |
| ------------ | ------------ | ------------------------------------------------- |
| User	     | Organization | Can access the users that shares its org_id       |
| Organization | Organization | Can access only its own org                       |
| Group        | Group        | Can access only its group                         |
| Business	 | Business     | Can access the businesses that it has direct access to |
| Category	 | All	        | Can access all categories                         |

#### Write access
| Resource     |	Scope	    | Details                                           |
| ------------ | ------------ | ------------------------------------------------- |
| User	     | User         | - Can update its user only                        |
| Organization | No access    |                                                   |
| Group        | No access    |                                                   |
| Business	 | Business     | - Can update businesses that it has direct access to <br> - Cannot create business |
| Category	 | not writable |                                                   |

### PUBLISHER
`PUBLISHER` role is meant for publisher wanting to use Partoo as a data source. It can read Partoo businesses subscribed to presence management product.

#### Read access
| Resource     |	Scope	                         | Details                                |
| ------------ | --------------------------------- | -------------------------------------- |
| User	     | No access                         |                                        |
| Organization | All                               | Can access all Partoo organizations    |
| Group        | No access                         |                                        |
| Business	 | Subscribed to Presence Management | Can access the businesses subscribed to presence management product |
| Category	 | All	                             | Can access all categories              |

## Technical Support
For any questions related to the API functionality, please contact the technical support team at api@partoo.fr.
