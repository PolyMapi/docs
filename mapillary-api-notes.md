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

# Vector tiles

Vector tiles provide an easy way to visualize vast amounts of data. Mapillary APIs are heavily based on vector tiles to provide the developers with flexibility to programmatically interact with the data they contain in custom ways. Vector tiles support filtering and querying rendered features. Mapillary vector tiles follow the [Mapbox tile specification](https://docs.mapbox.com/vector-tiles/specification/).

## Coverage tiles

Endpoint: https://tiles.mapillary.com/maps/vtp/mly1_public/2/{z}/{x}/{y}?access_token=XXX

Contain positions of images and sequences with original geometries (not computed)

- layer name: overview
  - zoom: 0 - 5 (inclusive)
  - geometry: Point
  - data source: images
  - properties
    - **captured_at**, int, timestamp in ms since epoch ;
    - **id**, int, ID of the image ;
    - **sequence_id**, string, ID of the sequence this image belongs to ;
    - **is_pano**, bool, if it is a panoramic image.

- layer name: sequence
  - zoom: 6-14 (inclusive)
  - geometry: LineString
  - data source: images captured in a single collection, sorted by captured_at
  - properties
    - **captured_at**, int, timestamp in ms since epoch ;
    - **id**, string, ID of the sequence (the legacy sequence key) ;
    - **image_id**, int, ID of the "best" (first) image representing the sequence ;
    - **organization_id** (optional), int, ID of the organization this image belongs to ;
    - **is_pano**, bool, if it is a panoramic sequence.

- layer name: image
  - zoom: 14
  - geometry: Point
  - data source: images
  - properties
    - **captured_at**, int, timestamp in ms since epoch ;
    - **compass_angle**, int, the compass angle of the image ;
    - **id**, int, ID of the image ;
    - **sequence_id**, string, ID of the sequence this image belongs to ;
    - **organization_id** (optional), int, ID of the organization this image belongs to ;
    - **is_pano**, bool, if it is a panoramic image.

## Coverage tiles (computed)

Endpoint: https://tiles.mapillary.com/maps/vtp/mly1_computed_public/2/{z}/{x}/{y}?access_token=XXX

Contain positions of images and sequences with computed geometries (not original).

The tile metadata is exactly the same as **Coverage tiles**.

## Map feature tiles, points

Endpoint: https://tiles.mapillary.com/maps/vtp/mly_map_feature_point/2/{z}/{x}/{y}?access_token=XXX

These tiles represent positions of map features which are detected on the Mapillary platform and are not traffic signs.

- layer name: **point**
  - zoom: 14
  - geometry: Point
  - data source: map features
  - properties
    - **id**, int, ID of the map feature
    - **value**, string, name of the class which this object represent
    - **first_seen_at**, int, timestamp in ms since epoch, capture time of the earliest image on which the detection contribute to this map feature
    - **last_seen_at**, int, timestamp in ms since epoch, capture time of the latest image on which the detection contribute to this map feature

You can see the full list of available points [here](https://www.mapillary.com/developer/api-documentation/points).

## Map feature tiles, traffic signs

Endpoint: https://tiles.mapillary.com/maps/vtp/mly_map_feature_traffic_sign/2/{z}/{x}/{y}?access_token=XXX

These tiles represent positions of map features which are detected on the Mapillary platform and are traffic signs.

The tile metadata is exactly the same as **Map feature tiles, points**, except that the layer name is **traffic_sign**.

You can see the full list of available traffic signs [here](https://www.mapillary.com/developer/api-documentation/traffic-signs).

# Entities

Each API call requires specifying the fields of the Entity you're interested in explicitly. A sample image by ID request which returns the id and a computed geometry could look as below. For each entity available fields are listed in the relevant sections. All IDs are unique and the underlying metadata for each entity is accessible at **https://graph.mapillary.com/:id?fields=A,B,C**. The responses are uniform and always return a single object, unless otherwise stated (collection endpoints). All collection endpoint metadata are wrapped in a **{"data": [ {...}, ...]}** JSON object.

## Image

### Endpoints

- https://graph.mapillary.com/:image_id
- https://graph.mapillary.com/images

Represents the metadata of the image on the Mapillary platform with the following properties.

### Fields

- altitude - float, original altitude from camera Exif calculated from sea level.
- atomic_scale - float, scale of the SfM reconstruction around the image.
- camera_parameters - array of float, focal length, k1, k2. See OpenSfM for details.
- camera_type - enum, type of camera projection: "perspective", "fisheye", "equirectangular" (or equivalently "spherical")
- captured_at - timestamp, capture time.
- compass_angle - float, original compass angle of the image.
- computed_altitude - float, altitude after running image processing, from sea level.
- computed_compass_angle - float, compass angle after running image processing.
- computed_geometry - GeoJSON Point, location after running image processing.
- computed_rotation - enum, corrected orientation of the image. See OpenSfM for details.
- exif_orientation - enum, orientation of the camera as given by the Exif tag. Read more.
- geometry - GeoJSON Point geometry.
- height - int, height of the original image uploaded.
- thumb_256_url - string, URL to the 256px wide thumbnail.
- thumb_1024_url - string, URL to the 1024px wide thumbnail.
- thumb_2048_url - string, URL to the 2048px wide thumbnail.
- thumb_original_url - string, URL to the original wide thumbnail.
- merge_cc - int, ID of the connected component of images that were aligned together.
- mesh - { id: string, url: string } - URL to the mesh.
- sequence - string, ID of the sequence, which is a group of images captured in succession.
- sfm_cluster - { id: string, url: string } - URL to the point cloud in .ply format.
- width - int, width of the original image uploaded.
- detections - detection entity, detections from the image

Sample image request:

```html
GET https://graph.mapillary.com/$IMAGE_ID?access_token=$TOKEN&fields=id,computed_geometry,detections.value

{
  "id": "$IMAGE_ID",
  "computed_geometry": {
    "type": "Point",
    "coordinates": [
      0,
      0
    ]
  },
  "detections": {
    "data": [
      {
        "value": "information--general-directions--g1",
        "geometry": "GjgKBm1weS1vchIYEgIAABgDIhAJ4iDyJhq8AQAApAG7AQAPGgR0eXBlIgkKB3BvbHlnb24ogCB4AQ==",
        "id": "34567"
      }
    ]
  }
}
```

### Image search

Image search can be done using the **graph.mapillary.com/images** endpoint and specifying fields as stated above in addition to a filter:

- start_captured_at - string: filter images captured after. Specify in the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format. For example: "2022-08-16T16:42:46Z"
- end_captured_at - string: filter images captured before. Same format as "start_captured_at"
- bbox - float,float,float,float: filter images in the bounding box. Specify in this order: left, bottom, right, top (or minLon, minLat, maxLon, maxLat).
- limit - int: the maximum number of image entities to return. Max and default is 2000.
- organization_id - int: filter images contributed to the specified organization ID.
- sequence_ids - list of IDs: filter images in the given sequence IDs (separated by commas). For example /images?sequence_ids=123,456

Sample image search request:

```
GET https://graph.mapillary.com/images?access_token=$TOKEN&fields=id&bbox=12.967,55.597,13.008,55.607

{
   "data": [
      {
        "id": "$IMAGE_ID_1"
      },
      {
        "id": "$IMAGE_ID_2"
      }
    ]
}
```

## Map feature

These are objects with a location which have been derived from multiple detections in multiple images.

You can see the full list of available points [here](https://www.mapillary.com/developer/api-documentation/points) and traffic signs [here](https://www.mapillary.com/developer/api-documentation/traffic-signs).

### Endpoints

    https://graph.mapillary.com/:map_feature_id

### Fields

- aligned_direction - float, compass angle that the object faces
- first_seen_at - timestamp, timestamp of the least recent detection contributing to this feature.
- last_seen_at - timestamp, timestamp of the most recent detection contributing to this feature.
- object_value - string, what kind of map feature it is.
- object_type - string, either a traffic_sign or point.
- geometry - GeoJSON Point geometry.
- images - list of IDs, which images this map feature was derived from.

Sample map feature request:

```html
GET https://graph.mapillary.com/$MAP_FEATURE_ID?access_token=$TOKEN&fields=id,geometry,object_value

{
  "id": "$MAP_FEATURE_ID",
  "geometry": {
    "type": "Point",
    "coordinates": [0, 0]
  },
"object_value": "object--support--utility-pole"
}
```

### Map feature search

Map feature search can be done using the **graph.mapillary.com/map_features** endpoint and specifying fields as stated above in addition to a filter:

- start_first_seen_at - string, filter map features first seen in an image captured after. Specify in the ISO 8601 format. For example: "2022-08-16T16:42:46Z"
- end_first_seen_at - string, filter map features first seen in an image captured before. Same format as "start_first_seen_at"
- start_last_seen_at - string, filter map features last seen in an image captured after. Same format as "start_first_seen_at"
- end_last_seen_at - string, filter map features last seen in an image captured before. Same format as "start_first_seen_at"
- bbox - float,float,float,float, filter map features in the bounding box. Specify in this order: left, bottom, right, top (or minLon, minLat, maxLon, maxLat).
- object_values: string, filter map features of the specified object value. Wildcard is supported. May be multiple values separated by comma.
- limit - int, the maximum number of map feature entities to return. Max and default is 2000.

Sample map feature search request:

```html
GET https://graph.mapillary.com/map_features?access_token=$TOKEN&fields=id,object_value&bbox=12.967,55.597,13.008,55.607

{
   "data": [
      {
        "object_value": "object--sign--advertisement",
        "id": "$MAP_FEATURE_ID_1"
      },
      {
        "object_value": "object--support--utility-pole",
        "id": "$MAP_FEATUREE_ID_2"
      }
    ]
}
```

### Download map features

You can download points and traffic signs from the [Mapillary website](https://www.mapillary.com/app). The data is in the GeoJSON format.

To download data, zoom in on the map to the location you’re interested in and turn on the map data (points or traffic signs). Press the “Download” button to download all visible points or traffic signs in a JSON file.

## Detection

Represent an object detected in a single image. For convenience this version of the API serves detections as collections. They can be requested as a collection on the resource (e.g. image or map feature) they contribute or belong to.

### Endpoints

- https://graph.mapillary.com/:image_id/detections - detections in the image with ID image_id
- https://graph.mapillary.com/:map_feature_id/detections - detections contributing to the map feature with ID map_feature_id

### Fields

- created_at - timestamp, when was this detection created.
- geometry - string, base64 encoded polygon.
- image - object, image the detection belongs to.
- value - string, what kind of object the detection represents.

Sample detection request:

```html
GET https://graph.mapillary.com/$IMAGE_ID/detections?access_token=$TOKEN&fields=image,value,geometry

{
  "id": "$IMAGE_ID"
  "image": {
    "geometry": {
      "type": "Point",
      "coordinates": [0, 0]
  },
  "id": "$DETECTION_ID"
  },
  "value": "information--highway-interchange--g1",
  "geometry": "GjgKBm1weS1vchIYEgIAABgDIhAJnC3uFxroAwAA7AbnAwAPGgR0eXBlIgkKB3BvbHlnb24ogCB4AQ=="
}
```

### Segmentation

The geometry field of a detection contains the segmentation data for each detection, but it must be decoded in order to get normalized coordinates for use with MapillaryJS or any image.

#### Sample decoding

In Python, segmentation can be decoded according to the following example:

```python
import base64
import mapbox_vector_tile

base64_string = "GjgKBm1weS1vchIYEgIAABgDIhAJnC3uFxroAwAA7AbnAwAPGgR0eXBlIgkKB3BvbHlnb24ogCB4AQ=="

decoded_data = base64.decodebytes(base64_string.encode('utf-8'))

detection_geometry = mapbox_vector_tile.decode(decoded_data)

print(detection_geometry) 
```

The output of the decoding will not be normalized, but gives a series of coordinates:

```python
detection_geometry = {'mpy-or': {'extent': 4096, 'version': 2, 'features': [{'geometry': {'type': 'Polygon', 'coordinates': [[[2402, 2776], [2408, 2776], [2413, 2762], [2413, 2746], [2409, 2736], [2404, 2732], [2363, 2734], [2357, 2747], [2359, 2771], [2363, 2776], [2377, 2779], [2383, 2789], [2389, 2788], [2392, 2783], [2396, 2783], [2402, 2776]]]}, 'properties': {}, 'id': 1, 'type': 3}]}}
```

To normalize this, divide each x and y coordinate by the extent (4096). To get exact pixel coordinates, multiply the normalized x by the width of the image containing the detection, and the normalized y coordinate by the height, remembering that pixel coordinates start at the top left corner of the image when visualized. Retrieve the height and width by making an API request for the image key with the height and width fields.

### Detection search

Detections are contained in images, so searched according to image location and timestamps. Detection search can be done using the graph.mapillary.com/detections endpoint and specifying fields as stated above in addition to a filter:

- start_captured_at - string, filter detections from image captured after. Specify in the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format. For example: "2022-08-16T16:42:46Z"
- end_captured_at - string, filter detections from images captured before. Same format as "start_captured_at"
- bbox - float,float,float,float, filter detections from images in the bounding box. Specify in this order: left, bottom, right, top (or minLon, minLat, maxLon, maxLat).
- limit - int, the maximum number of detection entities to return. Max and default is 2000.
- values - string, filter detections by specified detection value. Wildcard is supported. May be multiple values separated by comma.

Sample detection search request:

```json
GET https://graph.mapillary.com/detections?access_token=$TOKEN&fields=value&bbox=12.967,55.597,13.008,55.607

{
   {
   "data": [
      {
         "value": "general--traffic-sign--g1",
         "id": "1203507933411555"
      },
      {
         "value": "object--support--utility-pole",
         "id": "1213804269051588"
      },
    ]
}
```

## Organization

Represents an organization which can own the imagery if users upload to it.

### Endpoints

- https://graph.mapillary.com/:organization_id

### Fields

- slug - short name, used in URLs.
- name - nice name.
- description - public description of the organization.

Sample organization request:

```html
GET https://graph.mapillary.com/$ORGANIZATION_ID?access_token=$TOKEN&fields=name

{
  "id": "$ORGANIZATION_ID"
  "name": "My Organization"
}
```

## Sequence

Represents a sequence of Image IDs ordered by capture time.

### Endpoints

- https://graph.mapillary.com/image_ids?sequence_id=XXX

### Fields

- sequence_id - ID of the sequence.

## Multiple Entities

Returns multiple entities based on their IDs.

### Endpoints

- https://graph.mapillary.com?ids=ID1,ID2,ID3

Parameters

- ids - A list of entity IDs separated by comma. The provided IDs must be in the same type (e.g. all image IDs, or all detection IDs)

## Rate Limits

Requests to **graph.mapillary.com** are limited to 50,000 per minute per app. Exceeding the limit your app will be throttled and users will receive "Application request limit reached" error.

Requests to **tiles.mapillary.com** are limited to 50,000 per day.

# Glossary

## Client access token

The client access token is a token required for all API calls, whether they're Entity API calls or vector tile requests. This token identifies the application used for generating it and enables making requests on behalf of it. If a hacker steals it, they would be able to make a request on behalf of your application!

## User access token

The user access token is a token obtained by the user when authentication through the OAuth2 flow. It authenticates both the user and the application it used. It is preferred to send user tokens as the authorization header (see Authentication section). This token is equivalent to a user password, if a hacker steals it, they'll be able to call the API on behalf of the user.

[Get Access Token](https://www.mapillary.com/dashboard/developers)

### Useful link

[PointsTraffic signs](https://www.mapillary.com/developer/api-documentation/points)
