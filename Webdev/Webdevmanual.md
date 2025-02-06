### :orange[HTML Learning Manual]

Welcome to the exciting world of Web Development! This manual is designed to guide you through the fundamentals of HTML (HyperText Markup Language), the backbone of every webpage you see on the internet. Let's embark on this journey to understand how to structure content for the web.

:blue[Step 1: Introduction to HTML]

HTML is not a programming language; it's a **markup language**. This means HTML uses tags to structure content on a webpage. Think of it as the skeleton of a website, providing the basic framework upon which CSS (styling) and JavaScript (interactivity) are built.

*   **Purpose of HTML:** HTML is used to create the structure and content of web pages. It defines elements like headings, paragraphs, lists, links, images, and more.
*   **HTML Documents:**  HTML documents are plain text files with an `.html` or `.htm` extension. These files are interpreted by web browsers to display web pages.
*   **Basic HTML Structure:** Every HTML document follows a basic structure. Let's look at the fundamental components:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Page Title</title>
</head>
<body>
    <h1>Hello, World!</h1>
    <p>This is a paragraph of text.</p>
</body>
</html>
```

*   **`<!DOCTYPE html>`:**  This declaration defines the document type and version of HTML being used. `<!DOCTYPE html>` is for HTML5, the latest standard.
*   **`<html>`:** The root element of an HTML page. It tells the browser that this is an HTML document. The `lang="en"` attribute specifies the language of the document as English.
*   **`<head>`:** Contains meta-information about the HTML document, such as the title, character set, and links to stylesheets and scripts. This information is not displayed on the webpage itself but is crucial for browser settings and SEO.
    *   **`<meta charset="UTF-8">`:** Specifies the character encoding for the document, UTF-8 is widely used and supports most characters.
    *   **`<meta name="viewport" content="width=device-width, initial-scale=1.0">`:** Configures the viewport for responsive web design, ensuring the page scales correctly on different devices.
    *   **`<title>Page Title</title>`:** Sets the title that appears in the browser's title bar or tab.
*   **`<body>`:** Contains the visible content of the HTML document, such as text, images, links, and other media. This is what users see on the webpage.
    *   **`<h1>Hello, World!</h1>`:**  A level 1 heading, used for main titles. HTML provides headings from `<h1>` to `<h6>` (decreasing in importance).
    *   **`<p>This is a paragraph of text.</p>`:**  Represents a paragraph of text.

:blue[Step 2: HTML Elements and Tags]

HTML documents are composed of **HTML elements**. An HTML element is defined by a **start tag**, some **content**, and an **end tag**.

*   **Tags:** Tags are keywords enclosed in angle brackets (`< >`).
    *   **Start Tag:**  Marks the beginning of an element (e.g., `<h1>`).
    *   **End Tag:** Marks the end of an element and is the same as the start tag but with a forward slash before the element name (e.g., `</h1>`).
*   **Elements:**  An HTML element encompasses everything from the start tag to the end tag, including the content in between. For example, `<h1>Hello, World!</h1>` is an `h1` element.
*   **Empty Elements (Self-Closing Tags):** Some elements do not have content and are called empty elements. These elements usually have a self-closing tag in XHTML, but in HTML5, they are simply written with a start tag. Examples include `<br>` (line break) and `<img>` (image).  While `<br />` or `<img />` is valid in XHTML and older HTML, in HTML5, `<br>` and `<img>` are preferred.

**Common HTML Tags:**

*   **Headings:** `<h1>` to `<h6>` for different levels of headings.
*   **Paragraphs:** `<p>` for paragraphs of text.
*   **Links:** `<a>` (anchor) for creating hyperlinks to other web pages or resources.
    ```html
    <a href="https://www.example.com">Visit Example</a>
    ```
*   **Images:** `<img>` for embedding images.
    ```html
    <img src="image.jpg" alt="Description of the image">
    ```
    *   `src` attribute specifies the path to the image.
    *   `alt` attribute provides alternative text for the image, important for accessibility and SEO.
*   **Lists:**
    *   **Unordered Lists:** `<ul>` (unordered list) and `<li>` (list item) for lists with bullet points.
        ```html
        <ul>
            <li>Item 1</li>
            <li>Item 2</li>
            <li>Item 3</li>
        </ul>
        ```
    *   **Ordered Lists:** `<ol>` (ordered list) and `<li>` (list item) for numbered lists.
        ```html
        <ol>
            <li>First item</li>
            <li>Second item</li>
            <li>Third item</li>
        </ol>
        ```
*   **Divisions and Spans:**
    *   `<div>` (division) is a block-level element used as a container to group other HTML elements. It's often used for layout and styling.
    *   `<span>` is an inline element used as a container for inline content. It's often used to style parts of text within a paragraph.

:blue[Step 3: HTML Attributes]

HTML attributes provide additional information about HTML elements. They are always specified in the **start tag** and usually come in name-value pairs like `name="value"`.

*   **Syntax:** `attribute="value"`
*   **Common Attributes:**
    *   **`id`:**  Provides a unique identifier for an element within the HTML document. IDs are used by CSS and JavaScript to target specific elements.
        ```html
        <p id="intro">This is an introductory paragraph.</p>
        ```
    *   **`class`:** Specifies one or more class names for an element. Classes are used by CSS to apply styles to multiple elements with the same class name.
        ```html
        <p class="highlight">This paragraph is highlighted.</p>
        ```
    *   **`style`:**  Allows you to add inline CSS styles to an element.
        ```html
        <p style="color: blue;">This text is blue.</p>
        ```
        **Note:** While inline styles are possible, it's generally better practice to use external CSS stylesheets for better organization and maintainability.
    *   **`src`:**  Used with `<img>` and `<script>` elements to specify the source (path) of the image or script file.
        ```html
        <img src="images/logo.png" alt="Company Logo">
        ```
    *   **`href`:** Used with `<a>` elements to specify the URL of the link.
        ```html
        <a href="https://www.example.com">Visit Example</a>
        ```
    *   **`alt`:**  Used with `<img>` elements to provide alternative text for images.
    *   **`title`:** Specifies extra information about an element, often displayed as a tooltip when you hover over the element.
        ```html
        <p title="This is a tooltip">Hover over this text.</p>
        ```

:blue[Step 4: HTML Forms]

HTML forms are used to collect user input. They are essential for creating interactive web pages where users can submit data to a server.

*   **`<form>` Element:**  Defines an HTML form.
    *   **`action` attribute:** Specifies the URL where the form data should be sent when submitted.
    *   **`method` attribute:** Specifies the HTTP method used to submit the form data (e.g., `get` or `post`).
*   **Form Elements:** Various input elements are used within a `<form>` to collect different types of user input.
    *   **`<input>`:**  The most versatile form element, used for various input types.
        *   `type="text"`: For single-line text input.
        *   `type="password"`: For password input (text is masked).
        *   `type="email"`: For email input (browser may validate email format).
        *   `type="radio"`: For radio buttons (select one option from many).
        *   `type="checkbox"`: For checkboxes (select zero or more options).
        *   `type="submit"`: For a submit button to send form data.
        *   `type="reset"`: For a reset button to clear form fields.
    *   **`<textarea>`:** For multi-line text input.
    *   **`<button>`:**  For creating clickable buttons (can be used for form submission or other actions).
    *   **`<select>`:** For creating dropdown lists.
        *   `<option>` elements are used within `<select>` to define the options in the dropdown.
    *   **`<label>`:**  Defines a label for form elements, improving accessibility. Use the `for` attribute to link a label to a specific form element's `id`.

**Example Form:**

```html
<form action="/submit-form" method="post">
    <label for="name">Name:</label><br>
    <input type="text" id="name" name="user_name"><br><br>

    <label for="email">Email:</label><br>
    <input type="email" id="email" name="user_email"><br><br>

    <label>Gender:</label><br>
    <input type="radio" id="male" name="gender" value="male">
    <label for="male">Male</label><br>
    <input type="radio" id="female" name="gender" value="female">
    <label for="female">Female</label><br><br>

    <input type="submit" value="Submit">
