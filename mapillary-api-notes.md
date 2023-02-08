Most content taken from: https://www.mapillary.com/developer/api-documentation?locale=fr_FR#embed

Mapillary's v4 API provides two surfaces for interacting with the data:
vector tiles and entity endpoints.
Vector tiles provide a fast and easy way to render positions and encode filterable metadata.
Entity endpoints provide additional metadata which is not present in the tiles.

# URL

- Root endpoint for metadata: https://graph.mapillary.com
- Root endpoint for vector tiles: https://tiles.mapillary.com
  - Coverage tiles: https://tiles.mapillary.com/maps/vtp/mly1_public/2/{z}/{x}/{y}
  - Computed coverage tiles: https://tiles.mapillary.com/maps/vtp/mly1_computed_public/2/{z}/{x}/{y}
  - Map features (points) tiles: https://tiles.mapillary.com/maps/vtp/mly_map_feature_point/2/{z}/{x}/{y}
  - Map features (traffic signs) tiles: https://tiles.mapillary.com/maps/vtp/mly_map_feature_traffic_sign/2/{z}/{x}/{y}

# Embed images

Add Mapillary images to a web site or app with the Mapillary embed in an iframe.

The iframe tag below would show a 640x480px image with ID 2556542296.

```html
<iframe 
  src="https://www.mysite.example/embed?image_key=2556542296&style=photo" 
  height="480" 
  width="640"  
  frameborder="0">
</iframe>
```

The URL has the following format:

https://www.mysite.example/embed?image_key=IMAGE_ID&style=STYLE

Parameters:

- **image_key** (required) - Mapillary image ID ;
- **style** (optional) - determines the layout of the embed. Options are **photo** (only the image viewer is visible) ;
- **split** (image and map are visible, equal size) or **classic** (image and map are shown and visible, user can switch focus between map and image).

# Authentication

All requests against https://graph.mapillary.com and https://tiles.mapillary.com must be authorized. They require a client or user access tokens. Tokens can be sent in two ways

- using **?access_token=XXX** query parameters. This is a preferred method for interacting with vector tiles. Using this method is STRONGLY discouraged for sending user access tokens ;
- using a header such as **Authorization: OAuth XXX**, where XXX is the token obtained either through the OAuth flow that your application implements or a client token from https://mapillary.com/dashboard/developers.

This last method works for the Entity API calls.

## OAuth 2.0

### Authorization Code Flow

Mapillary's authentication endpoints follow the standard **OAuth 2.0 authorization code** flow, as per specification.

1. The app initializes the authorization request by opening the authorization URL in the user’s browser. See the **Authorization** endpoint below for how to construct the URL ;
2. The authentication page asks the user to authorize the app ;
3. When the app is authorized, the authentication page will redirect the user to the app’s callback URL with an authorization code ;
4. The app calls the token endpoint with the authorization code to exchange for the user access token. See the **Token Exchange** endpoint.

### Authorization

Endpoint: GET https://www.mapillary.com/connect

URL Parameters:

1. **client_id - int**: app's client ID ;
2. **redirect_uri - string** (optional): must be same as the callback URL of the app ;
3. **response_type - string** (optional): must be "code" ;
4. **scope - string** (optional): scope of the authorization. There are 3 possible scopes: read, write, or upload. The parameter must be subset of the app's scopes ;
5. **state - string** (optional): an arbitrary string that will be redirected back to the app.

Responses:

When successful, the user’s browser is redirected back to the app's configured callback URL with the following query parameters:

1. **code - string**: the authorization code. It's a temporary code that the app will exchange for the user access token later ;
2. **state - string** (optional): the state value passed from the authorization URL.

In case of an error, the following parameters will be redirected:

- **error - string**: the error code. Possible error codes:
  - user_denied: the user denied the authorization ;
  - invalid_request: user session is invalid or expired ;
  - invalid_scope: the requested scopes is out of the app's scopes ;
  - server_error: internal error.
- **state - string** (optional): The state value passed from the authorization URL.

Examples:

Assume the app's callback URL is "https://myapp.com/callback". Open the following authorization link:

https://www.mapillary.com/connect?client_id=12345&state=foo

After the user authorizes the app, the app will receive:

https://myapp.com/callback?state=foo&code=LONG_AUTHORIZATION_CODE

If the user denied the app, the app will receive:

https://myapp.com/callback?error=user_denied

### Token Exchange

Endpoint: **POST https://graph.mapillary.com/token**

This endpoint exchanges an authorization code or a refresh token for the user access token. Authorization is required, and the OAuth token must be the app's client secret.

Parameters:

1. **grant_type** - string: specify the type of the token. Either "authorization_code" or "refresh_token" ;
2. **client_id** - int: the app's client ID ;
3. **refresh_token** - string (optional): the current user access token can be used as the refresh token here (see **Refresh Flow**). When provided, grant_type must be "refresh_token" ;
4. **code** - string (optional): the temporary authorization code from the authorization redirection. When provided, grant_type must be "authorization_code" ;
5. **redirect_uri** - url (optional): the app's callback URL.

Responses:
When successful, the following properties will be returned in the JSON payload

1. **access_token** - string: The user access token.
2. **expires_in** - int: In how many seconds the access_token will be expired.
3. **token_type** - string: Must be "bearer".

Examples:

```json
curl https://graph.mapillary.com/token \
    -H "Content-Type: application/json" \
    -H "Authorization: OAuth CLIENT_SECRET" \
    -d'
{
    "grant_type": "authorization_code",
    "code": "LONG_AUTHORIZATION_CODE",
    "client_id": 1234
}'

{
  "access_token": "USER_ACCESS_TOKEN",
  "expires_in": 9000,
  "token_type": "bearer"
}
```

### Refresh Flow

Since the user access token returned from the Token Exchange endpoint will expire at some time, we need to refresh the token before it expires.

The recommended refresh flow:

1. When the app receives an user access token, remember (access_token, request_timestamp + expires_in)
2. Before calling Mapillary APIs with the user access token, check if the token is going to expire, i.e. check if request_timestamp + expires_in is less than currernt timestamp.
3. If it's going to expire, send a POST request to the **Token Exchange** endpoint with the user access token as the refresh token. See the examples below.

Examples:

```json
curl https://graph.mapillary.com/token \
    -H "Content-Type: application/json" \
    -H "Authorization: OAuth CLIENT_SECERT" \
    -d'
{
    "grant_type": "refresh_token",
    "refresh_token": "USER_ACCESS_TOKEN",
    "client_id": 1234
}'

{
  "access_token": "NEW_USER_ACCESS_TOKEN",
  "expires_in": 9000,
  "token_type": "bearer"
}
```
