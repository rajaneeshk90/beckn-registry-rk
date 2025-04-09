register/
Why is this API used:
    Creates a new NetworkParticipant record
    Sets up initial NetworkRole with status "INITIATED". This status indicates the subscriber needs to complete verification
    Associates domains with the subscriber
    Validates domain existence
    Registers the subscriber's cryptographic keys
    Sets key validity periods
    Associates geographic regions with the subscriber

how does this API work:
    Reads the input payload
    Parses it as JSON
    Handles both single subscriber (JSONObject) and multiple subscribers (JSONArray)
    For each subscriber in the payload:
        create NetworkParticipant and populate its relevant fields
        create NetworkRole and populate its relevant fields
        create ParticipantKey and populate its relevant fields
    Returns single subscriber JSON for single registration
    Returns array of subscribers for multiple registrations

Response: A key Id is generated to identify "the initial keyset (signing and encryption keys)" and is handed over to the subscriber. This key Id is marked as verified.

register API returns a response in this format

```
{
    "subscriber_id": "example.com",
    "subscriber_url": "https://example.com",
    "status": "INITIATED",
    "domain": "example.com",
    "type": "BG",
    "signing_public_key": "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...",
    "encr_public_key": "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...",
    "pub_key_id": "key-123",
    "valid_from": "2024-01-01T00:00:00Z",
    "valid_to": "2025-01-01T00:00:00Z",
    "city": "Bangalore",
    "country": "India",
    "created": "2024-01-01T00:00:00Z",
    "updated": "2024-01-01T00:00:00Z"
}
```

Note: 
    register/ is not an authorized API (it does not require a signature in the request header)
    Can handle registration of multiple subscribers in one call

After registration, subscribers need to go through a verification process (handled by the subscribe/ API) to become fully active in the network.

subscribe/
Why is this API used:
    The subscribe API is used to change the status to "SUBSCRIBED" after verification
    Allows adding/updating domains for an existing subscriber
    Allows adding new keys or updating existing ones
    Triggers the verification process( the subscriber actually controls the claimed URL and private key) through OnSubscribe task(using a challenge)
    Allows updating the subscriber's URL

How does this API work:
if 
    no signature in the request ||
    signature's verified flag is false in registry ||
    signature received in the request does not match to the subscribers saved signature ||
    verification of signature after decryption fails
then
    returns error

If no subscribers are provided in the request (empty request payload):
    Checks if the role isn't already subscribed
    Sends a challenge to the subscriber url, challenge must be decrypted and ans must be returned correctly
    subscribers send the decrypted challenge string as part of the sync response to the on_subscribe call
else if subscribers are provided in the payload:
    each subscribe request can have many subscribers, for each subscriber in subscribers
        if subscriber passes a key in request
            if it is a new key or the key passed is not verified
                it is considered a new key.
        set signing public key and enc public key in the passed key
        Cannot modify a verified registered key, need to create a new one
        each subscriber can have many domains, for each domain
            create a NetworkRole
            populate NetworkRole with subscriber_id, domain, type, NPid, url, and save it

other flows implemented using /subscribe endpoint
    Adding new keys:
        subscriber would make an authorized call to "/subscribe 
        if the  unique_key_id is not present in registry:
            create the entry for the key and mark it invalid.
        else if the unique_key_id is present and invalid 
            update the details 
        else if unique_key_id  is valid
            Throw exception , already exists. .
        end
        send a challenge corresponding to each added key
        if challenge in resolved for a key, mark it as verified
    Adding new service regions for a domain:
        Subscriber makes an authorized /subscribe request to the registry url with payload of new location
    Updating subscription url
        Subscriber makes an authorized /subscribe request to the registry url with payload { url : "" }
    Invalidating a key
        Pass valid_until to a current  time to expire the key. 

the format of the response is like
```
{
    "subscriber_id": "example.com",
    "subscriber_url": "https://example.com",
    "status": "INITIATED",  // or "SUBSCRIBED" after verification
    "domain": "example.com",
    "type": "BG",
    "signing_public_key": "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...",
    "encr_public_key": "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...",
    "pub_key_id": "key-123",
    "valid_from": "2024-01-01T00:00:00Z",
    "valid_to": "2025-01-01T00:00:00Z",
    "city": "Bangalore",
    "country": "India",
    "created": "2024-01-01T00:00:00Z",
    "updated": "2024-01-01T00:00:00Z"
}
```

Note: 
    subscribe/ is an authorized API (it require a signature in the request header)
    String pub_key_id = params.get("pub_key_id"); // specification uses unique_key_id
    String subscriber_id = params.get("subscriber_id");
    Can handle verification of multiple participant keys
    Updates subscriber status to "SUBSCRIBED" after successful verification
    If for any reason, challenge verification process could not complete, the subscriber can request a /on_subscribe by issuing an authorized /subscribe call on the registry with an empty payload. {}


The OnSubscribe task is responsible for verifying a subscriber's identity and completing their subscription process

on_subscribe/
When It's Used:
    When a new subscriber is registered
    When a subscriber's status changes to "INITIATED"
    When a new key is added
    When manually triggered through an empty payload /subscribe call

How does this API work:
    numResponsesRemaining is set to require numbers of challenge need to be passed. (keys.size())
    Registry sends a new challenge for each key (in a loop) for the participant which are not verified,  
        numResponsesRemaining is decremented for each passed challenge in the loop, for each key

    if at the end of the loop, the numResponsesRemaining==0, keys are marked as verified or subscriber as subscribed.

Notes:
    Registry sends below 3 parameters in the onSubscribe call. different than specification.
        input.put("subscriber_id", subscriber.getSubscriberId()); // should not be used in the payload as per the specification
        input.put("pub_key_id", pk.getKeyId()); // pub_key_id is not defined in the specification, related parameter is unique_key_id
        input.put("challenge", encrypted);