</form>
```

:blue[Step 5: HTML Tables]

HTML tables are used to display data in a structured grid format of rows and columns.

*   **`<table>`:** Defines an HTML table.
*   **`<tr>`:** Defines a table row.
*   **`<th>`:** Defines a table header cell (usually bold and centered).
*   **`<td>`:** Defines a table data cell (regular data).
*   **`<caption>`:** Defines a table caption (title).

**Example Table:**

```html
<table>
    <caption>Student Grades</caption>
    <thead>
        <tr>
            <th>Name</th>
            <th>Subject</th>
            <th>Grade</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Alice</td>
            <td>Math</td>
            <td>A</td>
        </tr>
        <tr>
            <td>Bob</td>
            <td>Science</td>
            <td>B</td>
        </tr>
        <tr>
            <td>Charlie</td>
            <td>History</td>
            <td>C</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td colspan="3">End of Report</td>
        </tr>
    </tfoot>
</table>
```

*   **`<thead>`:**  Groups the header content in a table.
*   **`<tbody>`:** Groups the body content in a table.
*   **`<tfoot>`:** Groups the footer content in a table.
*   **`colspan` attribute:** Used in `<th>` or `<td>` to make a cell span across multiple columns.

:blue[Step 6: HTML Semantic Elements]

HTML5 introduced semantic elements that clearly describe their meaning to both the browser and the developer. Semantic elements improve accessibility and SEO by providing structure and context to the content.

*   **`<header>`:** Represents the header of a document or section. Typically contains introductory content or navigation.
*   **`<nav>`:** Represents a section of navigation links.
*   **`<main>`:** Represents the main content of the document. There should be only one `<main>` element per page.
*   **`<article>`:** Represents a self-contained composition in a document, like a blog post, news article, or forum post.
*   **`<section>`:** Represents a thematic grouping of content, typically with a heading.
*   **`<aside>`:** Represents content that is tangentially related to the main content, like sidebars, pull quotes, or advertisements.
*   **`<footer>`:** Represents the footer of a document or section. Typically contains information about the author, copyright information, or links to related documents.

**Example using Semantic Elements:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Semantic HTML Example</title>
</head>
<body>

    <header>
        <h1>My Website</h1>
        <nav>
            <ul>
                <li><a href="/">Home</a></li>
                <li><a href="/about">About</a></li>
                <li><a href="/contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <article>
            <h2>Article Title</h2>
            <p>This is the main content of the article.</p>
            <section>
                <h3>Section in Article</h3>
                <p>Details within the section.</p>
            </section>
        </article>

        <aside>
            <h3>Related Links</h3>
            <ul>
                <li><a href="#">Link 1</a></li>
                <li><a href="#">Link 2</a></li>
            </ul>
        </aside>
    </main>

    <footer>
        <p>&copy; 2023 My Website</p>
    </footer>

</body>
</html>
```

