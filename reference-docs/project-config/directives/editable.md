# Editable

Defined on an HTML tag so that the user can edit the text within.

## Example
```javascript
module.exports = {
  "name": "p",
  "label": "Paragraph",
  "iconUrl": "https://livingdocs-assets.s3.amazonaws.com/magazine-design/assets/images/icons-components/icon_text.svg",
  "directives": [{
    "name": "text",
    "type": "editable",
    "plainText": true,
    "excludeFromTextCount": true,
    "maxLength": 20
  }],
  "html": "<p class=\"text\" doc-editable=\"text\">\n  Paragraph\n</p>"
}
```

## Config Options 

### `plainText`, `tagWhitelist` and `tagBlacklist`

Only one of these options can be used on a single directive. `plainText` does not allow any markup. `tagWhitelist` can be used to have exact control about the possible tags in content. `tagBlacklist` can be used to filter out only a few specific tags and allow everything else.

Note: Block level elements and elements like script and style are already prohibited in editable directives. Use the above options only to filter inline elements like `<a>` or `<strong>`.

- `plainText`: true | false, Ensures that a text can not contain any HTML tags (decodes HTML).
- `tagWhitelist`: Only allows selected tags in the content, e.g. `"tagWhitelist": ['a', 'em']` would only allow a and em tags, but would remove e.g. a strong tag.
- `tagBlacklist`: Removes certain tags from the content, e.g. `"tagBlacklist": ['a']` would remove all `<a>` tags.

### Character length

When a value is specified for at least one of the four character length properties the editor will display an inline counter just below the directive, and a panel in the sidebar. The editor will still save the document if requirements are not met, but publishing will not be possible until the `minLength` and `maxLength` requirements are satisfied. Any combination of these four properties can be used, but there values should be in an order where `minLength <= recommendedMinLength <= recommendedMaxLength <= maxLength`.

`minLength`, number

Defines the minimum length in characters that the text must contain. Use a value of `1` to make the content required. Publishing will not be posible if the value is not reached.

`maxLength`, number

Defines the maximum length in characters that the text must contain. Publishing will not be posible if the value is exceeded.

`recommendedMinLength`, number

Defines the minimum length in characters that the text should contain. Use a value of `1` to recommend adding content.

`recommendedMaxLength`, number 

Defines the maximum length in characters that the text should contain.

![Char Limit UI](./char-limit.png)

### Other

`excludeFromTextCount`, true | false

By default, the text counter counts the text of all editable directives. With this option a single directive can be excluded. The value can be true or false (default).

`placeholder`, string

The default placeholder that is shown in the editor as long as the directive is still empty. So you don't have to use lorem ipsum ;)

`optional`, true | false

If optional is set to true, then the directive will only be rendered if a value is actually filled in, otherwise the tag is skipped.
