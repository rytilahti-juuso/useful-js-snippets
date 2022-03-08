useful-js-snippets

https://stackoverflow.com/questions/15505225/inject-css-stylesheet-as-string-using-javascript

There are a couple of ways this could be done, but the simplest approach is to create a <style> element, set its textContent property, and append to the page’s <head>.
``` javascript
/**
 * Utility function to add CSS in multiple passes.
 * @param {string} styleString
 */
function addStyle(styleString) {
  const style = document.createElement('style');
  style.textContent = styleString;
  document.head.append(style);
}

addStyle(`
  body {
    color: red;
  }
`);

addStyle(`
  body {
    background: silver;
  }
`);
 ``` 
  
If you want, you could change this slightly so the CSS is replaced when addStyle() is called instead of appending it.
``` javascript
/**
 * Utility function to add replaceable CSS.
 * @param {string} styleString
 */
const addStyle = (() => {
  const style = document.createElement('style');
  document.head.append(style);
  return (styleString) => style.textContent = styleString;
})();

addStyle(`
  body {
    color: red;
  }
`);

addStyle(`
  body {
    background: silver;
  }
`);
```
  
  
u-block filters:
```
twitter.com##div[role='dialog']
twitter.com##[id$='PromoSlot']
twitter.com##html->body:style(overflow:visible !important;)
twitter.com##html:style(overflow:visible !important;)
```

Those last two lines can be applied to any website at all that blocks scrolling 
