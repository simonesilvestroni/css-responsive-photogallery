# Responsive photogallery and carousel in HTML and CSS

Companion technical gists for Minutes to Midnight’s related [case study](https://minutestomidnight.co.uk/projects/web-design/responsive-photogallery-carousel/). The photogallery and carousel are embedded as a module in the static site generator Jekyll.

## Disclaimer

The markup in the source files is bloated with many _utility classes_ from Bootstrap v5, because that was the CSS framework I was using back then. I since moved away from the concept altogether.

---

### The module

In order to add a photo gallery in a page or post, my module just needed to be invoked as an inclusion:

```liquid
{% include pattern-imagegallery.html folder="/assets/images/gallery-press/" id="1" %}
```

The HTML file carries the logic and the markup, while the two parameters are: a path to the folder containing the images, and a numeric ID. The only requirement is to follow the same naming convention for all the images, where the small low resolution version, used for preview in the page and navigation in the carousel, would have the prefix `thumb-`.

### Gallery thumbnails

Taking advantage of Jekyll's _static files_ feature, I set a default local path for all the images in the configuration file. It's instructing the system to treat each file contained in that path as an image:

```yaml
defaults:
  - scope:
      path: "assets/images"
    values:
      image: true
```

The `section` tag wrapping the whole module has its specific dynamic ID, necessary for when multiple galleries are present in the same page. Inside, a `div` tag serves as a flexbox container to show the thumbnails in a centered row.

```html
<section id="gallery-{{ include.id }}">
  <div class="d-flex flex-wrap justify-content-center">
```

The following logic fetches the correct images by filtering the path I passed earlier, working out their filenames to dynamically print a caption:

```liquid
{%- for image in site.static_files | where: "image", true -%}
  {%- capture galleryPath -%}{{ include.folder }}{%- endcapture -%}
  {%- if image.path contains galleryPath -%}
    {%- assign filenameparts = image.path | split: "/" -%}
    {%- assign imgCaption = filenameparts | last | replace: image.extname,'' | replace: 'thumb-', '' | replace: 'a_','' | replace: 'b_','' | replace: 'c_','' | replace: 'd_','' | replace: 'e_','' | replace: 'f_','' | replace: 'g_','' | replace: 'h_','' | replace: 'i_','' | replace: 'l_','' | replace: 'm_','' | replace: 'n_','' | replace: 'o_','' | replace: 'p_','' | replace: 'q_','' | replace: 'r_','' | replace: 's_','' | replace: 't_','' | replace: 'u_','' | replace: 'v_','' | replace: 'z_','' | replace: '-',' ' | replace: '_',' ' -%}
    {%- if image.path contains 'thumb-' -%}
    <div>
      <img src="{{ image.path }}" alt="{{ imgCaption | capitalize }}" width="150" height="150">
      <span>{{ imgCaption }}</span>
    </div>
    {%- endif -%}
  {%- endif -%}
{%- endfor -%}
```

To break it down:

- A `for` loop iterates through all the image files.
- `galleryPath` is a variable capturing the path parameter that I passed with the include function in the actual page.
- The first `if` condition restricts the context to the images contained within the `galleryPath`.
- The second `if` condition further restrics the context to the thumbnails.
- Image captions are generated through two steps:
    - `filenameparts` takes the actual image path, split through the trail slash;
    - `imgCaption` takes the last part of the file name minus the directory path and removes file suffixes and all the bits that aren’t useful to generate the caption.

### Creating the modal window

Before the closing div and section, I include a second pattern containing the modal window and the carousel itself:

```html
    {%- include pattern-modal-carousel.html -%}
  </div>
</section>
```

The modal is wrapped in another flexbox `div`, where the first element is the button responsible for opening the modal itself:

```html
<div>
  <input type="checkbox" id="m2m-modal-{{ include.id }}" name="m2m-modal-{{ include.id }}">
  <label for="m2m-modal-{{ include.id }}">📷 <span><strong>Open the gallery</strong></span></label>
```

By passing the same ID, I make sure each gallery has its own instruction, otherwise the system wouldn't know which one to open and close. Since the mechanism responsible for opening and closing the modal window is based on the well-established [checkbox hack](https://css-tricks.com/the-checkbox-hack/), a `label` HTML element is the _actual button_. The modal takes 95 percent of the available browser window width and height and is hidden by default, via SCSS:

```scss
.m2m-modal-container {
  [type="checkbox"]:not(:checked) {
    @extend .visually-hidden;
  }
}

.m2m-modal-wrap {
  width: 95vw;
  height: 95vh;
  margin: auto;
  border-radius: 4px;
  overflow: hidden;
  background-color: #000;
  align-self: center;
  opacity: 0;
  transform: scale(0.6);
  transition: opacity 250ms 250ms ease, transform 300ms 250ms ease;
}
```

I make the window appear when the checkbox is selected:

```scss
.m2m-modal-btn:checked ~ .m2m-modal {
  pointer-events: auto;
  opacity: 1;
  transition: all 300ms ease-in-out;
}
```

The close button is added in an `:after` pseudo-class, including a bit of further media query code (not shown here) to change its size on small devices.

```scss
.m2m-modal-btn:checked + label:after,
.m2m-modal-btn:not(:checked) + label:after {
  position: fixed;
  content: '❌';
  top: 4px;
  right: 4px;
  width: 40px;
  height: 40px;
  line-height: 40px;
  font-size: 1rem;
  text-align: center;
  background-color: $white;
  border-radius: 10em;
  transition: all 200ms linear;
  opacity: 0;
  pointer-events: none;
  transform: translateY(20px);
}
```

### Creating the carousel

The carousel is contained in a couple of `div`s, an unordered list and a navigation pattern. The list element contains the hi-res image, filtered by exclusion with the `unless` condition.

#### Hi-res images

```html
<div>
  <div>
    <ul scroll-behavior="smooth">
      {%- unless image.path contains 'thumb-' -%}
      <li>
        <div id="{{ imgCaption | slugify }}-{%- increment slideId -%}">
          [=== slide image here ===]
        </div>
      </li>
      {%- endunless -%}
    </ul>
    [=== navigation here ===]
  </div>
</div>
```

To break in down:

- The logic to pull the image is omitted, since it's the same as shown earlier for the thumbnails. 
- The ID for the `div` containing the image is made by two parts:
  - the caption variable with a `slugify` Jekyll filter which turns the spaces into dashes while removing capitalizations;
  - a filter to add an incremental number and keep the ID different throughout the carousel.
- _The hi-res images need to be responsive_ so that users on small devices could download pictures that weren't larger than their viewport. Avoiding one of the widespread reasons why Pagespeed fails with photo galleries was paramount.

#### Navigation

A navigation row sits at the bottom of the modal window, featuring the thumbnails linking to the related hi-res image above.

```html
<nav id="m2m-slider-nav">
  {% comment %}
    *-----------------------------------
    Here's the same logic used earlier
    to fetch the images from Jekyll's
    static files functionality.
    -----------------------------------*
  {% endcomment %}
  {%- assign imageNavPath = image.path | split: "/" | last | prepend: 'thumb-' -%}
  {%- assign slideId = 0 -%}
  {%- assign slideNavId = 0 -%}
  {%- assign slideNav = 0 -%}
  <a href="#{{ imgCaption | slugify }}-{%- increment slideNavId -%}">
    <img src="{{ galleryPath }}{{ imageNavPath }}" alt="{{ imgCaption | capitalize }}" title="Click to view {{ imgCaption | capitalize }}" width="120" height="120">
  </a>
</nav>
```

I introduce here three new variables. They generate numerically incrementing IDs that keep the navigation in sync with the hi-res images.

### Mobile functionality

Through SASS code, I made sure users could change slide by swiping left or right on the image, respecting any preference set in either the browser or the operating system for `reduced-motion`.

```scss
.m2m-carousel-scroll {
  @media (prefers-reduced-motion: no-preference) {
    scroll-behavior: smooth;
    @-moz-document url-prefix() {
      scroll-behavior: auto;
    }
  }
  display: flex;
  overflow-y: hidden;
  width: 100%;
  margin: 0;
  padding: 0;
  -webkit-overflow-scrolling: touch;
  scrollbar-width: none;
}
.m2m-carousel-scroll-item > img:active {
  cursor: grabbing;
  cursor: -webkit-grabbing;
}
@supports (scroll-snap-align: start) {
  .m2m-carousel-scroll {
    scroll-snap-type: x mandatory;
  }
  .m2m-carousel-scroll-item-outer {
    scroll-snap-align: center;
  }
}
@supports not (scroll-snap-align: start) {
  .m2m-carousel-scroll {
    scroll-snap-type: mandatory;
    scroll-snap-destination: 0 50%;
    scroll-snap-points-x: repeat(100%);
  }
  .m2m-carousel-scroll-item-outer {
    scroll-snap-coordinate: 0 0;
  }
}
```

### Responsive images

As stated earlier, I addressed the issue of large images on small devices. Since I had already implemented responsive images in the website, I just decided to grab the code to render the hi-res pictures which generates different sizes for different media viewports:

```liquid
{%- assign respFileNamePart = filenameparts | last -%}
{%- assign respImgPath = respFileNamePart | prepend: galleryPath | remove_first: "/" -%}
{% responsive_image_block %}
  path: {{ respImgPath }}
  alt: {{ imgCaption | capitalize }}
  margin-nil: 0
{% endresponsive_image_block %}
```

The source code above generates the following HTML:

```html
<figure>
  <img src="({{ site.url }}/assets/images/responsive/1200/a_in-cambridge.jpg" alt="In cambridge" srcset="({{ site.url }}/assets/images/responsive/576/a_in-cambridge.jpg 576w,({{ site.url }}/assets/images/responsive/768/a_in-cambridge.jpg 768w,({{ site.url }}/assets/images/responsive/1200/a_in-cambridge.jpg 1200w, ({{ site.url }}/assets/images/gallery-press/a_in-cambridge.jpg 1600w">
</figure>
```

It renders responsive images inside a `figure` tag, using `srcset` with the smallest resized image used as a fallback. Every time a new gallery is added to a page, Jekyll generates all the resized versions on its own.

---

### Note

Please note that the checkbox hack is not fully accessible. I need to figure out an alternative.

---