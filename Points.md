subscribe/
String pub_key_id = params.get("pub_key_id"); // specification uses unique_key_id
String subscriber_id = params.get("subscriber_id");

this implementation uses authorized calls. headers must have the signing keys present, that means subscriber needs to have an entry in the registry before calling /subscribe API. or else it throws an error.

if 
    no signature passed ||
    verified flag is false ||
    signature does not match to the user ||
    decryption and verification fails
then
    returns error


If no subscribers are provided in the request (empty request payload):
    Checks if the role isn't already subscribed
    The OnSubscribe task is responsible for verifying a subscriber's identity and completing their subscription process
    It's triggered when a new subscriber is registered or when their status changes to "INITIATED"
    Sends a challenge to the subscriber's endpoint that must be decrypted and returned correctly
    Can handle verification of multiple participant keys
    Updates subscriber status to "SUBSCRIBED" after successful verification

    When It's Called:
        During initial subscriber registration
        When adding new keys
        When subscriber status changes to "INITIATED"
        When manually triggered through an empty payload /subscribe call

    subscribers send the decrypted challenge string as part of the sync response to on_subscribe call

else if subscribers are provided in the payload:
    each subscribe request can have many subscribers, for each subscriber
        if subscriber passes a key in request. if it is a new key or the key passed is not verified, it is considered a new key.
        set signing public key and enc public key in the key passed
        Cannot modify a verified registered key. Please create a new key
        each subscriber can have many domains, for each domain
            create a role
            populate role with subscriber_id, domain, type, NPid, url, and save it


on_subscribe/
When It's Used:
    When a new subscriber is registered
    When a subscriber's status changes to "INITIATED"
    When a new key is added
    When manually triggered through an empty payload /subscribe call

Registry sends below 3 parameters in the onSubscribe call. different than specification.
    input.put("subscriber_id", subscriber.getSubscriberId()); // not in specification
    input.put("pub_key_id", pk.getKeyId()); // not in specification, counterpart is unique_key_id
    input.put("challenge", encrypted);

numResponsesRemaining is set to require numbers of challenge need to be passed. (keys.size())
Registry sends a new challenge for each key (in a loop) for the participant which are not verified,  
    numResponsesRemaining is decremented for each passed challenge in the loop, for each key

if at the end of the loop, the numResponsesRemaining==0, keys are marked as verified or subscriber as subscribed.



