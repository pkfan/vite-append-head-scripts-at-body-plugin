# append-head-scripts-at-body-vite-plugin

### Note
Your all (&lt;script src='index.js'&gt;&lt;/script&gt; tags replace from <head> tags and append at &lt;/body&gt; end tag

## How is this vite plugin work?
This is a vite plugin coded by Pkfan. This plugin work with follwoing steps:

1) Get html compile result from vite   
2) Extract/Remove all ( &lt;head&gt;content&lt;/head&gt; ) from html   
3) Extract/Remove all ( &lt;script src=""&gt;&lt;/script&gt; ) from ( &lt;head&gt;content&lt;/head&gt; )
4) Append all ( &lt;script src=""&gt;&lt;/script&gt; ) into ( &lt;/body&gt; ) end tag
5) return vite final ( &lt;html&gt; content &lt;/html&gt;) result and write into target file

# Follow these steps

#### Step 1: create ( appendHeadScriptsAtBody.js ) file in root directory where ( vite.config.js ) exist

```js
// appendHeadScriptsAtBody.js
function appendHeadScriptsAtBody() {
  return {
    name: "append-head-scripts-at-body",

    transformIndexHtml(html, ctx) {
      const replaceScriptFromHead = (htmlHead) => {
        let script = "";
        const regExpForScriptTagsSelection = new RegExp(
          "<script[^<>]+[^<>]+></script>"
        );

        let htmlHeadResult = htmlHead.replace(
          regExpForScriptTagsSelection,
          (match) => {
            script = `\n\t\t${match}`;
            return "";
          }
        );
        return { script, htmlHeadResult };
      };

      const replaceAllScriptFromHead = (htmlHead) => {
        let scripts = "";
        let htmlFinalHead = htmlHead;
        while (true) {
          let { script, htmlHeadResult } = replaceScriptFromHead(htmlFinalHead);

          if (!(script && script.trim() != "")) {
            break;
          }

          htmlFinalHead = htmlHeadResult;
          scripts = scripts + script;
        }

        return { scripts, htmlFinalHead };
      };

      const extractHeadTagsFromHtml = (htmlInput) => {
        const regExpForHeadTagsSelection = new RegExp("<head>(.|\n)*?</head>");

        let htmlHead = "";

        let htmlResultWithBodyTagsOnly = htmlInput.replace(
          regExpForHeadTagsSelection,
          (match) => {
            htmlHead = match;
            return "";
          }
        );

        return { htmlHead, htmlResultWithBodyTagsOnly };
      };

      const htmlTagStartingLength = (htmlResultWithBodyTagsOnly) => {
        // e.g [ ( <html lang="en"> ) of length 17 ] and [ ( <html> ) of length 6 ]
        const htmlStartTag = new RegExp("<html(.|\n)*?>").exec(
          htmlResultWithBodyTagsOnly
        )[0];
        const indexOfHtmlstartingTag =
          htmlResultWithBodyTagsOnly.indexOf(htmlStartTag);

        return htmlStartTag.length + indexOfHtmlstartingTag;
      };

      const concatHeadIntoHtml = ({
        htmlFinalHead,
        htmlResultWithBodyTagsOnly,
      }) => {
        const htmlStartTagLength = htmlTagStartingLength(
          htmlResultWithBodyTagsOnly
        );
        const startSlice = htmlResultWithBodyTagsOnly.slice(
          0,
          htmlStartTagLength
        );
        const endSlice = htmlResultWithBodyTagsOnly.slice(htmlStartTagLength);
        return `${startSlice}\n${htmlFinalHead}\n${endSlice}`;
      };

      const appendScriptsIntoHtmlEndBodyTag = ({ htmlResult, scripts }) =>
        htmlResult.replace("</body>", `\t\t${scripts}\n\t</body>`);

      if (ctx.chunk.isEntry) {
        let { htmlHead, htmlResultWithBodyTagsOnly } =
          extractHeadTagsFromHtml(html);

        const { scripts, htmlFinalHead } = replaceAllScriptFromHead(htmlHead);
        const htmlResult = concatHeadIntoHtml({
          htmlFinalHead,
          htmlResultWithBodyTagsOnly,
        });

        html = appendScriptsIntoHtmlEndBodyTag({ htmlResult, scripts });
      }

      return html;
    },
  };
}

export default appendHeadScriptsAtBody;

```

### Step 2: import ( appendHeadScriptsAtBody.js ) into ( vite.config.js )
```js
// vite.config.js

import { defineConfig } from "vite";
import appendHeadScriptsAtBody from "./appendHeadScriptsAtBody";

export default defineConfig({
  plugins: [appendHeadScriptsAtBody()],
  
  // other vite code here
});
```
