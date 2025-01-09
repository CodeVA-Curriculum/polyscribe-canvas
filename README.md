# `polyscribe-canvas`

This tool is designed to consume a plain-text version of a Canvas course, and render that course on the Canvas website via the Canvas API. This allows course authors to develop courses using their choice of drafting tool, and manage course updates using version control software.

Please review [this sample course repository](https://github.com/CodeVA-Curriculum/polyscribe-canvas/tree/main/docs/sample-course) for a complete example of a course repository designed to be processed by `polyscribe-canvas`.

## Course Structure

The `polyscribe-canvas` tool expects course content to be organized accordingly:

```
assets/
modules/
config.yaml
secret.yaml*
```

*\*Do not stage or commit `secret.yaml`; be sure to add it to `.gitignore` to avoid publishing your Canvas API token*

### `config.yaml`

The two `.yaml` files in the root directory of the course contain course-level information and credentials used to update the Canvas course via the Canvas API.

The `config.yaml` file can define the following fields:

| Field     | Description | Required? | Example |
| --------- | ----------- | --------- | ------- |
| `title`   | The name of the course | ❌ (title will not be overwritten if left undefined) | `title: AI Basics` |
| `id`      | The Canvas course ID, found in the course URL following `courses/` | ✅ | `id: 18151` |
| `authors` | The authors of the course content, separated by commas | ✅ | `authors: Jon Stapleton, Perry Shank` |
| `link`    | The Canvas instance URL (be sure to enclose in quotation marks to avoid parsing errors) | ✅ | `link: https://virtualvirginia.instructure.com` |
<!-- | `outline` | An array providing details on how to arrange the *Modules* page in Canvas. Read more about how to define this field below. | ❌ | See below | -->

Here is an example of a minimal `config.yaml` file:

```yaml
title: AI Basics
authors: Jon Stapleton
id: 18151
link: "https://virtualvirginia.instructure.com"
```

<!-- TODO: #### Defining the Course `outline` in `config.yaml`

The optional `outline` field is an array of objects providing information for how to arrange course elements on the *Modules* page in Canvas. If left undefined, `polyscribe-canvas` will leave the course *Modules* page unchanged. If defined, it will delete the existing *Modules* page and replace it with modules based on the objects defined in the `outline` field in the order written.

The objects in the `outline` array can define the following fields:

| Field      | Required? | Description |
| ---------- | --------- | ----------- |
| `title`    | ✅ | The title of the module |
| `folder`   | ❌ | Optionally, provide the containing folder for elements referenced in the module. The tool will append this string to the beginning of `element` strings, which allows authors to more easily list elements if they are organized into directories based on which module they should belong to. |
| `elements` | ❌ | An array of strings or objects providing information about the course elements which should appear in the module. Read more about defining elements below. |

The items in the `elements` array should be filenames pointing to files in the `modules` directory. Optionally, authors can pass additional settings to the module by using an `object` instead of a `string` to reference the element. Objects in the `elements` array can define the following fields:

| Field    | Type      | Required? | Default | Description |
| -------- | --------- | --------- | ------- | ----------- |
| `file`   | `string`  | ✅ | N/A     | The filename containing the element content |
| `folder` | `boolean` | ❌ | `true`  | If `false`, the tool will use the `file` field to find the element file *without* appending the `folder` field defined in the module object |
| `indent` | `int`     | ❌ | `0`     | Sets the amount of indentation for the element in the module |

The layout of the *Modules* page is completely determined by the contents of the `outline` array. The organization of files within the `modules` folder has no bearing on how `polyscribe-canvas` arranges the *Modules* page. -->

###  `secret.yaml`

The `secret.yaml` file contains one field:

| Field | Description |
| ----- | ----------- |
| `token` | The API access token generated by the instance admin |

**IMPORTANT: You should *never* publish your API access token. Guard it like you would a password, and add the `secret.yaml` to your `.gitignore` file to avoid inadvertently committing its contents to your version control tree.**

### The `modules` Directory

The `modules` directory contains all of the written material in the course. Course material can be organized into sub-directories (often corresponding to Canvas modules). These subdirectories should contain text material written into `.md` files in Markdown format. When `polyscribe-canvas` renders the course material, it will convert the content in the `.md` files to HTML, and create a course element for each `.md` file. 

The `polyscribe-canvas` tool will generate a `manifest.json` file in your `modules` directory when it publishes your course content to Canvas. **Stage and commit this file to your `git` repository--it is required to avoid creating duplicate course elements when rendering your material on Canvas.**

Here is an example of what a typical `modules` directory might look like:

```
modules/
    00-introduction/
        intro.md
        background.md
        context.md
    01-more-material/
        intro.md
        discussion.md
        additional-info.md
    manifest.json*
```

*\*This file is generated by `polyscribe-canvas`. You do not need to create this file yourself, but be sure to stage and commit it using your version control software after it is generated.*

You can read more about the contents of the `modules` directory in the section on **Course Elements** below.

### The `assets` Directory

The `assets` directory should contain all images and other media files referenced in the course content within the `modules` directory. The `assets` directory should have a "flat" structure, with no nested directories.

The `polyscribe-canvas` tool will generate a `manifest.json` file in your `assets` directory when it publishes your course content to Canvas. **Stage and commit this file to your `git` repository.**

Here is an example of what the `assets/` directory might look like for a typical course:

```
assets/
    logo.jpg
    diagram.png
    demo.gif
    avatar.webp
    manifest.json*
```

*\*This file is generated by `polyscribe-canvas`. You do not need to create this file yourself, but be sure to stage and commit it using your version control software after it is generated.*

## Course Elements

The "module" folders described above should contain `.md` files. These files represent course elements (e.g., discussion boards, pages, assignments, ~~quizzes~~ (work in progress)). Authors should write their course material using Markdown in these files. Here is an example of a typical `.md` file defining a Page course element:

```md
---
title: Welcome!
type: page
---

Hello! Welcome to the course. Here are some useful links you might need:

- [Search Engine](https://google.com)
- [Google Drive](https://drive.google.com)

If you have any questions, please reach out to <jonstapleton@codevirginia.org>!

> The material in this course was developed for [CodeVA](https://codevirginia.org) and is licensed under CC-BY-SA-NC
```

Each `.md` file must define several metadata fields in a "frontmatter" section at the top of the file. The section on **Frontmatter** below provides details about these requirements.

In addition to standard Markdown syntax, `polyscribe-canvas` supports several "directives" which provide enhanced user interfaces for common HTML snippets authors often wish to include in their course elements. You can read about directives in the **Directives** section below.

### Frontmatter

The `polyscribe-canvas` tool requires that authors create "frontmatter" in their `.md` files. Each file must define the following fields:

| Field | Description | Example |
| ----- | ----------- | ------- |
| `title` | The title of the course element, which is displayed as clickable text when the course element is listed (e.g., on the *Modules* page) | `title: Welcome!` |
| `type` | The type of course element `polyscribe-canvas` should create when rendering the `.md` file. Currently, `page`, `assignment`, and `discussion` are supported. | `type: page` |

<!--TODO: There are also some optional fields authors can define for their assignments:

| `published` | A `boolean` value indicating whether or not the element should be published on upload. Defaults to `false` for new assignments. | `published: false` | -->

For `page` elements, the two fields above are the only required ones. There are additional fields available for `assignment` and `discussion` types.

#### Assignment Frontmatter

When creating assignments, use frontmatter fields to configure assignment settings. You can define the following fields for `assignment`s in addition to the two required `title` and `type` fields:

| Field              | Type     | Required? | Default        | Allowed Values |
| ------------------ | -------- | --------- | -------------- | -------------- |
| `submission_types` | `array`  | ❌        | `[ "none" ]`   | `online_quiz`, `none`, `on_paper`, `discussion_topic`, `online_upload`, `online_text_entry`, `online_url` |
| `points_possible`  | `number` | ❌        | `0`            | any positive number (or zero) |
| `grading_type`     | `string` | ❌        | `"not_graded"` | `pass_fail`, `percent`, `letter_grade`, `gpa_scale`, `points`, `not_graded` |

#### Discussion Frontmatter

When creating discussion boards, use frontmatter fields to configure discussion settings. You can define the following fields for `discussions`s in addition to the two required `title` and `type` fields:

| Field                  | Type      | Required? | Default        | Allowed Values |
| ---------------------- | --------- | --------- | -------------- | -------------- |
| `discussion_type`      | `string`  | ❌        | `threaded`     | `side_comment`, `threaded`, `not_threaded` |
| `allow_rating`         | `boolean` | ❌        | `true`         | `true`, `false` |
| `require_initial_post` | `boolean` | ❌        | `true`         | `true`, `false` |
| `assignment`           | `object`  | ❌        | `null`         | See `assignment` frontmatter for values |

### Directives

In addition to supporting all the standard GitHub-flavored Markdown elements (read more about these [here](https://github.github.com/gfm/)), `polyscribe` includes support for several common elements that authors may want to include in their course elements. These "directives" are abstractions of more complex HTML structures compatible with Canvas's strict requirements.

There are two kinds of directives--directives preceded by `::` and directives preceded by `:::`. Directives with `::`, like the `::youtube` directive, take up one line and do not have content within them. Directives like `:::collapse`, with `:::`, have content within them. Read more about the two kinds of directives below.

#### The `::youtube` Directive

The `::youtube` directive represents an embedded YouTube video. Below, you can find an example and a screenshot of how the directive appears on Canvas:

```md
::youtube[A placeholder video]{#14I8C6FKINQ}

Watch the video above to learn some stuff!
```

![A screenshot of the ::youtube directive in context](docs/youtube-directive.png)

The `::youtube` directive requires you to pass the YouTube video ID within the `{}` following a `#`, like so:

```md
::youtube[A description of the video]{#<insert ID here>}
```

To find the ID of a YouTube video, visit it on [YouTube.com](https://youtube.com) and look at the URL. The ID is the string of symbols following the `https://youtube.com/watch?v=` part of the URL. For example, The ID for the video at [https://www.youtube.com/watch?v=14I8C6FKINQ](https://www.youtube.com/watch?v=14I8C6FKINQ) is `14I8C6FKINQ`.

#### The `:::collapse` Directive

The `:::collapse` directive represents a collapsable section triggered by a button. Below, you can find an example and a screenshot of how the directive appears on Canvas:

```md
Click the button below to reveal a message!

:::collapse{title="Button Text"}
Lorem ipsum
:::
```
Here's what the `:::collapse` looks like when it's closed:

![A screenshot of the closed :::collapse directive in context](docs/collapse-closed.png)

...and here's what it looks like when it's open:

![A screenshot of the open :::collapse directive in context](docs/collapse-open.png)

The `:::collapse` requires you to pass some text to display in the button to a `title` parameter between the `{}`. Be sure to enclose the button text in quotation marks. 

You should write content you would like the page to reveal when the user clicks the button after the `:::collapse` line using Markdown syntax. Fence the hidden content with a closing `:::` on a new line after the last piece of Markdown, like so:

```md
:::collapse{title="<button text goes here>"}
<!-- <Markdown content goes here> -->
:::
<!-- ^^ ending fence -->
```

<!-- #### TODO: The `scratch` Directive -->

<!-- TODO: ```
::scratch[Description]{#id}
``` -->

#### The `:::code-example` Directive

The `:::code-example` directive represents one or more `<pre />` elements paired with an image, often a screenshot of the code's output. Here is a simple example of a `:::code-example` and a screenshot of it's output on Canvas:

```md
Check out the program below!

:::code-example{src="tree-and-sun.png" alt="The output of the program above, displaying a sun and a tree rendered using simple shapes"}
```java
// colors
int yellow = color(245, 220, 32);
int green = color(46, 179, 93);
int brown = color(92, 81, 59);

void setup() {
    size(800,600);
}

void draw() {
    // sun
    fill(yellow);
    circle(150, 150, 100);

    // tree
    fill(brown);
    rect(400, 300, 20, 100);
    fill(green);
    circle(410, 310, 100);
}
```
:::
```

![A screenshot of the :::code-example directive as it appears on Canvas](docs/code-example-notabs.png)

The `alt` parameter provides the alt text for the image. The default value for the `alt` parameter is "The output of the program above". If you do not pass a `src` value, the `<pre />` section will take up the whole width of the element.

The `:::code-example` directive also has support for tabs, providing a way for authors to write code examples spread across multiple files. To activate this feature, pass the `tabs` parameter within the `{}` following `:::code-example`. Here's an example:

```md
Check out the program below!

:::code-example{src="obj-tree.png" tabs}
::name[Main]
```java
// MAIN PROGRAM
// global variables
Tree myTree; // declare Tree object variable

void setup() {
    size(400,400);

     myTree = new Tree();  // initalize Tree object variable
}

void draw() {
     myTree.display();    // call display method with attributes of `myTree`    
}
```

::name[Tree]
```java
// OBJECT BLUEPRINT
class Tree {
   int green = color(46, 179, 93);
   int brown = color(92, 81, 59);
   float x = 200;
   float y = 200;
   float size = 100;

   void display() {
      fill(brown);
      rect(x, y, size / 5, size);
      fill(green);
      circle(x+10, y+10, size);
   }
}
```
:::
```

![A screenshot of the :::code-example directive with tabs as it appears on Canvas](docs/code-example-tabs.png)

Each fenced code snippet should be preceded by a `::name` directive. The text passed to the `[]` after the directive sets the text displayed in each tab.

