#API

> To use a given `access_token`, send it in the Authorization HTTP Header as follows:

```
Authorization: Bearer <access_token>
```

Presently, there are three APIs available:

- [Fetching your own profile and campaign info](#fetch-your-own-profile-and-campaign-info)
- [Paging through a list of pledges to you](#paging-through-a-list-of-pledges-to-you)
- [Fetching a patron's profile info](#fetching-a-patron-39-s-profile-info)

These APIs are accessed using an OAuth client `access_token` obtained from the [OAuth](#oauth) section. Please go there first if you do not yet have one.

When performing an API request, the information you are allowed to see is determined by which `access_token` you are using. Please be sure to select your `access_token` appropriately. For example, __if someone has granted your OAuth client access to their profile information, and you try to fetch it using your own access_token instead of the one created when they granted your client access, you will instead just get your own profile information.__

## Fetch your own profile and campaign info

```ruby
require 'patreon'

class OAuthController < ApplicationController
  def redirect
    oauth_client = Patreon::OAuth.new(client_id, client_secret)
    tokens = oauth_client.get_tokens(params[:code], redirect_uri)
    access_token = tokens['access_token']

    api_client = Patreon::API.new(access_token)
    campaigns_response = api_client.fetch_campaign_and_patrons()
    @campaigns = campaigns_response['data']
  end
end
```

```php
<?php

require_once('vendor/patreon/patreon/src/patreon.php');

use Patreon\API;
use Patreon\OAuth;

$client_id = null;      // Replace with your data
$client_secret = null;  // Replace with your data
$creator_id = null;     // Replace with your data

$oauth_client = new Patreon\OAuth($client_id, $client_secret);

// Replace http://localhost:5000/oauth/redirect with your own uri
$redirect_uri = "http://localhost:5000/oauth/redirect";
// Make sure that you're using this snippet as Step 2 of the OAuth flow: https://www.patreon.com/platform/documentation/oauth
// so that you have the 'code' query parameter.
$tokens = $oauth_client->get_tokens($_GET['code'], $redirect_uri);
$access_token = $tokens['access_token'];
$refresh_token = $tokens['refresh_token'];

$api_client = new Patreon\API($access_token);
$campaigns = $api_client-fetch_campaign_and_patrons();
```

```python
import patreon
from flask import request
...

client_id = None      # Replace with your data
client_secret = None  # Replace with your data
creator_id = None     # Replace with your data

@app.route('/oauth/redirect')
def oauth_redirect():
    oauth_client = patreon.OAuth(client_id, client_secret)
    tokens = oauth_client.get_tokens(request.args.get('code'), '/oauth/redirect')
    access_token = tokens['access_token']

    api_client = patreon.API(access_token)
    user_response = api_client.fetch_user()
    user = user_response['data']
    included = user_response.get('included')
    if included:
        pledge = next((obj for obj in included
            if obj['type'] == 'pledge' and obj['relationships']['creator']['data']['id'] == creator_id), None)
        campaign = next((obj for obj in included
            if obj['type'] == 'campaign' and obj['relationships']['creator']['data']['id'] == creator_id), None)
    else:
        pledge = None
        campaign = None
```

```shell
curl --request GET \
  --url https://www.patreon.com/api/oauth2/api/current_user/campaigns \
  --header 'Authorization: Bearer <access_token>'
```

```javascript
import url from 'url'
import patreonAPI, { oauth as patreonOAuth } from 'patreon'

const CLIENT_ID = 'pppp'
const CLIENT_SECRET = 'pppp'
const patreonOAuthClient = patreonOAuth(CLIENT_ID, CLIENT_SECRET)

const redirectURL = 'http://mypatreonapp.com/oauth/redirect'

function handleOAuthRedirectRequest(request, response) {
    const oauthGrantCode = url.parse(request.url, true).query.code

    patreonOAuthClient.getTokens(oauthGrantCode, redirectURL, (tokensError, { access_token }) => {
        const patreonAPIClient = patreonAPI(access_token)

        patreonAPIClient(`/current_user`, (currentUserError, apiResponse) => {
            if (currentUserError) {
                console.error(currentUserError)
                response.end(currentUserError)
            }

            response.end(apiResponse);
        })
    })
})
```

```java
import com.patreon.OAuth;
import com.patreon.API;
import org.json.JSONObject;
import org.json.JSONArray;
...

String clientID = null;        // Replace with your data
String clientSecret = null;    // Replace with your data
String creatorID = null;       // Replace with your data
String redirectURI = null;     // Replace with your data
String code = null;            // get from inbound HTTP request

OAuth oauthClient = new OAuth(clientID, clientSecret);
JSONObject tokens = oauthClient.getTokens(code, redirectURI);
String accessToken = tokens.getString("access_token");

API apiClient = new API(accessToken);
JSONObject userResponse = apiClient.fetchUser();
JSONObject user = userResponse.getJSONObject("data");
JSONArray included = userResponse.getJSONArray("included");
JSONObject pledge = null;
JSONObject campaign = null;
if (included != null) {
   for (int i = 0; i < included.length(); i++) {
       JSONObject object = included.getJSONObject(i);
       if (object.getString("type").equals("pledge") && object.getJSONObject("relationships").getJSONObject("creator").getJSONObject("data").getString("id").equals(creatorID)) {
           pledge = object;
           break;
       }
   }
   for (int i = 0; i < included.length(); i++) {
       JSONObject object = included.getJSONObject(i);
       if (object.getString("type").equals("campaign") && object.getJSONObject("relationships").getJSONObject("creator").getJSONObject("data").getString("id").equals(creatorID)) {
           campaign = object;
           break;
       }
   }
}

   // use the user, pledge, and campaign objects as you desire
```

> Response:

```json
{
  "data": [{
    "attributes": {
      "created_at": "2017-10-20T21:39:01+00:00",
      "creation_count": 0,
      "creation_name": "Documentation",
      "discord_server_id": null,
      "display_patron_goals": false,
      "earnings_visibility": "public",
      "image_small_url": null,
      "image_url": null,
      "is_charged_immediately": false,
      "is_monthly": false,
      "is_nsfw": false,
      "is_plural": false,
      "main_video_embed": null,
      "main_video_url": null,
      "one_liner": null,
      "outstanding_payment_amount_cents": 0,
      "patron_count": 0,
      "pay_per_name": null,
      "pledge_sum": 0,
      "pledge_url": "/bePatron?c=0000000",
      "published_at": "2017-10-20T21:49:31+00:00",
      "summary": null,
      "thanks_embed": null,
      "thanks_msg": null,
      "thanks_video_url": null
    },
    "id": "0000000",
    "relationships": {
      "creator": {
        "data": {
          "id": "1111111",
          "type": "user"
        },
        "links": {
          "related": "https://www.patreon.com/api/user/1111111"
        }
      },
      "goals": {
        "data": []
      },
      "rewards": {
        "data": [{
            "id": "-1",
            "type": "reward"
          },
          {
            "id": "0",
            "type": "reward"
          }
        ]
      }
    },
    "type": "campaign"
  }],
  "included": [{
      "attributes": {
        "about": null,
        "created": "2017-10-20T21:36:23+00:00",
        "discord_id": null,
        "email": "corgi@patreon.com",
        "facebook": null,
        "facebook_id": null,
        "first_name": "Corgi",
        "full_name": "Corgi The Dev",
        "gender": 0,
        "has_password": true,
        "image_url": "https://c8.patreon.com/2/400/1111111",
        "is_deleted": false,
        "is_email_verified": false,
        "is_nuked": false,
        "is_suspended": false,
        "last_name": "The Dev",
        "social_connections": {
          "deviantart": null,
          "discord": null,
          "facebook": null,
          "spotify": null,
          "twitch": null,
          "twitter": null,
          "youtube": null
        },
        "thumb_url": "https://c8.patreon.com/2/100/1111111",
        "twitch": null,
        "twitter": null,
        "url": "https://www.patreon.com/drkthedev",
        "vanity": "drkthedev",
        "youtube": null
      },
      "id": "1111111",
      "relationships": {
        "campaign": {
          "data": {
            "id": "0000000",
            "type": "campaign"
          },
          "links": {
            "related": "https://www.patreon.com/api/campaigns/0000000"
          }
        }
      },
      "type": "user"
    },
    {
      "attributes": {
        "amount": 0,
        "amount_cents": 0,
        "created_at": null,
        "description": "Everyone",
        "id": "-1",
        "remaining": 0,
        "requires_shipping": false,
        "type": "reward",
        "url": null,
        "user_limit": null
      },
      "id": "-1",
      "relationships": {
        "creator": {
          "data": {
            "id": "1111111",
            "type": "user"
          },
          "links": {
            "related": "https://www.patreon.com/api/user/1111111"
          }
        }
      },
      "type": "reward"
    },
    {
      "attributes": {
        "amount": 1,
        "amount_cents": 1,
        "created_at": null,
        "description": "Patrons Only",
        "id": "0",
        "remaining": 0,
        "requires_shipping": false,
        "type": "reward",
        "url": null,
        "user_limit": null
      },
      "id": "0",
      "relationships": {
        "creator": {
          "data": {
            "id": "1111111",
            "type": "user"
          },
          "links": {
            "related": "https://www.patreon.com/api/user/1111111"
          }
        }
      },
      "type": "reward"
    }
  ]
}
```

This endpoint returns a JSON representation of the user's campaign, including its rewards and goals, and the pledges to it. If there are more than twenty pledges to the campaign, the first twenty will be returned, along with a link to the next page of pledges.

### HTTP Request

`GET https://www.patreon.com/api/oauth2/api/current_user/campaigns`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
includes | `rewards,creator,goals,pledges` | You can pass this `rewards`, `creator`, `goals`, or `pledges`

<aside class="success">
Remember — you must pass the correct <code>access_token</code> from the user.
</aside>

## Paging through a list of pledges to you

<!--  TODO: Make this code actual Ruby-->

```ruby
api_client = patreon.API(patron_access_token)
patrons_page = api_client.fetch_page_of_pledges(campaign_id, 10)
next_cursor = api_client.extract_cursor(patrons_page)
second_patrons_page = api_client.fetch_page_of_pledges(campaign_id, 10, cursor=next_cursor)
```

```python
api_client = patreon.API(patron_access_token)
patrons_page = api_client.fetch_page_of_pledges(campaign_id, 10)
next_cursor = api_client.extract_cursor(patrons_page)
second_patrons_page = api_client.fetch_page_of_pledges(campaign_id, 10, cursor=next_cursor)
```
```java
// TODO: Needs a code example of pagination
```
```shell
curl --request GET \
  --url https://www.patreon.com/api/oauth2/api/campaigns/<campaign_id>/pledges \
  --header 'Authorization: Bearer <access_token>
```

```javascript
// TODO: Add paginated example
```

```php
<?php
require_once('vendor/patreon/patreon/src/patreon.php');
use Patreon\API;
use Patreon\OAuth;
$access_token = **Your access token**;
$api_client = new Patreon\API($access_token);
// get page after page of pledge data
$campaign_id = $campaign_response['data'][0]['id'];
$cursor = null;
while (true) {
    $pledges_response = $api_client->fetch_page_of_pledges($campaign_id, 25, $cursor);
    // get all the users in an easy-to-lookup way
    $user_data = [];
    foreach ($pledges_response['included'] as $included_data) {
        if ($included_data['type'] == 'user') {
            $user_data[$included_data['id']] = $included_data;
        }
    }
    // loop over the pledges to get e.g. their amount and user name
    foreach ($pledges_response['data'] as $pledge_data) {
        $pledge_amount = $pledge_data['attributes']['amount_cents'];
        $patron_id = $pledge_data['relationships']['patron']['data']['id'];
        $patron_full_name = $user_data[$patron_id]['attributes']['full_name'];
        echo $patron_full_name . " is pledging " . $pledge_amount . " cents.\n";
    }
    // get the link to the next page of pledges
    $next_link = $pledges_response['links']['next'];
    if (!$next_link) {
        // if there's no next page, we're done!
        break;
    }
    // otherwise, parse out the cursor param
    $next_query_params = explode("?", $next_link)[1];
    parse_str($next_query_params, $parsed_next_query_params);
    $cursor = $parsed_next_query_params['page']['cursor'];
}
echo "Done!";
?>

```
> Response:

```json
{
  "data": [],
  "links": {
    "first": "https://www.patreon.com/api/oauth2/api/campaigns/1111111/pledges?page%5Bcount%5D=10&sort=created"
  },
  "meta": {
    "count": 0
  }
}
```

This endpoint returns a JSON list of pledges to the provided `campaign_id`. They are sorted by the date the pledge was made, and provide relationship references to the users who made each respective pledge.

The API response will also contain a `links` field which may be used to fetch the next page of pledges, or go back to the first page.

<aside class="notice">
When you made a creator page to gain API access, behind the scenes a <a href="#campaign">campaign resource</a> was created. You can access this resource as described in <a href="#fetch-your-own-profile-and-campaign-info">Fetching your own profile and campaign info</a> after authenticating via OAuth with your creator account to gain your `campaign_id`.
</aside>

### HTTP Request

`GET https://www.patreon.com/api/oauth2/api/campaigns/<campaign_id>/pledges`


### Paging

<aside class="success">
Remember — you must pass the correct <code>access_token</code> from the user.
</aside>

You may only fetch your own list of pledges. If you attempt to fetch another creator's pledge list, the API call will return an HTTP 403. If you would like to create an application which can manage many creator's campaigns, please contact us at [platform@patreon.com](mailto:platform@patreon.com).

## Fetching a patron's profile info
```ruby
require 'patreon'

class OAuthController < ApplicationController
  def redirect
    oauth_client = Patreon::OAuth.new(client_id, client_secret)
    tokens = oauth_client.get_tokens(params[:code], redirect_uri)
    access_token = tokens['access_token']

    api_client = Patreon::API.new(access_token)
    user_response = api_client.fetch_user()
    @user = user_response['data']
    included = user_response['included']
    if included
      @pledge = included.find {|obj| obj['type'] == 'pledge' && obj['relationships']['creator']['data']['id'] == creator_id}

    else
      @pledge = nil
    end
  end
end
```

```python
import patreon
from flask import request
...

client_id = None      # Replace with your data
client_secret = None  # Replace with your data
creator_id = None     # Replace with your data

@app.route('/oauth/redirect')
def oauth_redirect():
    oauth_client = patreon.OAuth(client_id, client_secret)
    tokens = oauth_client.get_tokens(request.args.get('code'), '/oauth/redirect')
    access_token = tokens['access_token']

    api_client = patreon.API(access_token)
    user_response = api_client.fetch_user()
    user = user_response['data']
    included = user_response.get('included')
    if included:
        pledge = next((obj for obj in included
            if obj['type'] == 'pledge' and obj['relationships']['creator']['data']['id'] == creator_id), None)
    else:
        pledge = None
```

```shell
curl --request GET \
  --url https://www.patreon.com/api/oauth2/api/current_user \
  --header 'authorization: Bearer <access_token>'
```

```php
<?php

require_once('vendor/patreon/patreon/src/patreon.php');

use Patreon\API;
use Patreon\OAuth;

$client_id = null;      // Replace with your data
$client_secret = null;  // Replace with your data
$creator_id = null;     // Replace with your data

$oauth_client = new Patreon\OAuth($client_id, $client_secret);

// Replace http://localhost:5000/oauth/redirect with your own uri
$redirect_uri = "http://localhost:5000/oauth/redirect";
// Make sure that you're using this snippet as Step 2 of the OAuth flow: https://www.patreon.com/platform/documentation/oauth
// so that you have the 'code' query parameter.
$tokens = $oauth_client->get_tokens($_GET['code'], $redirect_uri);
$access_token = $tokens['access_token'];
$refresh_token = $tokens['refresh_token'];

$api_client = new Patreon\API($access_token);
$patron_response = $api_client->fetch_user();
$patron = $patron_response['data'];
$included = $patron_response['included'];
$pledge = null;
if ($included != null) {
  foreach ($included as $obj) {
    if ($obj["type"] == "pledge" && $obj["relationships"]["creator"]["data"]["id"] == $creator_id) {
      $pledge = $obj;
      break;
    }
  }
}

/*
 $patron will have the authenticated user's user data, and
 $pledge will have their patronage data.
 Typically, you will save the relevant pieces of this data to your database,
 linked with their user account on your site,
 so your site can customize its experience based on their Patreon data.
 You will also want to save their $access_token and $refresh_token to your database,
 linked to their user account on your site,
 so that you can refresh their Patreon data on your own schedule.
 */

?>
```

```javascript
import url from 'url'
import patreonAPI, { oauth as patreonOAuth } from 'patreon'

const CLIENT_ID = 'pppp'
const CLIENT_SECRET = 'pppp'
const patreonOAuthClient = patreonOAuth(CLIENT_ID, CLIENT_SECRET)

const redirectURL = 'http://mypatreonapp.com/oauth/redirect'

function handleOAuthRedirectRequest(request, response) {
    const oauthGrantCode = url.parse(request.url, true).query.code

    patreonOAuthClient.getTokens(oauthGrantCode, redirectURL, (tokensError, { access_token }) => {
        const patreonAPIClient = patreonAPI(access_token)

        patreonAPIClient(`/current_user`, (currentUserError, apiResponse) => {
            if (currentUserError) {
                console.error(currentUserError)
                response.end(currentUserError)
            }

            response.end(apiResponse);
        })
    })
})
```

```java
import com.patreon.OAuth;
import com.patreon.API;
import org.json.JSONObject;
import org.json.JSONArray;
...

String clientID = null;        // Replace with your data
String clientSecret = null;    // Replace with your data
String creatorID = null;       // Replace with your data
String redirectURI = null;     // Replace with your data
String code = null;            // get from inbound HTTP request

OAuth oauthClient = new OAuth(clientID, clientSecret);
JSONObject tokens = oauthClient.getTokens(code, redirectURI);
String accessToken = tokens.getString("access_token");

API apiClient = new API(accessToken);
JSONObject userResponse = apiClient.fetchUser();
JSONObject user = userResponse.getJSONObject("data");
JSONArray included = userResponse.getJSONArray("included");
JSONObject pledge = null;
if (included != null) {
   for (int i = 0; i < included.length(); i++) {
       JSONObject object = included.getJSONObject(i);
       if (object.getString("type").equals("pledge") && object.getJSONObject("relationships").getJSONObject("creator").getJSONObject("data").getString("id").equals(creatorID)) {
           pledge = object;
           break;
       }
   }
}

   // use the user, pledge, and campaign objects as you desire
```

> Response:

```json
{
  "data": {
    "attributes": {
      "about": null,
      "created": "2017-10-20T21:36:23+00:00",
      "discord_id": null,
      "email": "corgi@example.com",
      "facebook": null,
      "facebook_id": null,
      "first_name": "Corgi",
      "full_name": "Corgi The Dev",
      "gender": 0,
      "has_password": true,
      "image_url": "https://c8.patreon.com/2/400/0000000",
      "is_deleted": false,
      "is_email_verified": false,
      "is_nuked": false,
      "is_suspended": false,
      "last_name": "The Dev",
      "social_connections": {
        "deviantart": null,
        "discord": null,
        "facebook": null,
        "spotify": null,
        "twitch": null,
        "twitter": null,
        "youtube": null
      },
      "thumb_url": "https://c8.patreon.com/2/100/0000000",
      "twitch": null,
      "twitter": null,
      "url": "https://www.patreon.com/corgithedev",
      "vanity": "corgithedev",
      "youtube": null
    },
    "id": "0000000",
    "relationships": {
      "pledges": {
        "data": []
      }
    },
    "type": "user"
  },
  "links": {
    "self": "https://www.patreon.com/api/user/0000000"
  }
}
```

This endpoint returns a JSON representation of the patron who granted your OAuth client an `access_token`. It is most typically used in the [OAuth "Log in with Patreon flow"](https://www.patreon.com/platform/documentation/oauth) to create or update the patron's account info in your application.

### HTTP Request

`GET https://www.patreon.com/api/oauth2/api/current_user`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
includes | `rewards,creator,goals,pledge` | You can pass this `rewards`, `creator`, `goals`, or `pledge`

<aside class="success">
Remember — you must pass the correct <code>access_token</code> from the user.
</aside>


## Advanced Usage

### Requesting specific data

```shell
curl --request GET \
  --url 'https://www.patreon.com/api/oauth2/api/campaigns/<campaign_id>/pledges?=&include=reward&fields%5Bpledge%5D=total_historical_amount_cents%2Cis_paused' \
  --header 'Authorization: Bearer <access_token>
```

To retrieve specific attributes or relationships other than the defaults, you can pass `fields` and `include` parameters respectively, each being comma-separated lists of attributes or resources.

<aside class="note">
For more information on requesting specific data, the <a href="http://jsonapi.org/format/#fetching">JSONAPI documentation</a> may be useful.
</aside>