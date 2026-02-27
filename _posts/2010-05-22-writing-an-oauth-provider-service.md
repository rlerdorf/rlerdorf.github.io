---
date: '2010-05-22'
layout: post
permalink: /archives/55-Writing-an-OAuth-Provider-Service.html
title: Writing an OAuth Provider Service
---


Last year I showed how to use pecl/oauth to write a [Twitter OAuth
Consumer](http://toys.lerdorf.com/archives/50-Using-pecloauth-to-post-to-Twitter.html).
But what about writing the other end of that? What if you need to
provide OAuth access to an API for your site? How do you do it?  
  
Luckily John Jawed and Tjerk have put quite a bit of work into
pecl/oauth lately and we now have full provider support in the
extension. It's not documented yet at php.net/oauth, but there are some
examples in
[svn](http://svn.php.net/viewvc/pecl/oauth/trunk/examples/provider/). My
particular project was to hook an OAuth provider service into a large
existing Kohana-based codebase. After a couple of iterations this should
now be trivial for others to do with the current pecl/oauth extension.  
  
**Step 1 - Create a Consumer Key registration page**  
  
In order for an application to communicate with your service you assign
it what is essentially a user id and password. Except in OAuth terms
these are known as the Consumer Key (CK) and shared secret. Your main
job here is to make sure the CK and secret are unique and unguessable
which means you need a decent entropy source. On Linux you have
/dev/random, and the non-blocking potentially slightly weaker
/dev/urandom mechanism. You can check how much entropy is available
with:

    cat /proc/sys/kernel/random/entropy_avail 

In general something like the following should give you a decent random
string of characters that you can use for your CK and secret:

``` 
        function new_consumer_key() {
                $fp = fopen('/dev/urandom','rb');
                $entropy = fread($fp, 32);
                fclose($fp);
                // in case /dev/urandom is reusing entropy from its pool, let's add a bit more entropy
                $entropy .= uniqid(mt_rand(), true);
                $hash = sha1($entropy);  // sha1 gives us a 40-byte hash
                // The first 30 bytes should be plenty for the consumer_key
                // We use the last 10 for the shared secret
                return array(substr($hash,0,30),substr($hash,30,10));
        }
```

You can of course read more entropy and use a longer hash, like sha256
or whirlpool if you want longer keys.  
  
**Step 2 - The OAuth endpoints**  
  
It would probably be a good idea to skim the [OAuth
Spec](http://oauth.net/core/1.0a/) at this point. We need two main end
points for the OAuth sequence of requests. The first is called the
"request token" end point. In my case I named it
*/v1/oauth/request\_token*. And since I am sliding this into a
Kohana-based system, I wrote a controller
(application/controllers/v1.php) which contained this in the
constructor:  

``` 
    try {
    $this->provider = new OAuthProvider();
    $this->provider->consumerHandler(array($this,'lookupConsumer')); 
    $this->provider->timestampNonceHandler(array($this,'timestampNonceChecker'));
    $this->provider->tokenHandler(array($this,'tokenHandler'));
    $this->provider->setParam('kohana_uri', NULL);  // Ignore the kohana_uri parameter
    $this->provider->setRequestTokenPath('/v1/oauth/request_token');  // No token needed for this end point
    $this->provider->checkOAuthRequest();
    } catch (OAuthException $E) {
    echo OAuthProvider::reportProblem($E);
    $this->oauth_error = true;
    }
```

  
This makes the pecl/oauth extension do all the heavy lifting for us. We
register a couple of callback functions. *lookupConsumer* will look up
the CK and check if it is valid. *timestampNonceChecker* will check
whether the timestamp of the request is sane and falls within the window
of our Nonce checks. And this function will, of course, also check
whether the provided Nonce has been used already to prevent replay
attacks. And finally the *tokenHandler* callback will check whether a
request or access token is valid. The *setParam('kohana\_uri',NULL)*
call tells the extension to ignore the kohana\_uri parameter in the URLs
because this was injected by an nginx rewrite rule and wasn't part of
the original request. The *lookupConsumer* callback happens early on. My
version looks like this:

``` 
        public function lookupConsumer($provider) {
                $consumer = ORM::Factory("consumer", $provider->consumer_key);
                if($provider->consumer_key != $consumer->consumer_key) {
                        return OAUTH_CONSUMER_KEY_UNKNOWN;
                } else if($consumer->key_status != 0) {  // 0 is active, 1 is throttled, 2 is blacklisted
                        return OAUTH_CONSUMER_KEY_REFUSED;
                }
                $provider->consumer_secret = $consumer->secret;
                return OAUTH_OK;
        }
```

Just a simple lookup in the data model and if found, the shared secret
is attached so it can be used to check the signature of the request. So,
now we need to write code for the different stages of the OAuth request.
I have an oauth method with a switch:

``` 
        public function oauth($action=NULL) {
                if($this->oauth_error) return;

                switch($action) {
                case 'request_token':
                        $token = Token_Model::create($this->provider->consumer_key);
                        $token->save();
                        // Build response with the authorization URL users should be sent to
                        echo 'login_url=https://'.Kohana::config('config.site_domain').
                             '/session/authorize&oauth_token='.$token->tok.
                             '&oauth_token_secret='.$token->secret.
                             '&oauth_callback_confirmed=true';
                        break;
```

Remember, the CK and the signature of the request has been checked
already in the constructor, so by the time we get here, as long as
$this-\>oauth\_error is false, we can generate a request token and
respond with a urlencoded set of parameters that include the
unauthorized request token and secret we just generated along with a
login url that the 3rd-party application will redirect the user to in
order for them to authorize the request token to be used on their
behalf.  
  
**Step 3 - Authorizing the request token**  
  
Back to your regular web UI, you now need to add a landing page for
authorizing request tokens. Make sure the language on the page makes it
clear to the user that they are authorizing a 3rd-party application to
act on their behalf. It is a good idea to make them re-enter their
password and explicitly click a button to do this authorization.  
  
**Step 4 - The Access token**  
  
Once the user has authorized the request token, the 3rd-party app needs
to exchange the authorized request token for an access token. My switch
case for this looks something like this:

``` 
        case 'access_token':
                $access_token = Token_Model::create($this->provider->consumer_key, 1);
                $access_token->save();
                $this->token->state = 2;  // The request token is marked as 'used'
                $this->token->save();
                // Now we need to find the user who authorized this request token
                $utoken = ORM::factory('utoken', $this->token->tok);
                if(!$utoken->loaded) {
                        echo "oauth error - token rejected";
                        break;
                }
                // And swap out the authorized request token for the access token
                $new_utoken = Utoken_Model::create(
                                           array('token'            => $access_token->tok,
                                                    'user_id'         => $utoken->user_id,
                                                    'application_id'=> $utoken->application_id,
                                                    'access_type'   => $utoken->access_type));
                $new_utoken->save();
                $utoken->delete();
                echo "oauth_token={$access_token->tok}&oauth_token_secret={$access_token->secret}";
                break;
```

  
As you can see I have a couple of data models that deal with tokens
here. Token\_Model is the token itself while UToken\_Model links users
to tokens. A token is either a request token (type 0) or an access
token, (type 1) and they can be in various states. The *tokenHandler*
callback keeps track of whether a token is used correctly. It looks like
this:  

``` 
        public function tokenHandler($provider) {
                $this->token = ORM::Factory("token", $provider->token);
                if(!$this->token->loaded) {
                        return OAUTH_TOKEN_REJECTED;
                } else if($this->token->type==1 && $this->token->state==1) {
                        return OAUTH_TOKEN_REVOKED;
                } else if($this->token->type==0 && $this->token->state==2) {
                        return OAUTH_TOKEN_USED;
                } else if($this->token->type==0 && $this->token->verifier != $provider->verifier) {
                        return OAUTH_VERIFIER_INVALID;
                }
                $provider->token_secret = $this->token->secret;
                return OAUTH_OK;
        }
```

  
Once we have handed out an access token, the 3rd-party app can store
this and sign API requests with the associated shared secret and send
that signature along with the consumer key. As you can see in the above
token handler, there is a revocation check. At any point the user can go
in through the Web UI and revoke an access token for a specific
3rd-party application.  
  
**Step 5 - An actual API call**  
  
Now that we have completed the OAuth dance, we can write API end points.
Because all the oauth magic happens in our controller constructor, we
are left to just implement our own api logic in our api methods. Mine
look something like this:

``` 
        public function user($cmd, $arg1=NULL, $arg2=NULL) {
                if(!$this->checkAccess()) return;
                $res = array();
                switch($cmd) {
                case 'settings':
                        foreach($this->user->settings as $key=>$val ) {
                                $res[$key] = $val;
                        }
                        $this->jsonResult($res);
                        break;
```

The *checkAccess* function uses the access token to look up the user who
authorized it and sets $this-\>user the same way your login flow will
load the user once authenticated.  
  
For the most part, the new OAuth Provider support in pecl/oauth takes
the pain out of writing an OAuth Provider. It is smart about detecting
the URL of the end points. You only have to indicate what the
request\_token (or 2-legged) endpoint is called, and it takes it from
there. You can also pass the endpoint explicitly to *checkOAuthRequest*
at each stage of the OAuth dance, but then you have to separate the
steps more than I have here. I find this approach nice and compact and
it lets me focus on the API part. I also like that it supports the
[OAuth ProblemReporting](http://wiki.oauth.net/ProblemReporting)
extension. This makes it a bit easier to track down problems. And by
problems I mean signature mismatches. Everyone who has ever done
anything with OAuth has struggled with them, and you will too. With the
ProblemReporting extension you will get the raw string that the provider
used to generate the signature. You just have to match that against what
your consumer signed and you should be able to track down signature
problems quickly.  
  
Let me know if you do something interesting with the pecl/oauth
extension. We may still be missing some use cases and right now there is
decent momentum to fix things, so get in there and try it out.
