# plugin-category-cloud

![Category Cloud Screenshot](https://raw.githubusercontent.com/moonbuck/plugin-category-cloud/main/category-cloud.jpeg)

A plugin for [Micro.blog](https://micro.blog "Micro.blog") that creates a page displaying links to your categories layed out as a category cloud. It is based upon the category cloud plugin you might already know and love that lives [here](https://github.com/chtny/plugin-category-cloud).

Once installed, the generated page will be found at `[SCHEME]://[HOSTNAME]/cloud/`.

Let’s have a look at the parameters:

![Plugin Parameters](https://raw.githubusercontent.com/moonbuck/plugin-category-cloud/main/plugin_parameters.jpeg)

The maximum and minum font sizes for the links are in [rem](https://www.sitepoint.com/understanding-and-using-rem-units-in-css/ "Understanding and Using REM Units in CSS") units. The maximum and minimum font weights are expected to be [integer](https://www.w3schools.com/cssref/pr_font_weight.asp "CSS Font Weight Property") values.

All of these values are optional. The defaults used when absent are those values used as placeholders in the plugin parameter fields.

The `Cloud Exclusion List` parameter is also optional. It’s default value is not the placeholder, but an empty `string`. If, however, you choose to place a `,` separated list of your categories such as `my-awesome-category, Imma Be Anchorized`; then, the plugin will skip over these categories when generating the cloud so that they do not appear in the output.

The page template lives at `layouts/cloud/single.html`.

```html
{{ define "main" }}

{{ with site.Taxonomies.categories }}

{{/* Max and min page counts */}}
{{ $max_count := mul (len (index .ByCount 0).Pages) 1.0 }}
{{ $min_count := mul (len (index .ByCount.Reverse 0).Pages) 1.0 }}

{{/* Max and min font sizes in rem units */}}
{{ $max_size := 1.34 }}
{{ with site.Params.cloud_max_rem }}{{ $max_size = . | float }}{{ end }}
{{ $min_size := 0.9 }}
{{ with site.Params.cloud_min_rem }}{{ $min_size = . | float }}{{ end }}

{{/* Max and min font weights */}}
{{ $max_weight := 800 }}
{{ with site.Params.cloud_max_weight }}{{ $max_weight = . | int }}{{ end }}
{{ $min_weight := 200 }}
{{ with site.Params.cloud_min_weight }}{{ $min_weight = . | int }}{{ end }}

{{ $exclusion_list := slice "" }}
{{ with site.Params.cloud_exclusion_list }}
  {{ $exclusion_list = apply (split . ",") "anchorize" "." }}
{{ end }}

<div id="category-cloud">

{{ range $name, $taxonomy := . }}

{{ if not (in $exclusion_list $name )}}
  {{ $page_count := len $taxonomy.Pages }}
  {{ $percent_max_count := div $page_count ($max_count) }}
  {{ $font_size := mul $max_size $percent_max_count }}
  {{ if (lt $font_size $min_size) }}{{ $font_size = $min_size }}{{ end }}
  {{ $font_weight := mul $max_weight $percent_max_count }}
  {{ if (lt $font_weight $min_weight) }}{{ $font_weight = $min_weight }}{{ end }}
  {{ $fract_weight := mod $font_weight 100 }}
  {{ $font_weight = cond (eq $fract_weight 0) $font_weight (add (sub $font_weight $fract_weight) (cond (ge $fract_weight 50) 100 0)) }}

<a 
  href="{{ "/categories/" | relLangURL }}{{ $name | urlize }}"
  class="category-cloud-link"
  style="font-size:{{ $font_size }}rem; font-weight:{{ $font_weight }}"
  >{{ $name }}({{ $page_count }})</a>
  
  {{ end }}
{{ end }}

</div>

{{ end }}

{{ end }}
```

The stylesheet used to layout the cloud leaves at `static/assets/css/category-cloud.css`.

```css
#category-cloud { 
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-content: center;
  align-items: baseline;
  gap: 0px 10px;
  width: 90%;
  max-width: 900px;
  padding: 0px 20px;
  margin: 40px auto; 
}

.category-cloud-link { 
  color: #666; 
  margin: 0px;
  padding: 0px;
}

.category-cloud-link:hover { color: #000; }
```

The file living at `content/cloud.md` specifies the front matter for the page.

```json
{
  "title": "Cloud",
  "description": "A category cloud weighted by posts per category.",
  "type": "cloud",
  "menu": {
    "main": {
      "name": "Cloud",
      "title": "Cloud",
      "identifier": "cloud",
      "url": "/cloud/",
      "weight": 110
    }
  }
}
```

Leave the `type` alone (as that is what points it to `layouts/cloud/single.html`). You can play with the `title` and `description` values. These set the values I would expect your theme to draw from when constructing the page `<head>`. The `menu` creates a menu item to include the page in your navigation. Leave the `url` alone (this is how the link’s target is derived). You can play around with the other values. You can adjust the value of `weight` to slide the menu item up or down your list of navigation items. You can remove the `menu` entry entirely if you do not want the page to show up in your navigation menu.