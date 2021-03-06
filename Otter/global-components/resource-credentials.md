﻿---
title: Resource Credentials
keywords: otter,executions
---

Otter can manage, store, and control access to tokens, passwords, API keys, or any of the other secrets needed for automating modern infrastructure with the use of Resource Credentials.

You can associate different credentials with different [Environments](../modeling-infrastructure/environments), and permit or restrict different [Users](/docs/otter/administration/security) from managing or viewing the actual passwords for credentials. Of course, credentials are always encrypted before being stored.

There are several built-in types, and new types may be added with an [Extension](../administration/extensions):

{.docs}
- **Private Key** - a private key with an optional username and passphrase
- **BuildMaster** - a [connection to the BuildMaster](https://inedo.com/support/tutorials/buildmaster/utilizing-infrastructure-sync) API
- **ProGet** - a connection to a [ProGet server](https://inedo.com/proget)
- **Username & Password** - just a simple username and password, such as a Windows domain account

## Example: IIS Application Pool Credentials

In Microsoft's IIS, an Application Pool may be configured to run under a certain user account. Obviously, to do that, you must supply that username and password. Both of these fields are available on the [Ensure App Pool](../reference/operations/iis/ensure-app-pool) operation, but that means you would need to have those values be stored in plain text, in your [Plan](/docs/otter/core-concepts/plans).

![Storing a Credential in Otter](/resources/documentation/otter/resource-3.png){.screenshot}

Alternatively, you could create a Username & Password credential...

![Storing a Credential in Otter](/resources/documentation/otter/resource-1.png){.screenshot}

... and then specify that, for the Otter credentials property.

![Using a Credential in Otter](/resources/documentation/otter/resource-2.png){.screenshot}


Otter will automatically map the appropriate fields at configuration time, allowing you to give the team viability into which credentials are being used, without sharing the actual credentials themselves.

## Scoped Credentials {#scoped data-title="Scoped Credentials"}

Prior to Otter v2.2.4, resource credentials required unique names, preventing cascading behavior across environments, or in other words, the ability to reference a single resource credential across all environments. As of Otter v2.2.4, this restrction is removed enabling a single credential name (e.g. "WindowsAccount") to be created against multiple environments such that the correct one is chosen at execution time based on the environment in context.

The following rule is enforced by Otter when resolving resource credentials:

When resolving a resource credential at execution time, candidates by name are selected in order of matching properties:

{.docs}
 - environment
 - ancestor environment
 - system
