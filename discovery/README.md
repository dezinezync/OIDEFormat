## Discovery of Public keys

Various mechanisms for discovery of the public key from a provider are discussed below. Each mechanism is independantly usable or all can be implemented to provide optimum support.  

These mechanisms are listed in the order of preference by the spec. Do note that the order can change indeterminately until this spec reaches a stable state. 

## Mechanisms
### 1. HTTPS
The provider exposes their public key over HTTPS by a publicly known address. An example of the same:

```
GET /oide/public.pem HTTP/1.1
Accept: application/x-pem-file
Host: example.org
User-Agent: UA/String/Of/Your/Application
```

`example.org` should make the above information publicly known. However, it can also choose to only allow authenticated requests for the same using whichever auth mechanism it prefers. See [1]().

### 2. x-callback-url
For iOS apps, it's recommended to use the `x-callback-url` spec as embedding HTTP servers in your app isn't recommended for this purpose. This is also useful if yours is a mobile only application. 

```
providerapp://x-callback-url/oidepubkey?x-source=consumerApp&x-success=consumerapp://x-callback-url/acceptoidepubkey&x-error=sourceapp://x-callback-url/error
```

1. All providers implementing this mechanism is recommended to use the `/oidepubkey` path only. 

2. The Provider is recommended to ensure that the source key is always provided and is valid. They provider can use this information to look up it's Whitelist of trusted consumerapps (or blacklist of malicious ones). 

3. If the request is valid, optionally call the success callback url with the path to the public key stored in a publically accessible container. 

Please refer to the **X-Callback-URL* [Spec](http://x-callback-url.com/specifications/) for more info.
