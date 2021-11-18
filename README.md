# plugin-category-cloud

![Category Cloud with Page Counts](https://raw.githubusercontent.com/moonbuck/plugin-category-cloud/main/images/with_counts.jpeg)

A plugin for [Micro.blog](https://micro.blog "Micro.blog") that creates a page displaying links to your categories layed out as a category cloud. Once installed, the generated page will be found at `[SCHEME]://[HOSTNAME]/cloud/`. It is based upon the category cloud plugin you might already know and love that lives [here](https://github.com/chtny/plugin-category-cloud).

## Parameters
### Plugin Interface
The plugin’s parameter interface looks like this:

![Plugin Parameters](https://raw.githubusercontent.com/moonbuck/plugin-category-cloud/main/images/plugin_parameters.jpeg)

All of the plugin’s simple parameter values are settable via the interface. All of the simple parameters plus two additional parameters may be set via data file. For an understanding of how my plugins use data files, have a look at [this post](https://moondeer.blog/2021/11/11/feeding-data-to.html "Feeding Data to My Plugins").

### Data File
The template data file is located at `data/plugin_category_cloud/params.toml` and it looks like this (notice the comments describe what the parameters do):

```TOML
# Max font size to use in the cloud in rem units.
#
MaxSize = 1.34

# Min font size to use in the cloud in rem units.
#
MinSize = 0.9

# Max font weight to use in the cloud.
#
MaxWeight = 800

# Min font weight to use in the cloud.
#
MinWeigth = 200

# Category names are fetched in their anchorized form.
# Setting this parameter to true will convert anchorized
# forms into capitalized and spaced forms.
#
#   my-category → My Category
#
Humanize = true

# The default path to a category page takes the form:
#  /categories/my-anchorized-category
# 
# If the category has been lifted into the main menu,
# and its path has been altered (perhapse dropping /categories)
# the plugin can try to match a category to its menu item and
# use the URL value of the menu item.
#
MenuItemMatching = true

# Whether to display parenthesized page counts along
# with the category names.
#
DisplayPageCounts = true

# Class to set on the cloud's category <a> tags.
#
Class = 'category-cloud-link'

# List of categories to exclude from the category cloud.
#
# For example: ['Pinned', 'Temporary']
#
ExclusionList = [ ]

# The plugin fetches the category names in anchorized form.
# The 'Humanize' parameter will return the category names
# to their capitalized and spaced form. The DisplayNames
# parameter allows for direct control over how a category
# name will be displayed
#
[DisplayNames]

# To define exactly how to display a specific category name 
# enter its value below with its anchorized form as the key.
# My use case for this option is this category:
#
# stream-of-consciousness = 'Stream of Conscioussness'
#

```

### Examples
#### `ExclusionList`
[moondeer.blog](https://moondeer.blog) is configured to include a *Pinned* category. None of the screenshots, however, happen to include such a category. The reason is that I have configured the value of `ExclusionList` like so:

```TOML
ExclusionList = ['Pinned]
```

#### `DisplayPageCounts`

The screenshot at the top shows a category cloud that includes parenthesized page counts. Configure the value of `DisplayPageCounts` like so:

```TOML
DisplayPageCounts = false
```

And you get a category cloud that’s all:

![Category Cloud without Page Counts](https://raw.githubusercontent.com/moonbuck/plugin-category-cloud/main/images/without_counts.jpeg)

The CSS for the category cloud gets injected into the page head via `layouts/partials/category-cloud-style.html`:

```go
{{- /* Inject category cloud styling into page head */ -}}

{{- $display_page_counts := true -}}

{{- /* Resolve the data file and check for parameter value */ -}}
{{- $data_file := site.Data.plugin_category_cloud_params -}}
{{- $data_file = $data_file | default site.Data.plugin_category_cloud.params -}}

{{- with $data_file -}}
  {{- if (isset . "DisplayPageCounts") -}}
    {{- $display_page_counts = .DisplayPageCounts -}}
  {{- end -}}
{{- end -}}
      
{{- /* Check for value set via plugin parameter interface */ -}}
{{- with site.Params.cloud_display_page_counts -}}
  {{- if (in (slice "true" "false") .) -}}
    {{- $display_page_counts = eq . "true" -}}
  {{- end -}}
{{- end -}}
    
<style>
div#category-cloud { 
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-content: center;
  align-items: baseline;
  gap: 10px 10px;
  width: 90%;
  max-width: 900px;
  padding: 0px 20px;
  margin: 40px auto; 
}

a.category-cloud-link { 
  color: #666; 
  margin: 0px;
  padding: 0px;
  line-height: 1;
}

a.category-cloud-link > span.category-count {
  display: {{ cond $display_page_counts "initial" "none" }};
}

a.category-cloud-link:hover { color: #000; }
</style>
```

The page count spans are always included. The value of `DisplayPageCounts` controls whether `span.category-count` elements get their `display` property set to `initial` or `none`. I figure this way it would be easy to toggle them on and off using Javascript should I ever feel the desire.

#### `Humanize`

Set `Humanize` to `false` and you get a cloud displaying the anchorized category names kinda like:

![Non-Humanized](https://raw.githubusercontent.com/moonbuck/plugin-category-cloud/main/images/non-humanized.jpeg)

#### `DisplayNames`

Setting `Humanize` to `true` will only get you so far. Where I to revert `DisplayNames` to its default value:

```TOML
[DisplayNames]

```

I would end up with a category cloud that was all:

![Without Display Names](https://raw.githubusercontent.com/moonbuck/plugin-category-cloud/main/images/without_display_names.jpeg)

By configuring `DisplayNames` to override a couple of troublesome category names:

```TOML
[DisplayNames]
microblog = 'Micro.blog'
stream-of-consciousness = 'Stream of Consciousness'
```

I am able to put the period back into *Micro.blog* and correct the capitalization in the middle of *Stream of Consciousness*:

![Category Cloud without Page Counts](https://raw.githubusercontent.com/moonbuck/plugin-category-cloud/main/images/without_counts.jpeg)

#### `MenuItemMatching` 

By creating content files within a custom theme, it is totally possible to change the path to a category’s page. I happen to create files for all my categories so I am able to control various page level attributes. For categories that I also include in the main navigation menu, I also alter their URLs to remove `/categories`. Take the category page for [Critters](https://moondeer.blog/critters/ "Critters"), for example:

```md
+++
title = 'Critters'
description = 'All things critter-related.'
url = '/critters/'
aliases = ['/categories/critters/']

[menu.main]
name = 'Critters'
title = 'Critters'
identifier = 'critters'
url = '/critters/'
weight = 60
+++
*All things critter-related.*
```

Since the plugin generates URLs for the categories that include `/categories` by default, you can tell the plugin to check the main menu item URLs for an item whose URL’s last path component is equal to the category’s anchorized name. When a match is found, the menu item’s URL will be used in place of the default construction.

## Page Configuration

The file living at `content/cloud.md` specifies the front matter for the page.

```md
+++
title = 'Cloud'
description = 'A category cloud weighted by posts per category.'
type = 'cloud'
+++
```

Leave the `type` alone (as that is what points it to `layouts/cloud/single.html`). You can play with the `title` and `description` values. These set the values I would expect your theme to draw from when constructing the page `<head>`.