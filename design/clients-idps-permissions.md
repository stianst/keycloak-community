# Identities

* Server admins - should be able to restrict globally what can be used within a realm
* Realm admins - should be able to restrict what can or can not be used within a realm by what identities
* Client registrator - an identity that can register clients
* Trust registrator - an identity that can register trust relationships with Keycloak

# Clients

## Client registration authentication

When a client is registered we need to authenticate the client registrator regardless of the mechanism.

For Admin APIs that is straightforward as the client registrator is either an admin or a service account.

For DCR it is one of:

* An admin
* A service account
* An initial access token
* Software Statement (not supported atm) - a third-party has attested to the client metadata 

For CIMD there currently is no concept of authentication, but there should be. It should be:

* An HTTPS server - as the client metadata is retrieved from an HTTPS endpoint the server is authenticated as part of the TLS initation
* Software Statement (not supported atm) - a third-party has attested to the client metadata

## Client registration permissions

Overly broad permissions on creating clients can be a big security risk, and depending on the identity it should
be possible to limit what can or can not be registered.

Some examples of what a client registrator identity may or may not be able to do

* Add a client scope on a role
* Add a protocol mapper
* Add a claim; indirectly adding/overwriting a claim using a protocol mapper
* Add a client scope
* Issue tokens for a specific audience (client)
* Enable specific capabilities for the client

## Protocol mappers vs client scopes vs claims

Client scopes uses one or more protocol mappers to add one or more claims to a token. There is a problem today that 
we don't have a strict concept of claims, with the exception of some hardcoded not-allowed claims.

Claims in tokens should be a first class citizen in Keycloak, where protocol mappers are required to explicitly declare
what claims they add. Or, anything else the protocol mapper does for that matter.

## Client policies

Client policies should in general be associated with an identity. During registration the relevant identity is the
client registrator identity, and while in use it is the client identity.

There is some overlap with client policies, client registration policies, and FGA, that we need to sort out:

* There should not be client policies and client registration policies, only client policies
* Permissions should ideally be driven by authorization services, and not by client policies

Think of it this way:

* Cedar is a great way to allow or deny something - for example this client registrator is not permitted to map this client scope to this client
* OPA is a great way to do more fine-grained policies - for example the redirect_uri base url must match the post_redirect_uri base url

Cedar and OPA are just examples; we should have a built-in mechanisms in Keycloak as well such as authorization services
and client policies. Client policies should ideally be extended to a generic policy mechanism in Keycloak, that we 
can build policies on other things as well, not just clients.

## Two layers of protection

It's insufficient to only check permissions and policies when a client is registered. Permissions and policies should
also be checked when issuing tokens for instance:

* A client registrator is not permitted to add a specific protocol mapper when registering the client
* A client is not permitted to override or add a specific claim when tokens are issued

# Identity Provider and User Federation providers

When registering an identity provider or user federation provider there is a federation registrator identity, that
can at very course level limit what can be done. Similarly to clients we need mechanisms to enforce what mappers and
other settings can be registered.

Additionally, we need to ability to associate an identity provider or user federation with an identity, as today it 
operates without any identity, meaning it can do anything. The same should apply to other "automation", all aspects of
identity providers, but also to workflows.

If we introduce the ability to associate an identity (a special service account?) with a user federation provider for 
example we can control what roles for instance it can grant to users through FGA. That can be further extended on 
with for instance a custom Cedar policy.

# Authorization Services

Authorization Services and FGA are very important; we should really double down on this and continue evolving it. Further,
we should enable authorization services more throughout Keycloak, where some examples include:

* Can a user authenticate to a client
* Can a user federation provider grant a role to a user

We should also add support for text-based policies, where Cedar could be a great fit for Keycloak. The nice thing about
authorization services is that we can plug-in other policy engines with a custom KC authorization policy. We don't need
to support Cedar and Authorization Services in for example Admin endpoints directly, Admin endpoints can use authorization
services, which in turn can invoke Cedar policies.

# Policies

We need more policies in Keycloak, but at the same time we don't need many different ways to achieve the same. We should
have the same policy framework and a consistent language regardless if its client policies, user federation policies, 
authentication policies, etc.

We should really also support text-based policies, OPA is a candidate, but from the perspective of Keycloak would be
fairly heavy. We may want to have our own language, or something.

# User Profile

User profile today has some mechanism to select what attributes a user or an admin can manage. It's very course grained
and does not extend to for example user federation providers.

Ideally this should be a permission thing, and not something baked into user attributes directly, so we should consider
some way of leveraging authorization services to provider better control over what identities can read or write what
user attributes.

We also need to figure out how to extend the concept of user profiles to clients, identity providers, realm attributes, etc.
Any place we use attributes, there should be validation and a schema associated with it. 

# Trust Relationships

Any aspect of Keycloak that delegates thrust to a third-party should be configured through a trust relationship. That
is what we have identity providers and user federation providers for.

Similarly, CIMD should be considered a trust relationship. In order to use CIMD the first thing required would be to
configure the trust relationship. That basically boils down to a client federation trust with one or more trusted 
domains/URLs, and ability to verify software statements. 

Other examples of client federation trust relationships include SPIFFE, Kubernetes Service Accounts, OpenID Federation,
and potentially even DCR.

# Verifiable Claims

We don't have any mechanism to verify or trust claims beyond the fact that someone at some point updated the value.

By introducing some concept of verified claims we can attest to the value, and provide higher level of assurance on the value of the claim.

This could for instance be a custom user attribute that is mapped to a claim in a token. Where does this value come from? Who updated it? Have we verified it in any way?

This can also be valid in terms of standard claims such as exp, aud, etc.. We don't strictly know how these values where set today. Did Keycloak set these values, or did some random protocol mapper set it?

For verifiable credentials this part is rather important; a verifiable credential issued from untrusted user attributes is not exactly all that meaningful, which is why today we require an admin to verify the attributes prior to creating a credential from it. However, if we can attest to the attributes with a reasonable level of assurance then we can automatically create credentials from them.

## Verifying claims

The value of a claim can be verified by one or more humans; for instance the user could update the value, then a manager attest to it; it could also be an admin that updates the value and one or more other admins verifies it.

We should also support third-party verification of claims. For example looking up a user in a database to verify their
employment records, or sending a email to have the user verify their own email address. The latter should probably be sending a code in the email to add to the login, rather than a required action ;)

Finally, we could also leverage attestetions to digitally verify a signature from a third-party to attest to the claim.