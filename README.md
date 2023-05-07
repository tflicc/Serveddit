# ⁉️ &nbsp; About

#### 🚧&nbsp; This app is under construction and not suitable for public use until this notice is taken down.

This sever manages requesting and refreshing a client's `access_token` using securely stored `refresh_tokens`. <b>That's it.</b>

# ⚙️ &nbsp; The Process

When the client (i.e. Snoop) sends a user to their browser to authorize the requested scopes, it also sends a POST request to `/client/new`. The POST contains a `UUID` that Reddit requires to verify the authentication request; Serveddit uses the `UUID` to create and manage `Client` SQLAlchemy models.

When the user chooses to approve/deny scope access, Reddit will redirect to `/new/user`. The redirect contains the required information to create a new `User` instance; specifically, `clientUUID` and `refresh_token`.

After a user has logged in to the app, if / when a client tries to use an expired `access_token` (they only last 1 hour) it simply sends a GET request to `/{clientUUID}/{redditUsername}`. The server refreshes the token and sends the fresh `access_token` to the client.

# 💩&nbsp; Endpoints

## Create

- `POST /new/client/{UUID}` - create a new `Client` instance in the database
- `POST /new/user` - create a new `User` instance in the database.
  - ⚠️ &nbsp; Clients should <b >not</b> try to use this endpoint: Reddit sends a redirect here containing the necessary info
  - ⚠️ &nbsp; Clients must exist before a new user can be created

## Read

- `GET /{clientUUID}/{userID}` - Try and fetch a valid `access_token` for the `User` managed by `clientUUID`. `User`s are searched by `userID`, an int value stored by the client.
- `GET /{clientUUID}` - Check if `clientUUID` is known to the server
- `GET /{clientUUID}/users` - Fetch a list of `userID`s managed by `clientUUID`

## Update

- `PUT /{clientUUID}/{redditUsername}` - Replace the `refresh_token` for `redditUsername` with the `refresh_token` contained in the payload. Both `clientUUID` and its managed `redditUsername` must already exist.

## Delete

- `DELETE /{clientUUID}` - Delete the `Client` instance identified by `clientUUID`, as well as all `User` instances it managed. <span style="color:red">Cannot be undone</b>.
- `DELETE /{clientUUID}/{redditUsername}` - Delete the `User` instance `redditUsername`
