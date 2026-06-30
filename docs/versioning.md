# Version Numbering

Starting with PostgreSQL {{pgversion}}, AXDB uses the following format:

`MAJOR.MINOR.PATCH`

For example:

`{{pspgversion}}`

Where:

- **MAJOR** = The upstream PostgreSQL major version (e.g. {{pgversion}} → PostgreSQL {{pgversion}})
- **MINOR** = The upstream PostgreSQL release number
- **PATCH** = AXDB's internal build number (specific to packaging or AXDB-only updates)

| **Release type** | **Version example** | **What changes?** |
| --- | --- | --- |
| **First AXDB build** | {{pgversion}}.0.1 | Initial AXDB build on upstream {{pgversion}}.0, build #1 |
| **AXDB-only update** | {{pgversion}}.0.2 | Build #2 on the same upstream {{pgversion}}.0 |
| **Upstream patch/feature** | {{pgversion}}.1.1 | Upstream release → MINOR bump to 1, PATCH reset to 1 |
| **Next AXDB build** | {{pgversion}}.1.2 | Build #2 on upstream {{pgversion}}.1 |

The above versioning is only applicable to AXDB {{pgversion}}.x.x as it is the only AXDB forked server. The **third digit** is **always** AXDB’s build number, never an upstream patch. If you see it go from …1 → …2 this means a AXDB-specific update was shipped.

!!! note
    If you're looking for more information, check the [FAQ](faq.md#does-minor-cover-both-upstream-feature-and-patch-releases).
