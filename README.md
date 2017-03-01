# Shopify Liquid Json Parser / Decoder

As the name suggests these snippets allow you to be able to use json strings either directly or pulled from metafields inside your Shopify templates.

## Getting Started

This project is built entirely from liquid which means it is supported on all Shopify stores. Let's hope that one day this library is replaced by native support.

### Installing

Installing is rather simple. Just copy the two following files in to your snippets directory. The file names are important so keep them the same.

```
json_decode.liquid - This is the file that does the hardwork
jd__function.liquid - This provides core functions for the json_decode snippet and also provides convenience methods.
json_decode_output.liquid - This file is optional and just provides another way to access the data
json_lazy_decode.liquid - This file is optional and provides a more efficient decode for single array values.
```

And your done!

## Usage

Ok lets start with an example. Here we are going to store a JSON string in to the `json` variable in our template. This is standard liquid.

```liquid
{%- capture 'json' -%}
{
    "product": {
        "colour": "red",
        "condition": "new",
        "body_html": "Test",
        "vendor": "Apple",
	}
}
{%- endcapture -%}
```

Usually from here you wouldn't be able to do anything with this data except push it out to render on the client side and handle it with JS.

Lets now load this string in to the json_decode snippet so that we can then access it later in the liquid file

```liquid
{%- capture json_error -%}
    {%- include 'json_decode' jd__namespace:'example' jd__data:json -%}
{%- endcapture -%}
```

A quick break down of what the above does

1. We wrap the include in a `capture` as it doesn't actually output anything unless there is an error.
If there is an error the `capture ` puts it in a variable rather than exposing it to your store customers.
This is optional however it is highly recommended to use this method.

2. We `include` the file `json_decode` from our snippets directory. Along with this we set the top level namespace for our json which is important to be able to handle multiple json files.
As we can see this is done by passing the string `example` to the `jd__namespace` include variable. We also then pass our `json` variable we created earlier in to the `jd__data` include variable.
Both `jd__namespace` and `jd__data` are required for the snippet to work.

What the above does for us is actually build 2 new variables in our liquid template called `jd__global_keys` and `jd__global_values`. It is possible to use these directly however the
following methods allow you to access the values in a much simplier way.

## Accessing your JSON data.

There are multiple methods to access your data and we will start with the simpliest approach and that is outputting your data directly in your template.

Your data is accessed by using a simple dot notation syntax.

#### First Method


To access the `body_html` in the json example above we would first use our namespace `example` that was set above in your include statement and then following our nested json object as per below.

`example.product.body_html`

We can't however use this directly within a variable tag as Liquid doesn't allow assignment of variables dynamically. Instead we use a helper snippet and in this case `json_decode_output`

`{% include 'json_decode_output' with 'example.product.body_html' %}`

We simply pass the dot syntax to the `with` function of the include statement.

#### Second Method

The same method can be achieved by using another include snippet. The `jd__function` snippet works by passing a function name and variables in to the `with` part of the include tag.

The available functions are `echo`, `keys` and `values`. All functions variables are passed by a delimited pipe `|` symbol.

-----

##### ECHO

The `echo` function works by first adding the `|` symbol and then the variable name you want to access such as below

```liquid
{% include 'jd__function' with 'echo|example.product.body_html' %}
```

-----

##### KEYS

The `keys` function allows you to access the keys of arrays and objects so that you can either iterate over or output the data.

The syntax for `keys` is `keys|:name|:separator` the `:name` attribute is the name of the Object or Array you want to access. In the above example we could use `example.product` to
access an array containing `colour, condition, body_html and vendor` with their full namespaces. The `separator` attribute is optional and if not used will cause the below function to output its
 value to a `jd__yield_1` liquid variable. If the `separator` attribute is used then the array will be outputted directly to the template using the provided seperator between array keys.

```liquid
{%- include 'jd__function' with 'keys|example.product' -%}{%- assign product_keys = jd__yield_1 -%}
```
The above will set `product_keys` as an array which can then be used in for loops or with array filters.

```liquid
{%- include 'jd__function' with 'keys|example.product|<br>' -%}
```

This on the other hand will output each of the keys directly and be separated with the defined `<br>`

-------

##### VALUES

The `values` function matches the above `keys` function with the only difference being the variables value will be outputted rather than the key name

```liquid
{%- include 'jd__function' with 'values|example.product' -%}{%- assign product_keys = jd__yield_1 -%}
```
The above will set `product_keys` as an array which can then be used in for loops or with array filters.

```liquid
{%- include 'jd__function' with 'values|example.product|<br>' -%}
```

This on the other hand will output each of the keys directly and be separated with the defined `<br>`

##### VALUES

The `values` function matches the above `keys` function with the only difference being the variables value will be outputted rather than the key name

```liquid
{%- include 'jd__function' with 'values|example.product' -%}{%- assign product_values = jd__yield_1 -%}
```
The above will set `product_keys` as an array which can then be used in for loops or with array filters.

```liquid
{%- include 'jd__function' with 'values|example.product|<br>' -%}
```

This on the other hand will output each of the keys directly and be separated with the defined `<br>`

## Example Usage

Looping over JSON object/array keys

```liquid
{%- include 'jd__function' with 'keys|example.product' -%}{%- assign product_keys = jd__yield_1 -%}
{%- for key in product_keys -%}
{%- assign prepared_function = echo | append: '|' | append: key -%}
    {{ key }} has a value of {% include 'jd__function' with prepared_function %} <br>
    {{ key }} has a value of {% include 'json_decode_output' with key %} <br>
{%- endfor -%}
```

Storing a variable

```liquid
{%- capture body_html -%}{% include 'jd__function' with 'echo|example.product.body_html' %}{%- endcapture -%}
```

Accessing Keys and Values

```liquid
{% include 'json_decode_output' with 'example.product__keys' | split: jd__separator_2 | join: '<br>'  %}
{% include 'json_decode_output' with 'example.product__values' | split: jd__separator_2 | join: '<br>'  %}
```

## Full Usage

```liquid
{%- capture 'json' -%}["one","two","three"]{%- endcapture -%}
{%- capture json_error -%}{%- include 'json_decode' jd__namespace:'count' jd__data:json -%}{%- endcapture -%}
{%- if json_error != '' -%}{{ json_error }}{%- endif %}
{%- include 'jd__function' with 'values|count' -%}{%- assign values = jd__yield_1 -%}
Counting to 3 : {{ values | join: ", " }}
```

## Performance Issues and Work Arounds

As this library is written in liquid its processing speed isn't the greatest. 

We have found when looping over products in a collection can slow the liquid render time to greater than 5 seconds.

To overcome this we have developed a very simple lazy parser that only looks for a key and responds with the array value.

Usage is as follows

```liquid
{% include 'json_lazy_decode' json: product.metafields.global.JAN key: 'three_sixty' %}{% assign three_sixty = jd__yield_1 %}
```
