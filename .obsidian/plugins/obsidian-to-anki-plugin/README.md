# Changes made

The changes were made in [main.js](https://github.com/envico801/obsidian-config/blob/main/.obsidian/plugins/obsidian-to-anki-plugin/main.js) and serve to work with the [conversion script](https://github.com/envico801/obsidian-to-anki-card-converter) and/or fix indentation problems.

These 3 changes go together

1. This avoids selecting whitespaces that are possibly behind the start of a code block.

    ```javascript
    // @#@# text = text.replace(/(?:^|\n)(?: {0,3})(```+|~~~+)(?: *)([^\s`~]*)\n([\s\S]*?)\n(?: {0,3})\1/g, function (wholeMatch, delim, language, codeblock) {
    text = text.replace(/(```+|~~~+)(?: *)([^\s`~]*)\n([\s\S]*?)\n*\1/g, function (wholeMatch, delim, language, codeblock) {
    ```

2. This is to avoid adding 2 new lines in front of a captured code block, to keep the previous ident (for example if we are in a list with indentation).

    ```javascript
    // @#@# return '\n\n¨G' + (globals.ghCodeBlocks.push({text: wholeMatch, codeblock: codeblock}) - 1) + 'G\n\n';
    return '¨G' + (globals.ghCodeBlocks.push({text: wholeMatch, codeblock: codeblock}) - 1) + 'G\n\n';
    ```

3. This modifies how lists are selected, it allows to select everything with the same indentation, including code blocks.

    ```javascript
    // @#@# text = text.replace(/(\n\n|^\n?)(( {0,3}([*+-]|\d+[.])[ \t]+)[^\r]+?(¨0|\n{2,}(?=\S)(?![ \t]*(?:[*+-]|\d+[.])[ \t]+)))/gm,
    text = text.replace(/(\n\n|^\n?)(( {0,3}([*+-]|\d+[.])[ \t]+)[^\r]+?(¨0|\n{2,}(((\s{3,}¨G\dG\n{1,}\s{3,}[\s\S]*?)(?=(\n^\S)))|(?=\S))(?![ \t]*(?:[*+-]|\d+[.])[ \t]+)))/gm,
    ```

This prevents the plugin from highlighting any part of the text, for now I don't use it. 

```javascript
// @#@# const HIGHLIGHT_REGEXP = /==(.*?)==/g;
const HIGHLIGHT_REGEXP = /@#@#@#@/g;
```

This changes the way mathematical expressions are selected, they now use the `§` symbol instead of the `$` symbol *(conflicts with javascript 💀)*.

```javascript
// const OBS_INLINE_MATH_REGEXP = /(?<!\$)\$((?=[\S])(?=[^$])[\s\S]*?\S)\$/g;
const OBS_INLINE_MATH_REGEXP = /(?<!\§)\§((?=[\S])(?=[^$])[\s\S]*?\S)\§/g;
```
```javascript
// const OBS_DISPLAY_MATH_REGEXP = /\$\$([\s\S]*?)\$\$/g;
const OBS_DISPLAY_MATH_REGEXP = /\§\§([\s\S]*?)\§\§/g;
```

This fixes the regex to be able to add multi-language tags to obsidian (tags in obsidian format - #tagName).

```javascript
// @#@# const OBS_TAG_REGEXP = /#(\w+)/g;
const OBS_TAG_REGEXP = /#([\p{L}\p{N}\p{Emoji}\p{M}_/-]+)/gu;
```

These 2 changes create a custom format for creating the code blocks, allowing us to add the corresponding html tags so that the code is highlighted correctly

1. Here it checks if it is a custom code block (created by us) and if it is, it leaves it as is and replaces it with a unique identifier (`¨G{number}G`) to be modified again later.
   ```javascript
     const customCodeblockRegex = / *<!-- codeblock-start -->([\s\S]*?)<!-- codeblock-end -->/g
     const isThereACustomCodeblock = text.match(customCodeblockRegex)

     if (isThereACustomCodeblock) {
         for (let codeblock of isThereACustomCodeblock) {
             const id = '¨G' + (globals.ghCodeBlocks.push({text: codeblock, codeblock}) - 1) + 'G\n\n';
             text = text.replace(codeblock.trim(), id)
         }
     } else {
         var repFunc = function (wholeMatch, match, left, right) {
             // encode html entities
             var codeblock = left + showdown.subParser('encodeCode')(match, options, globals) + right;
             return '\n\n¨G' + (globals.ghCodeBlocks.push({text: wholeMatch, codeblock: codeblock}) - 1) + 'G\n\n';
         };

         // Hash <pre><code>
         text = showdown.helper.replaceRecursiveRegExp(text, repFunc, '^ {0,3}<pre\\b[^>]*>\\s*<code\\b[^>]*>', '^ {0,3}</code>\\s*</pre>', 'gim');
     }
     
       // @#@#
       // var repFunc = function (wholeMatch, match, left, right) {
       //     // encode html entities
       //     var codeblock = left + showdown.subParser('encodeCode')(match, options, globals) + right;
       //     return '\n\n¨G' + (globals.ghCodeBlocks.push({text: wholeMatch, codeblock: codeblock}) - 1) + 'G\n\n';
       // };
       //
       // // Hash <pre><code>
       // text = showdown.helper.replaceRecursiveRegExp(text, repFunc, '^ {0,3}<pre\\b[^>]*>\\s*<code\\b[^>]*>', '^ {0,3}</code>\\s*</pre>', 'gim');
   ```

2. And in this step it does something very similar to the previous step, it hides the code block for a moment, performs the pertienent checks and reinserts it without the main parser modifying it.
   ```javascript
         // @#@# text = ext.filter(text, globals.converter, options);

         const customCodeblockRegex = / *<!-- codeblock-start -->([\s\S]*?)<!-- codeblock-end -->/g
         const codeblockStartTagRegex = /<!-- codeblock-start -->\s*/g
         const codeblockEndTagRegex = /\s*<!-- codeblock-end -->/g
         const isThereACustomCodeblock = text.match(customCodeblockRegex)
         const codeblocksDict = {}

         if (isThereACustomCodeblock) {
             for (let i = 0; i < isThereACustomCodeblock.length; i++) {
                 let codeblock = isThereACustomCodeblock[i]
                 codeblock = codeblock.replace(codeblockStartTagRegex, "")
                 codeblock = codeblock.replace(codeblockEndTagRegex, "")
                 codeblocksDict[i] = codeblock
                 text = text.replace(isThereACustomCodeblock[i], `<p>C${i}C</p>`)
             }
         }

         // Main parse ⬇️
         text = ext.filter(text, globals.converter, options);

         for (let numKey in codeblocksDict) {
             text = text.replace(`<p>C${numKey}C</p>`, codeblocksDict[numKey])
         }
   ```

This is a modification to the function that creates the links with the note in obsidian. Full explanation in discussion [#553 - some suggestions about ”Add File Link“ function](https://github.com/ObsidianToAnki/Obsidian_to_Anki/discussions/555)

```javascript
        // @#@# note.fields[field] += '<br><a href="' + url + '" class="obsidian-link">Obsidian</a>';

        let mdFilename = ""

        // We separate the string by "&file=".
        const splitUrl = url.split("&file=");

        // Check if the split resulted in two parts
        if (splitUrl.length === 2) {
            // Extract the part after "&file="
            const encodedPathWithExtension = splitUrl[1]; // Extracted part containing path with extension
            const decodedPathWithExtension = decodeURIComponent(encodedPathWithExtension);

            // Split the filename from its directory path
            const pathSegments = decodedPathWithExtension.split('/');
            const filenameWithExtension = pathSegments.pop()?.trim();

            // Check if the filename with extension is not empty
            if (filenameWithExtension) {
                // Split the filename from its extension
                const filenameSegments = filenameWithExtension.split('.');
                mdFilename = filenameSegments[0]?.trim();
            }
        }
        note.fields[field] += '<br><a href="' + url + `" class="obsidian-link">Obsidian${mdFilename ? " - " + mdFilename : ""}</a>`;
```

Check the [configuration folder](https://github.com/envico801/obsidian-config/tree/main/Configuration%20of%20individual%20plugins/obsidian-to-anki-plugin) to know what changes to take into account.
