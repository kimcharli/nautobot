# Nautobot v1.0

This document describes all new features and changes in Nautobot 1.0, a divergent fork of NetBox 2.10.  For the launch of Nautobot 1.0 and for the purpose of this document,  all “new” features or “changes” are referring to the features and changes comparing Nautobot 1.0 coming from NetBox 2.10.  All future release notes will only refer to features and changes relative to prior releases of Nautobot.

Users migrating from NetBox to Nautobot should also refer to the ["Migrating from NetBox"](../installation/migrating-from-netbox.md) documentation as well.

## v1.0.0b1 (2021-02-24)

### Added

#### Custom Fields on All Models

[Custom fields](../additional-features/custom-fields.md) allow user-defined fields, or attributes, on specific data models such as sites or devices. Historically, custom fields have been supported only on “primary” models (Site, Device, Rack, Virtual Machine, etc.) but not on “organizational” models (Region, Device Platform, Rack Group, etc.) or on “device component” models like interfaces. As of Nautobot 1.0, custom fields are now supported on every model, including interfaces.

#### Customizable Statuses

A new ["Status" model](../models/extras/status.md) has been added, allowing users to define additional permitted values for the "status" field on any or all of the models that have such a field (Cable, Circuit, Device, IPAddress, PowerFeed, Prefix, Rack, Site, VirtualMachine, VLAN). The default sets of statuses permitted for each model remain the same as in NetBox 2.10, but you are now free to define additional status values as suit your needs and workflows.

One example application for custom statuses would be in defining additional values to apply to a Device as part of an automation workflow, with statuses such as `upgrading` or `rebooting` to reflect the progress of each device through the workflow, allowing automation to identify the appropriate next action to take for each status.

#### Data Validation Plugin API

