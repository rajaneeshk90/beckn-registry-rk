register/

Reads the input payload
Parses it as JSON
Handles both single subscriber (JSONObject) and multiple subscribers (JSONArray)
For each subscriber in the payload:
    create NetworkParticipant
    create NetworkRole
    create ParticipantKey
Returns single subscriber JSON for single registration
Returns array of subscribers for multiple registrations

After registration, subscribers need to go through a verification process (handled by the subscribe API) to become fully active in the network.


subscribe/
String pub_key_id = params.get("pub_key_id"); // specification uses unique_key_id
String subscriber_id = params.get("subscriber_id");

this implementation uses authorized calls. headers must have the signing keys present, that means subscriber needs to have an entry in the registry before calling /subscribe API. or else it throws an error.

Can handle verification of multiple participant keys
Updates subscriber status to "SUBSCRIBED" after successful verification

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



The OnSubscribe task is responsible for verifying a subscriber's identity and completing their subscription process

on_subscribe/
When It's Used:
    When a new subscriber is registered
    When a subscriber's status changes to "INITIATED"
    When a new key is added
    When manually triggered through an empty payload /subscribe call

Registry sends below 3 parameters in the onSubscribe call. different than specification.
    input.put("subscriber_id", subscriber.getSubscriberId()); // should not be used in the payload as per the specification
    input.put("pub_key_id", pk.getKeyId()); // pub_key_id is not defined in the specification, related parameter is unique_key_id
    input.put("challenge", encrypted);

numResponsesRemaining is set to require numbers of challenge need to be passed. (keys.size())
Registry sends a new challenge for each key (in a loop) for the participant which are not verified,  
    numResponsesRemaining is decremented for each passed challenge in the loop, for each key

if at the end of the loop, the numResponsesRemaining==0, keys are marked as verified or subscriber as subscribed.





