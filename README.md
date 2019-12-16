# Twitstream to Post
This plugin implements the Twitter Real-Time Streaming API as described at https://developer.twitter.com/en/docs/tweets/filter-realtime/overview, in particular the `https://stream.twitter.com/1.1/statuses/filter.json` endpoint.

The plugin provides Settings page for wp-admin so user can authenticate the API using oAuth and manage filters. **API Filters** can be `handle` or `term`, and are combined to comma-delimited entries in the API call. **Tweet Filters** are applied to the tweets arriving on the stream connection from the API. **Post Handling** defines terms which the user wishes to include or exclude, and post types for the resulting Tweets to post to along with whether to save the post as Draft or Published.

## Filters
Filters are applied as parameters in the API call. They are also applied to the incoming Tweet stream to discard most messages and create several custom post types with the resulting Tweets.

**Example of Filter applied to the API call**
User enters the Twitter handles @NWSTornado, @NWSSevereThunderstorm, and @NWSFlashFlood.
A call is made to [GET users/lookup](https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-users-lookup) to obtain the user ID's, as the streaming API does not suport user handles.
The generated API call call would be similar to<br />`https://stream.twitter.com/1.1/statuses/filter.json?follow=######,######,######`

**Example of Filter allied to the incoming Tweets**
User enters the term " KS " (including the spaces) and selects "Process" as the Filter Command. Therefore the plugin will discard any Tweet from the above users not containing the search term In this example, weather watches and warnings for US states other than Kansas are dropped.

**Example of output specificaiton**
User enters Custom Post type name `watch-warn` and selects "Publish" as the post status. Therefore, Tweets matching the filter will post to the `watch-warn` CPT, with the Tweet image as the post featured image and the Tweet content (plus the URL to the original Tweet) as the post content, and publishing the post immediately.

**Case #2**
User enters the hashtag `#kswx#` as the filter to apply on the API call. Call is made as <br />`https://stream.twitter.com/1.1/statuses/filter.json?track=kswx`
User enters the term "KAKENews" and selects "Discard" as the Filter Command. Therefore the plugin will discard any tweets containing `KAKENews` and keep all the rest of the `kswx` tagged tweet flow.
User specifies CPT `kswx` and "Draft" as the post status. Tweet image is set as post featured image, Tweet content + Tweet URL is set as post content. Post is made in "Draft" status.


### The API Call
All userIDs and terms are concatenated into a comma-delimited set of terms supplied to the API. <br />`https://stream.twitter.com/1.1/statuses/filter.json?track=term,term,term&follow=######,######,######`

The API is called once. If the user edits the filters applied to the API, the current conneciton is terminated and a new one is made with the updated parameters. A cron task checks once a minute to ensure the streaming connection is still open. 

Tweets matching the specifications of the call (which are treated as OR conditions) are returned in a real-time stream. The plugin does NOT poll for data. 

On arrival of a tweet, a discard or process decision is based made on the Output Specifications entered by the user. These are applied in the order listed in the admin; drag and drop should be used in the admin to reorder the terms. The first matched term breaks out of the matching. 

Tweet data from the step above is then posted to the CPT defined by the user in the admin. Post status is also set as specified by the user.
