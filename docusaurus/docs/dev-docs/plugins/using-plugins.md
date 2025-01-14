---
title: Using Strapi plugins
description: todo
displayed_sidebar: devDocsSidebar
pagination_next: dev-docs/plugins/documentation
---

# Using Strapi built-in plugins

:::info
This section is about using Strapi built-in plugins from a developer's perspective. Not what you're looking for? Read the [plugins introduction](/dev-docs/plugins) and find your use case and recommended section to read from there.
:::

## Built-in plugins

Strapi comes with the following built-in plugins that are officially maintained and documented by the Strapi core team:

<CustomDocCardsWrapper>
<CustomDocCard emoji="ℹ️" title="Content Source Map" description="The Content Source Map plugin, used with Vercel Visual Editing, enhances the content edition experience." link="/dev-docs/plugins/content-source-map" />
<CustomDocCard emoji="ℹ️" title="Documentation" description="The Documentation plugin is useful to document the available endpoints once you created an API." link="/dev-docs/plugins/documentation" />
<CustomDocCard emoji="✉️" title="Email" description="The Email plugin enables applications to send emails from a server or an external provider." link="/dev-docs/plugins/email"/>
<CustomDocCard title="GraphQL" description="The GraphQL plugin adds a GraphQL endpoint to fetch and mutate your content." link="/dev-docs/plugins/graphql"/>
<CustomDocCard emoji="🌍" title="Internationalization (i18n)" description="The i18n plugin allows creating, managing and distributing localized content in different languages." link="/dev-docs/plugins/i18n"/>
<CustomDocCard emoji="👀" title="Sentry" description="The Sentry plugin enables you to track errors in a Strapi application using Sentry." link="/dev-docs/plugins/sentry"/>
<CustomDocCard emoji="⬆️" title="Upload" description="The Upload plugin powers the Media Library found in the admin panel and allows versatile file uploads." link="/dev-docs/plugins/upload"/>
<CustomDocCard emoji="👥" title="Users & Permissions (U&P)" description="The U&P plugin offers JWT-based authentication and ACL strategies for API protection and user permissions." link="/dev-docs/plugins/users-permissions"/>
</CustomDocCardsWrapper>

## Automatic plugins discovery

Strapi automatically loads plugins installed with npm. Under the hood, Strapi scans every `package.json` file of the project dependencies, and looks for the following declaration:

```json
"strapi": {
  "kind": "plugin"
}
```

Installed plugins can also be manually enabled or disabled.

## Manual enabling/disabling of plugins

To disable a plugin without uninstalling it, switch its `enabled` key to `false` in the [`/config/plugins.js` file](/dev-docs/configurations/plugins).
