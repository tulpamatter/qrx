# QRx
> A portable loom protocol small enough to fit in a qr code

![](garden.gif)

`qrx` is a minimalist, generative web environment that runs entirely in a single HTML file. It uses your browser's IndexedDB to save "files" and integrates with Large Language Models (LLMs) to create and modify content.

The entire system is a reactive, stateful application controlled by a **sequential command stream** in the URL. It updates live as you edit the address bar, turning it into a powerful, shareable command line for orchestrating agentic workflows.

## Quick Start

### Prerequisites

*   [Node.js](https://nodejs.org/) (which includes npm)
*   [Git](https://git-scm.com/)

### Installation & Usage

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/tulpamatter/qrx.git
    cd qrx
    ```

2.  **Install dependencies:**
    ```bash
    npm install
    ```

3.  **Run the development server:**
    ```bash
    npm run dev
    ```
    This will start a local server. Open the provided URL in your browser to start using `qrx`.

4.  **Build the project:**
    ```bash
    npm run build
    ```
    This command builds the application, minifies it into a single `dist/index.html` file, and generates a corresponding QR code named `dist/qrx.qrcode.png`.

## LLM Configuration

To use the generative features (`?p`), you must configure an LLM provider. This is done by setting an API key, hostname, and model name. These values are retrieved in order of priority:

1.  **URL Parameters (Temporary):** Add `k` (API key), `h` (hostname), and `m` (model) to the URL for the current command sequence.
2.  **Local Storage (Persistent):** Set the values in your browser's developer console for them to be saved for future sessions.

```javascript
// Example configuration for Google's Generative AI API
localStorage.setItem('k', 'YOUR_API_KEY');
localStorage.setItem('h', 'generativelanguage.googleapis.com'); // Or any openAI api capable endpoint
localStorage.setItem('m', 'gemini-2.5-flash'); // Or any other compatible model
```

## URL Command Stream API

Interaction with `qrx` is performed by constructing a command stream in the URL's hash. The parameters are executed **sequentially, in the order they appear**. The system maintains a live, in-memory "accumulator" that is transformed by each command.

---

### `#<page_name>`

This determines the initial page to load and the default target for all operations. It's the page that is ultimately rendered to the screen after the command stream finishes.

*   `#home` is the default page if no hash is provided.

---

### Action Commands
These commands perform an operation, often transforming the accumulator.

#### `?w=<content>`
**Write:** Destructively overwrites the content of the current target page with the provided literal string. This is a permanent change.

*   **Example:** `/#settings?w={"theme":"dark"}`

#### `?p=<prompt>`
**Prompt:** Executes a generative prompt. It sends the current, live content of the accumulator to the LLM and overwrites the target page's content with the response. This is a permanent change.

*   **Example:** `/#story?p=continue the story`

#### `?x=<script>`
**Execute:** Executes a JavaScript string. The script has access to the live accumulator (`$a`), the savable source (`$s`), and the database helper functions (`$r`, `o`) for complex, agentic operations.

*   **Example:** `/#home?x=alert($a)`

---

### State-Setting Commands
These commands modify the state for subsequent commands in the stream.

#### `?t=<page_name>`
**Target:** Redirects all subsequent read/write actions (`w`, `p`) to a different page. This is the key to multi-file workflows.
*   `?t` (with no value) resets the target to the main page from the hash.

*   **Example (agent scratchpad):** `/#app?t=scratch&w=thinking...&t&p=based on @scratch, build the app`

#### `?k`, `?h`, `?m`
**Environment:** Sets the API Key (`k`), Host (`h`), and Model (`m`) for any `?p` commands that follow in the stream.

*   **Example:** `/#test?m=gemini-pro&p=run a complex task&m=gemini-2.5-flash&p=summarize the result`

#### `?s=<page_name>`
**System Prompt:** Sets the system prompt for subsequent `?p` commands by loading the content of the specified page.

*   **Example:** `/#code?s=coder-persona&p=write a function`

---

### Composition & Context

#### `?b=<page_name>`
**Boot:** A temporary, runtime-only prepend. It loads the content of `<page_name>` and prepends it to the live accumulator. This is ideal for loading CSS, libraries, or dependencies. **This change is not saved** to the target page's content.

*   **Example:** `/#app?b=styles.css&b=runtime.js`

#### `@<page_name>` (in prompts)
**Inject:** When used inside a `?p` prompt, it injects the content of the specified page directly into the prompt string before sending it to the LLM. This is for providing specific context.

*   **Example:** `/#summary?p=summarize this article: @long-article`