# friend-oauth2

friend-oauth2 is an oauth2 workflow for [Friend][1].  It is an implementation of the [Authorization Code Grant](http://tools.ietf.org/html/rfc6749#section-4.1) type as described in the [RFC][3].

[Working examples][2] have been implemented for [app.net's OAuth2](https://github.com/appdotnet/api-spec/blob/master/auth.md), [Facebook's server-side authentication](https://developers.facebook.com/docs/authentication/server-side/), and [Github's OAuth2](http://developer.github.com/v3/oauth/).


## Installation

```clojure
[friend-oauth2 "0.0.4"]
```

Obviously requires [Friend][1].

## Documentation

For now, the best reference is the [Friend-OAuth2 examples][2]. Also please refer to the [Friend README][1].

Check out the ring-app handlers in the examples for some examples of how authentication and authorization routes are set up per Friend's config.

### Configuring your handler.

(See the one of the [example handlers][2] (appdotnet_handler.clj, facebook_handler.clj or github_handler.clj) for working examples.)

1. `client-config` holds the basic information which changes from app-to-app regardless of the provider: client-id, client-secret, and the applications callback url.

```clojure
(def client-config
  {:client-id "my-client-id-from-OAuth2-provider"
   :client-secret "my-client-secret-from-OAuth2-provider"
   :callback {:domain "http://my-domain.com" :path "/callback"}})
```

2. In the uri-config, the `authentication-uri` map holds the provider-specific configuration for the initial redirect to the OAuth2 provider (the user-facing GET request).  Please note, a base64 random ["state" parameter](http://tools.ietf.org/html/rfc6749#section-4.1.1) for [CSRF-protection](http://tools.ietf.org/html/rfc6749#section-10.12) is automatically set and passed to the provider by default.

```clojure
(def uri-config
  {:authentication-uri {:url "https://alpha.app.net/oauth/authenticate"
                       :query {:client_id (:client-id client-config)
                               :response_type "code"
                               :redirect_uri (oauth2/format-config-uri client-config)
                               :scope "stream,email"}}

   :access-token-uri { ;; ...

```


3. Again in the uri-config, the `access-token-uri` map holds the provider-specific configuration for the access_token request, after the code is returned from the previous redirect (a server-to-server POST request).

```clojure
(def uri-config
  {:authentication-uri { ;; ...

   :access-token-uri {:url "https://alpha.app.net/oauth/access_token"
                      :query {:client_id (:client-id client-config)
                              :client_secret (:client-secret client-config)
                              :grant_type "authorization_code"
                              :redirect_uri (oauth2/format-config-uri client-config)}}})
```

4. `access-token-parsefn` is a provider-specific function which parses the access_token response and returns just the access_token. If your OAuth2 provider does not follow the RFC (http://tools.ietf.org/html/rfc6749#section-5.1) then you can pass in a custom function to parse the access-token response.  See the [Facebook and Github examples][2] for reference.

```clojure
(defn access-token-parsefn
  [response]
  (-> response
      :body
      ring.util.codec/form-decode
      (get "access_token")))
```

5. config-auth is passed to Friend for role verification (for now...to-do: better way to handle this?)

```clojure
     :workflows [(friend-oauth2/workflow
                  {:client-config client-config
                   :uri-config uri-config
                   :config-auth {:roles #{::user}}})]})))
```


## To-do:

* Handle refresh tokens (http://tools.ietf.org/html/rfc6749#section-1.5).
* Handle exceptions/errors after redirect and access_token request (http://tools.ietf.org/html/rfc6749#section-7.2).
* Handle token types (http://tools.ietf.org/html/rfc6749#section-7.1).
* Handle CSRF state match failure some way other than redirecting (?).
* auth-map: should we be using the access-token as identity?  Are there any downsides to this, especially in terms of security?
* Move client_id/client_secret to Authorization header (necessary? Good for security or immaterial? Does FB support this?) (http://tools.ietf.org/html/rfc6749#section-2.3.1)

## License

Distributed under the MIT License (http://dd.mit-license.org/)

[1]: https://github.com/cemerick/friend
[2]: https://github.com/ddellacosta/friend-oauth2-examples
[3]: http://tools.ietf.org/html/rfc6749