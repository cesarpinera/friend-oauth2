# Changelog

## 0.0.3 -> 0.0.4

* [Added CSRF protection](https://github.com/ddellacosta/friend-oauth2/issues/6).
* Refactored/restructured tests.
* Refactored to use more idiomatic clojure.
* [Uses ring.util.request's path-info to safely extract path info](https://github.com/ddellacosta/friend-oauth2/issues/2).
* [Uses default login-uri from config](https://github.com/ddellacosta/friend-oauth2/issues/3).

## 0.0.2 -> 0.0.3

* Nothing much, just updated versions of dependencies, and ensured tests are still passing.

## 0.0.1 -> 0.0.2

* Added tests! Refactored!
* A helper function has been added (`format-config-uri`) to configure the redirect url in the config.
* :redirect-uri in the uri-config has been renamed to :authentication-uri, as it more closely matches the RFC (and it actually makes sense)
* The access-token-parsefn functionality has been tweaked.  If the access-token is returned as defined in the spec (http://tools.ietf.org/html/draft-ietf-oauth-v2-31#section-5.1, as "application/json"), then it will automatically handle that.  Otherwise you can still pass in the access-token-parsefn to override, and it will use that.  See the [Facebook and Github examples][2] for reference.  **Note that this function also now takes the entire response, rather than just the body.**
