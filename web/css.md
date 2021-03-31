# CSS

## SCSS

```
# placeholder selector
# https://sass-lang.com/documentation/style-rules/placeholder-selectors

%action-button {
  width: 100px;
  height: 100px;
  background-color: red;
}

.button-save {
  @extend %action-button;
  background-color: blue;
}

.button-delete {
  @extend %action-button;
  background-color: yellow;
}
```