:blue[Step 7: HTML Media]

HTML allows you to embed multimedia content like images, videos, and audio into your web pages.

*   **Images (`<img>`):** Already covered in Step 2.
*   **Videos (`<video>`):** Embeds video content.
    *   `src` attribute: Specifies the URL of the video file.
    *   `controls` attribute: Adds video controls (play, pause, volume, etc.).
    *   `width` and `height` attributes: Set the dimensions of the video player.
    *   `<source>` element: Allows specifying multiple video sources in different formats for browser compatibility.
*   **Audio (`<audio>`):** Embeds audio content.
    *   `src` attribute: Specifies the URL of the audio file.
    *   `controls` attribute: Adds audio controls (play, pause, volume, etc.).
    *   `<source>` element: Allows specifying multiple audio sources in different formats.

**Example Media Embedding:**

```html
<h2>Image</h2>
<img src="images/landscape.jpg" alt="Landscape Image" width="500">

<h2>Video</h2>
<video controls width="500">
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
    Your browser does not support the video tag.
</video>

<h2>Audio</h2>
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    Your browser does not support the audio element.
</audio>
```

:blue[Step 8: HTML Best Practices]

*   **Semantic HTML:** Use semantic elements to structure your content meaningfully. This improves accessibility, SEO, and code maintainability.
*   **Accessibility (A11y):**  Design your HTML with accessibility in mind.
    *   Use proper heading structure (`h1` to `h6`).
    *   Provide `alt` text for images.
    *   Use labels for form controls.
    *   Ensure sufficient color contrast.
