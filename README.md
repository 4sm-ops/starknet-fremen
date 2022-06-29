# starknet-fremen

## About

The Fremen Protocol is a Web3, smart contracts-based social graph for the StarkNet Ecosystem designed to empower creators to own the links between themselves and their community, forming a fully composable, user-owned social graph. The protocol is built from the ground up with modularity in mind, allowing new features to be added while ensuring immutable user-owned content and social relationships.


## Profiles

Any address can create a profile and receive an ERC-721 Fremen Profile NFT. Profiles are represented by a ProfileStruct:

```
# A struct containing profile data.
#
# pubCount The number of publications made to this profile.
# followNFT The address of the followNFT associated with this profile, can be empty..
# followModule The address of the current follow module in use by this profile, can be empty.
# handle The profile's associated handle.
# uri The URI to be displayed for the profile NFT.

struct ProfileStruct:
    member pubCount: Unit256
    member followNFT: felt
    member followModule: felt
    member handle: felt
    member uri: felt
end
```

Profiles have a specific URI associated with them, which is meant to include metadata, such as a link to a profile picture or a display name for instance, the JSON standard for this URI is not yet determined. Profile owners can always change their follow module or profile URI.

## Publications

Profile owners can `publish` to any profile they own. There are three `publication` types: `Post`, `Comment` and `Mirror`. Profile owners can also set and initialize the `Follow Module` associated with their profile.

Publications are on-chain content created and published via profiles. Profile owners can create (publish) three publication types, outlined below. They are represented by a `PublicationStruct`:

```
# A struct containing data associated with each new publication.

# profileIdPointed The profile token ID this publication points to, for mirrors and comments.
# pubIdPointed The publication ID this publication points to, for mirrors and comments.
# contentURI The URI associated with this publication.
# referenceModule The address of the current reference module in use by this profile, can be empty.
# collectModule The address of the collect module associated with this publication, this exists for all publication.
# collectNFT The address of the collectNFT associated with this publication, if any.

struct PublicationStruct:
    member profileIdPointed: Unit256
    member pubIdPointed: Unit256
    member contentURI: felt
    member referenceModule: felt
    member collectModule: felt
    member collectNFT: felt
end

```

#### Publication Types

##### Post

This is the standard publication type, akin to a regular post on traditional social media platforms. Posts contain:

1. A URI, pointing to the actual publication body's metadata JSON, including any images or text.
2. An uninitialized pointer, since pointers are only needed in mirrors and comments.

##### Comment

This is a publication type that points back to another publication, whether it be a post, comment or mirror, akin to a regular comment on traditional social media. Comments contain:

1. A URI, just like posts, pointing to the publication body's metadata JSON.
2. An initialized pointer, containing the profile ID and the publication ID of the publication commented on.

##### Mirror

This is a publication type that points to another publication, note that mirrors cannot, themselves, be mirrored (doing so instead mirrors the pointed content). Mirrors have no original content of its own. Akin to a "share" on traditional social media. Mirrors contain:

1. An empty URI, since they cannot have content associated with them.
2. An initialized pointer, containing the profile ID and the publication ID of the mirrored publication.

### Profile Interaction

There are two types of profile interactions: follows and collects.

#### Follows

Wallets can follow profiles, executing modular follow processing logic (in that profile's selected follow module) and receiving a `Follow NFT`. Each profile has a connected, unique `FollowNFT` contract, which is first deployed upon successful follow. Follow NFTs are NFTs with integrated voting and delegation capability.

The inclusion of voting and delegation right off the bat means that follow NFTs have the built-in capability to create a spontaneous DAO around any profile. Furthermore, holding follow NFTs allows followers to `collect` publications from the profile they are following (except mirrors, which are equivalent to shares in Web2 social media, and require following the original publishing profile to collect).

#### Collects

Collecting works in a modular fashion as well, every publication (except mirrors) requires a `Collect Module` to be selected and initialized. This module, similarly to follow modules, can contain any arbitrary logic to be executed upon collects. Successful collects result in a new, unique NFT being minted, essentially as a saved copy of the original publication. There is one deployed collect NFT contract per publication, and it's deployed upon the first successful collect.

When a mirror is collected, what happens behind the scenes is the original, mirrored publication is collected, and the mirror publisher's profile ID is passed as a "referrer." This allows neat functionality where collect modules that incur a fee can, for instance, reward referrals. Note that the `Collected` event, which is emitted upon collection, indexes the profile and publication directly being passed, which, in case of a mirror, is different than the actual original publication getting collected (which is emitted unindexed).

## Freemen Modularity

Stepping back for a moment, the core concept behind modules is to allow as much freedom as possible to the community to come up with new, innovative interaction mechanisms between social graph participants. For security purposes, this is achieved by including a whitelisted list of modules controlled by governance.

To recap, the Freement Protocol has three types of modules:

1. `Follow Modules` contain custom logic to be executed upon follow.
2. `Collect Modules` contain custom logic to be executed upon collect. Typically, these modules include at least a check that the collector is a follower.
3. `Reference Modules` contain custom logic to be executed upon comment and mirror. These modules can be used to limit who is able to comment and interact with a profile.

Note that collect and reference modules should _not_ assume that a publication cannot be re-initialized, and thus should include front-running protection as a security measure if needed, as if the publication data was not static. This is even more prominent in follow modules, where it can absolutely be changed for a given profile.

Lastly, there is also a `ModuleGlobals` contract which acts as a central data provider for modules. It is controlled by a specific governance address which can be set to a different executor compared to the Hub's governance. It's expected that modules will fetch dynamically changing data, such as the module globals governance address, the treasury address, the treasury fee as well as a list of whitelisted currencies.

### Upgradeability

This iteration of the Freement Protocol implements a transparent upgradeable proxy for the central hub to be controlled by governance. There are no other aspects of the protocol that are upgradeable. In an ideal world, the hub will not require upgrades due to the system's inherent modularity and openness, upgradeability is there only to implement new, breaking changes that would be impossible, or unreasonable to implement otherwise.

This does come with a few caveats, for instance, the `ModuleGlobals` contract implements a currency whitelist, but it is not upgradeable, so the "removal" of a currency whitelist in a module would require a specific new module that does not query the `ModuleGlobals` contract for whitelisted currencies.

## Security and Privacy concerns

### Concerns

In the next decade, web services will evolve to become truly personal, living in more places than just your browser, and reason over every intimate detail of our personal lives. There are examples to demonstrate this already. For example, in the past five years, the number of in-home smart assistants has grown from zero to half a billion web-connected devices. Our private lives have become a public commodity and as web services evolve to become more personal, we need to rethink how we control our data.

### Pseudonymously 

... with zero knowledge verified NFTs. This means you can prove credentials, ownership, or facts without them tracing back to you.

### ZK Proof

In your wallet(s), you have NFTs that can point back to your identity (aka, getting doxxed). But what if you can verify ownership of NFTs while staying pseudonymous?
When you connect your wallet(s), we verify your NFTs. Then, we create ZK badges out of them.

### ZK badges

This means you can prove ownership of an NFT without it tracing back to you.

### Building your identities with ZK Badges

Once you have ZK proof, you can add create ZK badges for an anonymous wallet.
Zk badges verify you own an NFT but leaves no bread crumbs back to your personal wallets.
