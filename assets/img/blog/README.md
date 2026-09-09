# Blog figures

Upload figures here, grouped by post, for example:

```text
assets/img/blog/belief-bias/accuracy-by-condition.png
assets/img/blog/position-bias/response-order.png
```

Use lowercase filenames with hyphens. PNG or SVG works well for plots and
diagrams; JPEG works well for photos. Export plots with a background and readable
labels so they also remain legible in dark mode.

Add a figure to the corresponding Markdown file in `_posts/` using the site's
existing figure component (replace the example filename with your uploaded file):

```liquid
{%
  include figure.liquid
  path="assets/img/blog/belief-bias/accuracy-by-condition.png"
  alt="Describe the plotted conditions and the main pattern in the results."
  caption="Figure 1. Explain what is shown, including units and error bars."
  loading="lazy"
%}
```

The component provides a responsive image and a caption. Only embed a figure
after uploading it, so the post never displays a broken image.
