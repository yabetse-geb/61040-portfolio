# Pset 2: Composing Concepts

# Concepts for URL Shortening
1. **Contexts:** The contexts act as a namespace or a scope as strings within a context will be unique, but they may repeat across different contexts. In the URL shortening app the contexts are the shortURLBases. For example, tinyurl.com/home and tinyurl.com/About with "tinyurl.com" as the shortURLBase
2. **Storing used strings:** The NonceGeneration stores sets of used strings so that there may be a way to check generated strings to ensure new generated strings are unique. The set of used strings relates to the counter implementation as the length of the set of used strings is the same as the counter value.
3. **Words as nonces:** One advatnage for the user is that it is easier to remember the shortenings. One disadvantage of this scheme is that it there are much less
unique options for shortened URLs so users are more likely to find that their desired word/phrase is taken. To realize this idea you would need to modify the
NonceGeneration by adding a set of strings to the concept state, and modifying generate action to taking a string or concatenation of strings from this set of strings, still ensuring that it is a string that hasn't been returned before for that context.



# Synchronizations for URL Shortening
1. **Partial Matching**: This difference occurs because the generate's sync is responsible or creating a nonce and the generate action in NonceGeneration only requires the shortUrlBase as a context, and targetUrl is irrelevant for this action. However, for the register sync, it must creating the mapping of the shortUrlBase and targetUrl which is why it needs all the information, being the nonce, shortUrlBase, and targetUrl.
2. **Omitting names**: This convention isn't used in every case because the case in which it is used where the argument name and bound variable name are identical has no information lost when the argument name is not explicitly written. However, when the bound variable and argumenwt name differ it is helpful to explicitly differenciate which is the input being passed in, and what parameter is storing the passed in input.
3. **Inclusion of request**: The request action is included in the first two syncs because they are directly triggered/invoked by a user, while the third sync is triggered by an internal system action.
4. **Fixed Domain**: To implement this I would change the syncs by no longer taking the shortUrlBase argument for Request.shortenUrl actions in the when clauses as the domain is a fixed value. For the then clauses the value passed in to the shortUrlBase arguments would be the fixed domain as a string literal.
5. **Adding a sync**:
**sync** expireResource:
**when** ExpiringResource.expireResource ():(resource: shortUrl)
**then** UrlShortening.delete (shortUrl)

# Extending the design
## 1. Concepts:

**concept** Analytics

**purpose** allow users to see the count of accesses for a link

**principle** When a URL is is registered a number of accesses count for that url is set to 0. Then when a URL is accessed, that count keeping track of the number of accesses for that URL increments by 1, and when this analytic is requested the system reports the count to the user alone (the individual who registered the shortening)

**state**
- a set of ShortenedUrls with
    - shortenedUrl String
    - a accessCount Number

**actions**
- startTracking(shortUrl:String)
    - **requires** shortUrl does not exist in ShortenedUrls
    - **effect** creates a new record for shortUrl in ShortenedUrls with accessCount of 0
- viewAccessCount(shortUrl:String):(accessCount:Number)
    - **requires** shortUrl exists in ShortenedUrls
    - **effect** returns current accessCount
- increment (shortUrl: Stirng)
    - **requires** shortUrl exists in ShortenedUrls
    - **effect** increment accessCount for shortUrl by 1


**concept** LinkOwner [User]

**purpose** ensures only the creator of a shortened URL can view its analytics

**principle** after a user registers a short URL, they are its owner and can request the URL's analytics

**state** a set of Ownerships with
- shortUrl:String
- userId: User

**actions**
- assign(shortUrl: String, userId:User)
    - **requires** shortUrl exists in Ownerships
    - **effect** associates userId as the owner of shortUrl
- confirmOwnership(shortUrl: String, userId: User): (confirmation: Boolean)
    - **effect** returns true if userId is the owner of shortUrl, otherwise returns false

## 2. Syncs:

**sync** beginAnalytics
- **when**
    - Request.shortenUrl(userId, shortUrlBase, targetUrl)
    - UrlShortening.register(shortUrlSuffix, shortUrlBase, targetUrl): (shortUrl)
- **then**
    - Analytics.startTracking(shortUrl: shortUrl)
    - LinkOwner.assign(shortUrl: shortUrl, userId: userId)

**sync** recordAccess
- **when** UrlShortening.lookup(shortUrl): targetUrl
- **then** Analytics.increment(shortUrl)

**sync** viewAnalytics
- **when**
    - Request.viewAnalytics(shortUrl)
    - Ownership.verifyOwnership(shortUrl, userId): (confirmation)
- **where** confirmation=true
- **then** Analytics.getCount(shortUrl)


## 3. Feature request analysis:
1. Allowing users to choose their own short URLs:
    - modify UrlShortening.register to allow user-provided shortUrlSuffix
2. Using the “word as nonce” strategy to generate more memorable short URLs:
    - should not be implemented since collisions are more likely for shortened URLs
3. Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL:
    - The Analytics conept would have a modified state to include the targetUrl in addition to shortUrl. So, the state would become:
    - **state**:
        -  a set of ShortenedUrls
            - with a targetUrl:String
            - a totalAccessCount:Number
            - a set of ShortUrlRecords with
                - a shortUrl:String
                - an accessCount:Number
    - also the Analytics.startTracking action would need to take in the targetUrl as a parameter and set totalAccessCount to 0 along with the shortUrl accessCount. Therefore the beginAnalytics would need to pass in the targetUrl in the then clause when calling startTracking,
    - the Analytics.increment action would also need to take in a targetUrl parameter and incrememnt totalAccesscount along with the accessCount of the shortUrl. Therefore, the recordAccess sync will pass in the targetUrl as wellin the then clause.
    - create a new sync for that listens for this new kind of request (for instance Request.viewGroupedAnalytics) and a new action in the Analytics concept to get the total access count for the short urls of a particular target url (e.g getCountByTargetUrl(targetUrl) )this would be the added sync:
        - **sync** view GroupedAnalytics
            - **when** Request.viewGroupedAnalytics(targetUrl, userId:User)
            - **then** Analytics.getCountByTargetUrl(targetUrl)
4. Generate short URLs that are not easily guessed:
    - Create a new concept that is identical to NonceGeneration[Context] called SecureNonceGeneration, except it produces highly random nonces using a strong cryptographic method
5. Supporting reporting of analytics to creators of short URLs who have not registered as user.
:
    - This is an undesireable feature because if we want the analytics of short URLs to be viewable by creators we need a secure way to authenticate that creator when they return, and registering as a user provides this authentication.
