# useful-js-snippets
Useful js snippets to manipulate DOM through console (or more preferred way of saving the function to Chrome's code snippets).

## Search and Replace text elements
- Modifies text inside e.g. div, span and p
- is Case sensitive.
- source: https://chat.openai.com/share/07bd388c-615d-4876-865d-0ac43889c68f 
``` javascript
{// extra brackets so that the function is not cached.
function searchAndReplace(original, replace) {
    // Function to recursively search and replace text in text nodes
    function replaceText(node) {
        if (node.nodeType === Node.TEXT_NODE) {
            if (node.textContent) {
                node.textContent = node.textContent.replace(new RegExp(original, 'g'), replace);
            }
        } else {
            node.childNodes.forEach(replaceText);
        }
    }

    // Start the search and replace from the body of the document
    replaceText(document.body);
}
}
```

## Addstyle

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
## Filter
 
u-block filters:

https://www.reddit.com/r/assholedesign/comments/p5phe2/you_are_now_completely_unable_to_view_twitter_on/  
  
```
twitter.com##div[role='dialog']
twitter.com##[id$='PromoSlot']
twitter.com##html->body:style(overflow:visible !important;)
twitter.com##html:style(overflow:visible !important;)
```

Those last two lines can be applied to any website at all that blocks scrolling 
  
 ## Generate UUID
 
  
  ``` javascript
  let guid = function() {
	function s4() {
	  return Math.floor((1 + Math.random()) * 0x10000)
		.toString(16)
		.substring(1);
	}
	return s4() + s4() + '-' + s4() + '-' + s4() + '-' +
	  s4() + '-' + s4() + s4() + s4();
  }
```

## Extract elements from a website
 ``` javascript
{
    // Adjust this selector to match your page's main content/container
    let mainContent = document.getElementsByClassName('v-slot-main-content')[0];

    if (mainContent) {
        let all_text = '';

        // Variables for table parsing
        let inTable = false;
        let inThead = false;
        let tableHeaderCount = 0;
        let hasHeaderRow = false;  // We'll detect once we parse a row with <th>
        let currentRowCells = [];

        /**
         * Flushes the current row to the output as a Markdown row:
         * e.g. "| cell1 | cell2 | cell3 |"
         */
        function flushCurrentRow() {
            if (currentRowCells.length > 0) {
                all_text += '| ' + currentRowCells.join(' | ') + ' |\n';
                currentRowCells = [];
            }
        }

        /**
         * Inserts the separator row (for Markdown) if we encountered a header row:
         * e.g. "| --- | --- | --- |"
         */
        function insertHeaderSeparator(count) {
            let separatorCells = [];
            for (let i = 0; i < count; i++) {
                separatorCells.push('---');
            }
            all_text += '| ' + separatorCells.join(' | ') + ' |\n';
        }

        /**
         * Recursively parses inline child nodes into a single string.
         * - Converts links <a href="...">link text</a> into [link text](...)
         * - Converts <strong>/<b> to **...**
         * - Converts <em>/<i> to *...*
         * - Recursively handles nested elements.
         */
        function parseInlineNodes(parentNode) {
            let textBuffer = '';
        
            parentNode.childNodes.forEach(child => {
                if (child.nodeType === Node.TEXT_NODE) {
                    // Plain text
                    textBuffer += child.textContent;
                } 
                else if (child.nodeType === Node.ELEMENT_NODE) {
                    let tag = child.tagName.toLowerCase();
        
                    if (tag === 'a') {
                        // Convert <a href="...">...</a> to [text](href)
                        let href = child.getAttribute('href') || '';
                        let linkText = parseInlineNodes(child);
                        textBuffer += `[${linkText}](${href})`;
                    } 
                    else if (tag === 'strong' || tag === 'b') {
                        // Convert <strong>/<b> to **...** but preserve leading/trailing whitespace
                        let strongText = parseInlineNodes(child);
        
                        // Capture leading and trailing whitespace
                        let leading = strongText.match(/^(\s+)/) ? strongText.match(/^(\s+)/)[0] : '';
                        let trailing = strongText.match(/(\s+)$/) ? strongText.match(/(\s+)$/)[0] : '';
        
                        // Trim content so that ** markers are recognized properly
                        let content = strongText.trim();
                        textBuffer += `${leading}**${content}**${trailing}`;
                    } 
                    else if (tag === 'em' || tag === 'i') {
                        // Convert <em>/<i> to *...* but preserve leading/trailing whitespace
                        let emText = parseInlineNodes(child);
        
                        let leading = emText.match(/^(\s+)/) ? emText.match(/^(\s+)/)[0] : '';
                        let trailing = emText.match(/(\s+)$/) ? emText.match(/(\s+)$/)[0] : '';
        
                        let content = emText.trim();
                        textBuffer += `${leading}*${content}*${trailing}`;
                    } 
                    else {
                        // For any other inline element, recurse to gather text
                        textBuffer += parseInlineNodes(child);
                    }
                }
            });
        
            return textBuffer;
        }

        // Recursive function to traverse child nodes
        function traverseNodes(node) {
            if (node.nodeType === Node.ELEMENT_NODE) {
                let tag = node.tagName.toLowerCase();

                // -- HEADERS (h1 - h6) --
                if (['h1', 'h2', 'h3', 'h4', 'h5', 'h6'].includes(tag)) {
                    let headerLevel = parseInt(tag.charAt(1));
                    let prefix = '#'.repeat(headerLevel);
                    all_text += prefix + ' ' + node.textContent.trim() + '\n\n';
                }

                // If you have a special class you want to capture as normal text:
                else if (node.classList && [...node.classList].some(cls => cls.includes("user-chat-width"))) {
                    all_text += node.textContent.trim() + '\n\n';
                }

                // -- CODE BLOCKS --
                else if (tag === 'pre') {
                    // Adjust language fence as you prefer (e.g., ```java, ```js, etc.)
                    all_text += '```java\n' + node.textContent.trim() + '\n```\n\n';
                }

                // -- PARAGRAPHS --
                else if (tag === 'p') {
                    // Skip <p> if its direct parent is an <li>, to prevent duplicate text in lists
                    let parentTag = node.parentElement?.tagName?.toLowerCase();
                    if (parentTag !== 'li') {
                        // Also skip if we are inside a table cell (because we handle the entire cell at once)
                        if (!inTable) {
                            // Use parseInlineNodes to preserve inline tags like <a>, <strong>, <em>, etc.
                            let pContent = parseInlineNodes(node).trim();
                            if (pContent.length > 0) {
                                all_text += pContent + '\n\n';
                            }
                        }
                    }
                }

                // -- LIST ITEMS (WITH INDENTATION + ORDERED LIST SUPPORT) --
                else if (tag === 'li') {
                    // Determine indentation depth by counting ancestor <ul>/<ol>
                    let depth = 0;
                    let ancestor = node.parentElement;
                    while (ancestor) {
                        let ancestorTag = ancestor.tagName && ancestor.tagName.toLowerCase();
                        if (ancestorTag === 'ul' || ancestorTag === 'ol') {
                            depth++;
                        }
                        ancestor = ancestor.parentElement;
                    }
                    // For each level above 1, add 2 spaces
                    let indentation = '  '.repeat(Math.max(0, depth - 1));

                    // Check immediate parent to see if it's <ol> or <ul>
                    let parentTag = node.parentElement && node.parentElement.tagName
                        ? node.parentElement.tagName.toLowerCase()
                        : '';

                    // Convert list item content (including possible <a>/<strong>/<em> inline elements)
                    let liContent = parseInlineNodes(node).trim();

                    if (parentTag === 'ol') {
                        // Markdown auto-numbers if each item is "1."
                        all_text += `${indentation}1. ${liContent}\n`;
                    } else {
                        all_text += `${indentation}- ${liContent}\n`;
                    }
                }

                // -- TABLE HANDLING START --
                else if (tag === 'table') {
                    inTable = true;
                    all_text += '\n';  // Blank line before a table
                    // Recurse over children
                    Array.from(node.childNodes).forEach(traverseNodes);
                    // After finishing the table
                    flushCurrentRow();
                    inTable = false;
                    inThead = false;
                    tableHeaderCount = 0;
                    hasHeaderRow = false;
                    currentRowCells = [];
                    all_text += '\n';  // Blank line after table
                }

                else if (tag === 'thead') {
                    inThead = true;
                    // Recurse children
                    Array.from(node.childNodes).forEach(traverseNodes);
                    inThead = false; // done with thead
                }

                else if (tag === 'tbody') {
                    // Recurse children
                    Array.from(node.childNodes).forEach(traverseNodes);
                }

                else if (tag === 'tr' && inTable) {
                    // Before processing this <tr>, flush any leftover row data from the previous one
                    flushCurrentRow();
                    // Recurse so we can capture <th> or <td> inside
                    Array.from(node.childNodes).forEach(traverseNodes);
                    // Now flush the row we just built
                    let rowLength = currentRowCells.length;
                    flushCurrentRow();

                    // If it was a header row and we haven't inserted the separator yet
                    if (hasHeaderRow && !tableHeaderCount) {
                        tableHeaderCount = rowLength;
                        if (tableHeaderCount > 0) {
                            insertHeaderSeparator(tableHeaderCount);
                        }
                    }
                }

                // -- HEADERS IN TABLE --
                else if (tag === 'th' && inTable) {
                    hasHeaderRow = true;
                    // Collect textContent from <th> (including any children)
                    let textContent = node.textContent.trim().replace(/\s+/g, ' ');
                    currentRowCells.push(textContent);
                }

                // -- CELLS IN TABLE --
                else if (tag === 'td' && inTable) {
                    // Collect textContent from <td> (including children)
                    let textContent = node.textContent.trim().replace(/\s+/g, ' ');
                    currentRowCells.push(textContent);
                }

                // -- DIV WITH SPECIFIC STYLE (CODE BLOCK) --
                else if (tag === 'div') {
                    let styleAttr = node.getAttribute('style') || '';
                    if (
                        styleAttr.includes('background:#eeeeee') &&
                        styleAttr.includes('border:1px solid #cccccc') &&
                        styleAttr.includes('padding:5px 10px')
                    ) {
                        // All text content as a code block without a language spec
                        let codeText = node.textContent.trim();
                        all_text += '```\n' + codeText + '\n```\n\n';
                    } else {
                        // If not the style we want, just recurse
                        Array.from(node.childNodes).forEach(traverseNodes);
                    }
                }

                // For any other element we haven't explicitly handled, recurse
                else {
                    Array.from(node.childNodes).forEach(traverseNodes);
                }
            }
        }

        // Start traversing from the main content element
        traverseNodes(mainContent);

        // Finally, log the collected text
        console.log(all_text);
    } else {
        console.log('No element with class "v-slot-main-content" found.');
    }
}

 ```