*   **Validation:** Validate your HTML code to ensure it is correctly written and follows standards. You can use online HTML validators like the [W3C Markup Validation Service](https://validator.w3.org/).
*   **Code Formatting:** Keep your HTML code clean, well-formatted, and consistently indented for readability.
*   **Separation of Concerns:** Separate structure (HTML), presentation (CSS), and behavior (JavaScript). Use external CSS stylesheets and JavaScript files instead of inline styles and scripts as much as possible.
*   **Responsive Design:** Use the viewport meta tag and consider responsive design principles to ensure your website works well on different screen sizes.

This manual provides a solid foundation in HTML. To further enhance your skills, practice writing HTML code, explore more advanced topics, and refer to the resources below. Happy coding!

**Helpful Resources:**

*   **MDN Web Docs HTML:** [https://developer.mozilla.org/en-US/docs/Web/HTML](https://developer.mozilla.org/en-US/docs/Web/HTML) - Comprehensive and detailed documentation on HTML from Mozilla.
*   **W3Schools HTML Tutorial:** [https://www.w3schools.com/html/](https://www.w3schools.com/html/) - A beginner-friendly tutorial with examples and exercises.
*   **HTML Living Standard:** [https://html.spec.whatwg.org/multipage/](https://html.spec.whatwg.org/multipage/) - The official HTML specification for in-depth understanding.
*   **Can I use... Support tables for HTML5, CSS3, etc:** [https://caniuse.com/](https://caniuse.com/) - Check browser compatibility for different HTML features.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTML Learning Manual Example</title>
</head>
<body>

    <header>
        <h1>:orange[HTML Learning Manual]</h1>
        <nav>
            <ul>
                <li><a href="#introduction">:blue[Step 1: Introduction to HTML]</a></li>
                <li><a href="#elements-tags">:blue[Step 2: HTML Elements and Tags]</a></li>
                <li><a href="#attributes">:blue[Step 3: HTML Attributes]</a></li>
                <li><a href="#forms">:blue[Step 4: HTML Forms]</a></li>
                <li><a href="#tables">:blue[Step 5: HTML Tables]</a></li>
                <li><a href="#semantic-elements">:blue[Step 6: HTML Semantic Elements]</a></li>
                <li><a href="#media">:blue[Step 7: HTML Media]</a></li>
                <li><a href="#best-practices">:blue[Step 8: HTML Best Practices]</a></li>
                <li><a href="#resources">Helpful Resources</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <section id="introduction">
            <h3>:blue[Step 1: Introduction to HTML]</h3>
            <p>HTML is not a programming language; it's a <strong>markup language</strong>. ... (Content from Step 1)</p>
        </section>

        <section id="elements-tags">
            <h3>:blue[Step 2: HTML Elements and Tags]</h3>
            <p>HTML documents are composed of <strong>HTML elements</strong>. ... (Content from Step 2)</p>
            <!-- Example code from Step 2 -->
            <pre><code>&lt;ul&gt;
    &lt;li&gt;Item 1&lt;/li&gt;
    &lt;li&gt;Item 2&lt;/li&gt;
    &lt;li&gt;Item 3&lt;/li&gt;
&lt;/ul&gt;</code></pre>
        </section>

        <section id="attributes">
            <h3>:blue[Step 3: HTML Attributes]</h3>
            <p>HTML attributes provide additional information about HTML elements. ... (Content from Step 3)</p>
            <!-- Example code from Step 3 -->
            <pre><code>&lt;p id="intro"&gt;This is an introductory paragraph.&lt;/p&gt;</code></pre>
        </section>

        <section id="forms">
            <h3>:blue[Step 4: HTML Forms]</h3>
            <p>HTML forms are used to collect user input. ... (Content from Step 4)</p>
            <!-- Example code from Step 4 -->
            <pre><code>&lt;form action="/submit-form" method="post"&gt;
    &lt;label for="name"&gt;Name:&lt;/label&gt;&lt;br&gt;
    &lt;input type="text" id="name" name="user_name"&gt;&lt;br&gt;&lt;br&gt;

    &lt;label for="email"&gt;Email:&lt;/label&gt;&lt;br&gt;
    &lt;input type="email" id="email" name="user_email"&gt;&lt;br&gt;&lt;br&gt;

    &lt;label&gt;Gender:&lt;/label&gt;&lt;br&gt;
    &lt;input type="radio" id="male" name="gender" value="male"&gt;
    &lt;label for="male"&gt;Male&lt;/label&gt;&lt;br&gt;
    &lt;input type="radio" id="female" name="gender" value="female"&gt;
    &lt;label for="female"&gt;Female&lt;/label&gt;&lt;br&gt;&lt;br&gt;

    &lt;input type="submit" value="Submit"&gt;
&lt;/form&gt;</code></pre>
        </section>

        <section id="tables">
            <h3>:blue[Step 5: HTML Tables]</h3>
            <p>HTML tables are used to display data in a structured grid format ... (Content from Step 5)</p>
            <!-- Example code from Step 5 -->
            <pre><code>&lt;table&gt;
    &lt;caption&gt;Student Grades&lt;/caption&gt;
    &lt;thead&gt;
        &lt;tr&gt;
            &lt;th&gt;Name&lt;/th&gt;
            &lt;th&gt;Subject&lt;/th&gt;
            &lt;th&gt;Grade&lt;/th&gt;
        &lt;/tr&gt;
    &lt;/thead&gt;
    &lt;tbody&gt;
        &lt;tr&gt;
            &lt;td&gt;Alice&lt;/td&gt;
            &lt;td&gt;Math&lt;/td&gt;
            &lt;td&gt;A&lt;/td&gt;
        &lt;/tr&gt;
        &lt;tr&gt;
            &lt;td&gt;Bob&lt;/td&gt;
            &lt;td&gt;Science&lt;/td&gt;
            &lt;td&gt;B&lt;/td&gt;
        &lt;/tr&gt;
        &lt;tr&gt;
            &lt;td&gt;Charlie&lt;/td&gt;
            &lt;td&gt;History&lt;/td&gt;
            &lt;td&gt;C&lt;/td&gt;
        &lt;/tr&gt;
    &lt;/tbody&gt;
    &lt;tfoot&gt;
        &lt;tr&gt;
            &lt;td colspan="3"&gt;End of Report&lt;/td&gt;
        &lt;/tr&gt;
    &lt;/tfoot&gt;
&lt;/table&gt;</code></pre>
        </section>

        <section id="semantic-elements">
            <h3>:blue[Step 6: HTML Semantic Elements]</h3>
            <p>HTML5 introduced semantic elements that clearly describe their meaning ... (Content from Step 6)</p>
            <!-- Example code from Step 6 -->
            <pre><code>&lt;!DOCTYPE html&gt;
&lt;html lang="en"&gt;
&lt;head&gt;
    &lt;meta charset="UTF-8"&gt;
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"&gt;
    &lt;title&gt;Semantic HTML Example&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;

    &lt;header&gt;
        &lt;h1&gt;My Website&lt;/h1&gt;
        &lt;nav&gt;
            &lt;ul&gt;
                &lt;li&gt;&lt;a href="/"&gt;Home&lt;/a&gt;&lt;/li&gt;
                &lt;li&gt;&lt;a href="/about"&gt;About&lt;/a&gt;&lt;/li&gt;
                &lt;li&gt;&lt;a href="/contact"&gt;Contact&lt;/a&gt;&lt;/li&gt;
            &lt;/ul&gt;
        &lt;/nav&gt;
    &lt;/header&gt;

    &lt;main&gt;
        &lt;article&gt;
            &lt;h2&gt;Article Title&lt;/h2&gt;
            &lt;p&gt;This is the main content of the article.&lt;/p&gt;
            &lt;section&gt;
                &lt;h3&gt;Section in Article&lt;/h3&gt;
                &lt;p&gt;Details within the section.&lt;/p&gt;
            &lt;/section&gt;
        &lt;/article&gt;

        &lt;aside&gt;
            &lt;h3&gt;Related Links&lt;/h3&gt;
            &lt;ul&gt;
                &lt;li&gt;&lt;a href="#"&gt;Link 1&lt;/a&gt;&lt;/li&gt;
                &lt;li&gt;&lt;a href="#"&gt;Link 2&lt;/a&gt;&lt;/li&gt;
            &lt;/ul&gt;
        &lt;/aside&gt;
    &lt;/main&gt;

    &lt;footer&gt;
        &lt;p&gt;&amp;copy; 2023 My Website&lt;/p&gt;
    &lt;/footer&gt;

&lt;/body&gt;
&lt;/html&gt;</code></pre>
        </section>

        <section id="media">
            <h3>:blue[Step 7: HTML Media]</h3>
            <p>HTML allows you to embed multimedia content like images, videos, and audio ... (Content from Step 7)</p>
            <!-- Example code from Step 7 -->
            <pre><code>&lt;h2&gt;Image&lt;/h2&gt;
&lt;img src="images/landscape.jpg" alt="Landscape Image" width="500"&gt;

&lt;h2&gt;Video&lt;/h2&gt;
&lt;video controls width="500"&gt;
    &lt;source src="video.mp4" type="video/mp4"&gt;
    &lt;source src="video.webm" type="video/webm"&gt;
    Your browser does not support the video tag.
&lt;/video&gt;

&lt;h2&gt;Audio&lt;/h2&gt;
&lt;audio controls&gt;
    &lt;source src="audio.mp3" type="audio/mpeg"&gt;
    &lt;source src="audio.ogg" type="audio/ogg"&gt;
    Your browser does not support the audio element.
&lt;/audio&gt;</code></pre>
        </section>

        <section id="best-practices">
            <h3>:blue[Step 8: HTML Best Practices]</h3>
            <p><strong>Semantic HTML:</strong> Use semantic elements to structure your content meaningfully. ... (Content from Step 8)</p>
        </section>

        <section id="resources">
            <h3>Helpful Resources</h3>
            <ul>
                <li><a href="https://developer.mozilla.org/en-US/docs/Web/HTML">MDN Web Docs HTML</a></li>
                <li><a href="https://www.w3schools.com/html/">W3Schools HTML Tutorial</a></li>
                <li><a href="https://html.spec.whatwg.org/multipage/">HTML Living Standard</a></li>
                <li><a href="https://caniuse.com/">Can I use... Support tables for HTML5, CSS3, etc</a></li>
            </ul>
        </section>
    </main>

    <footer>
        <p>&copy; 2023 HTML Learning Manual</p>
    </footer>

</body>
</html>
```