Data quality assurance in Nautobot becomes easier with the new [data validation plugin API](../plugins/development.md#implementing-custom-validators). This makes it possible to codify organizational standards.  Using a data validation [plugin](../../plugins/), an organization can ensure all data stored in Nautobot meets its specific standards, such as enforcing device naming standards, ensuring certain prefixes are never used, asserting that VLANs always have a name, requiring interfaces to always have a description, etc. The ability to ensure a high quality of data becomes much more streamlined; error-prone, manual process becomes automated; and there is no more need to actively run reports to check data quality.

A [data validation plugin](https://github.com/nautobot/nautobot-plugin-data-validation-engine) is available that addresses many common use cases for data validation, but you are also free to implement your own plugin to meet your own unique requirements.

#### Detail Views for more Models

Detailed view pages are now provided for models including ClusterGroup, ClusterType, DeviceRole, Manufacturer, Platform, and RackRole.

#### Docker-Based Development Environment

In addition to the previously available virtual-environment-based developer workflow, Nautobot now additionally supports a [development environment based around Docker](../development/getting-started.md#docker-development-environment-workflow) as an alternative.

#### Git Integration as a Data Source

[Git integration](../models/extras/gitrepository.md) offers users an option to integrate into a more traditional NetDevOps pipeline for managing Python modules, Jinja templates, and YAML/JSON data.  There are several use cases that have historically required users to either manage Python modules on the filesystem or use Jinja2 templates within the GUI. With this new feature, users can add a Git repository from the UI or REST API, the contents of which will be synchronized into Nautobot immediately and can be later refreshed on-demand. This allows users to more easily update and manage:

- *Jobs* - store your Python modules that define Jobs (formerly known as Custom Scripts and/or Reports) in a Git repository
- *Export Templates* - store your Jinja templates used to create an export template in a Git repository
- *Config Contexts* - store your YAML/JSON data used within a config context in a Git repository
- *Arbitrary Files* - usable by custom plugins and apps

Not only does this integration and feature simplify management of these features in Nautobot, it offers users the ability to use Git workflows for the management of the jobs, templates, and data ensuring there has been proper review and approval before updating them on the system.

#### GraphQL Support

Nautobot now provides an HTTP API endpoint supporting [GraphQL](https://graphql.org/). This feature adds a tremendous amount of flexibility in querying data from Nautobot. It offers the ability to query for specific datasets across multiple models in a single query.  Historically, if you wanted to retrieve the list of devices, all of their interfaces, and all of their neighbors, this would require numerous REST API calls.  GraphQL gives the flexibility to get all the data desired and nothing unnecessary, all in a single API call.

For more details, please refer to the GraphQL website, as well as to the [Nautobot GraphQL](../additional-features/graphql.md) documentation.

#### Plugin API Enhancements

Plugins can now provide custom [data validation](#data-validation-plugin-api) logic.

Plugins can now include executable [Jobs](../additional-features/jobs.md) (formerly known as Custom Scripts and Reports) that will automatically be added to the list of available Jobs for a user to execute.

Additional data models defined by a plugin are automatically made available in [GraphQL](../additional-features/graphql).

Plugins can now define additional Django apps that they require and these dependencies will be automatically enabled when the plugin is activated.

#### Single Sign-On / Social Authentication Support

Nautobot now supports single sign on as an authentication option using OAuth2, OpenID, SAML, and others, using the [social-auth-app-django](https://python-social-auth.readthedocs.io/en/latest/) module. For more details please refer to the guide on [SSO authentication](../configuration/authentication/sso.md).

#### User-Defined Relationships

User-Defined, or "custom", [relationships](../additional-features/relationships.md) <!-- FIXME no docs yet --> allow users to create their own relationships between models in Nautobot to best suit the needs of their specific network design. Nautobot comes with opinionated data models and relationships.

<!-- FIXME: improve example here -->
For example, a VLAN is mapped to a Site by default.  After a VLAN is created today, you then assign that VLAN to an Interface on a Device. This Device should be within the initial mapped Site.  However, many networks today have different requirements and relationships for VLANs (and many other models): VLANs may be limited to racks in Layer 3 DC fabrics; VLANs may be mapped to multiple buildings in a campus; they may span sites.  Other use cases include circuits, ASNs, or IP addressing--just to name a few--allowing users to define the exact relationships required for their network.

### Changed

#### Code Reorganization

All of the individual Django apps in NetBox (`dcim`, `extras`, `ipam`, etc.) have been moved into a common `nautobot` Python package namespace. The `netbox` application namespace has been moved to `nautobot.core`. This will require updates when porting NetBox custom scripts and reports to Nautobot jobs, as well as when porting NetBox plugins to Nautobot.

#### Configuration and Settings

TODO FIXME

- installed apps
- nautobot_config.py
- nautobot-server init
- ???

#### Consolidating Custom Scripts and Reports into Jobs

Nautobot has consolidated NetBox's "custom scripts" and "reports" into what is now called [Jobs](../additional-features/jobs.md).

The job history ([results](../models/extras/jobresult.md)) table on the home page now shows metadata on each job such as the timestamp and the user that executed the job. Additionally, jobs can be defined and executed by the system and by plugins, and when they are, users can see their results in the history too. UI views have been added for viewing the details of a given job result, and the [JobResult](../models/extras/jobresult.md) model now provides standard APIs for Jobs to log their status and results in a consistent way.

Job result history is now retained indefinitely unless intentionally deleted. Historically only the most recent result for each custom script or report was retained and all older records were deleted.

Python modules that define jobs can now be stored in Git and easily added to Nautobot via the UI as documented above in [Git Integration as a Data Source](#git-integration-as-a-data-source).

#### Hiding UI Elements based on Permissions

Historically, a user viewing the home page and navigation menu would see a list of all model types and menu items in the system, with a “lock” icon on items that they were not granted access to view in detail.

As an [option](../configuration/optional-settings.md#hide_restricted_ui), administrators can now choose to instead hide un-permitted items altogether from the home page and the navigation menu, providing a simpler interface for limited-access users. The prior behavior remains as the default.

#### Installation and Bringup

TODO FIXME

- pip install
- nautobot-server
- ???

#### Navigation Menu Changes

The "Other" menu has been renamed to "Extensibility" and many new items have been added to this menu.

[Status](../models/extras/status.md) records have been added to the "Organization" menu.

#### New Name and Logo

NetBox has been changed to Nautobot throughout the code, UI, and documentation, and Nautobot has a new logo.

#### Packaging Changes

Nautobot is now packaged using [Poetry](https://python-poetry.org/) and builds as an installable Python package. `setup.py` and `requirements.txt` have been replaced with `pyproject.toml`. Releases of Nautobot are now published to [PyPI](https://pypi.org/), the Python Package Index, and therefore can now be installed using `pip install nautobot`.

#### User-Defined Custom Links

Nautobot allows for the definition of [custom links](../models/extras/customlink.md) to add to various built-in data views. These can be used to provide convenient cross-references to other data sources outside Nautobot, among many other possibilities.

Historically this feature was restricted so that only administrators could define and manage custom links. In Nautobot this has been moved into the main user interface, accessible to any user who has been granted appropriate access permissions.

#### User-Defined Export Templates

Nautobot allows for the definition of custom [Jinja2 templates](../models/extras/exporttemplate.md) to use to format exported data.

Historically this feature was restricted such that only administrators could define and edit these templates. In Nautobot this has been moved into the main user interface, accessible to any user who has been granted appropriate access permissions.

#### User-Defined Webhooks

Nautobot allows for the creation of [webhooks](../models/extras/webhook.md), HTTP callbacks that are triggered automatically when a specified data model(s) are created, updated, and/or deleted.

Historically this feature was restricted such that only administrators could define and manage webhooks. In Nautobot this has been moved into the main user interface, accessible to any user who has been granted appropriate access permissions.

#### UUID Primary Database Keys

Database keys are now defined as Universally Unique Identifiers (UUIDs) instead of integers, protecting against certain classes of data-traversal attacks.

### Removed

#### Secrets

Secrets storage and management has been removed from Nautobot.

#### Related Devices

The "Related Devices" table has been removed from the detailed Device view.

### Fixed

- Fixed a bug in which object permissions were not filtered correctly in the admin interface. <!-- FIXME(john): improve the description of this fix -->
- Fixed a bug in which the UI would report an exception if the database contains ChangeLog entries that reference a nonexistent ContentType.

---
