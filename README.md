SSTwitter
============================

This is a silverstripe module which allows for basic integration between Silverstripe & Twitter.

Upon installation a new tab will be added to 'Settings' in the CMS which will ask for your Twitter Consumer Key & Secret. You can get these by setting up an Application on Twitter: https://dev.twitter.com/

After saving those fields you will be given an 'Auto-fill' option which will take you through the Twitter authorization
process and fill in the fields automatically for you.


Usage
--------

**$TwitterConnectURL** (TwitterApp::connect_url())
This displays a link where a logged in user will be taken through the Twitter authentication process to connect their Twitter account.

**$TwitterDisconnectURL** (TwitterApp::disconnect_url())
This will disassociate the Twitter account from the Member.

**$TwitterLoginURL** (TwitterApp::login_url())
This will return a url whereby the user can login to their Silverstripe account through Twitter where previously connected.

	<a href="$TwitterConnectURL">Connect</a><br />
	<a href="$TwitterDisconnectURL">Disconnect</a><br />
	<% if TwitterLoginURL %>
		<a href="$TwitterLoginURL">Login</a>
	<% else %>
		Twitter Login is disabled.
	<% end_if %>


Extending
---------
SSTwitter uses [PHPTwitter](http://www.github.com/micmania1/phptwitter) for its Twitter Authentication which has a central Twitter->api() method which handles all API requests. This means you can easily harness its power to interact directly with Twitter.
Below is an example of how you would get the latest tweets for the account connected to SiteConfig.

In Page_Controller.php:

    public function LatestTweets($count = 3) {
    
    	$tweets = new ArrayList();
   
    	$siteConfig = SiteConfig::current_site_config();
    	$twitter = TwitterApp::curr();
    	$twitter->setAccess(new OauthToken($siteConfig->TwitterAccessToken, $siteConfig->TwitterAccessSecret));
    
    	// Get the latest Tweets
    	if($twitter->hasAccess()) {
    	    $result = $twitter->api("1.1/statuses/user_timeline.json", "GET", array(
    	        "screen_name" => $siteConfig->TwitterScreenName,
    	        "count" => (int) $count
    	    ));
    	    
    	    if($result->statusCode() == 200) {
    	    	$json = json_decode($result->body(), true);
    	    	if(count($json) > 0) {
    	    		foreach($json as $tweet) {
    	    			$tweets->push(array(
    	    				"Tweet" => $tweet['text'],
    	    				"Created" => $tweet['created_at'],
    	    				"Link" => "http://www.twitter.com/".rawurlencode($tweet['user']['screen_name'])."/status/".rawurlencode($tweet['id_str'])
    	    			));
    	    		}
    	    	}
    	    }
    	}
    	return $tweets;
    }


In Page.ss

    <% if LatestTweets %>
    	<ul>
    		<% loop LatestTweets %>
    			<li>$Tweet - <a href="$Link">$Created</a></li>
    		<% end_loop %>
    	</ul>
    <% end_if %>
    


