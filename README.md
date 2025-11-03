# QRx
> A portable loom protocol small enough to fit in a qr code

![](garden.gif)

`qrx` is a minimalist, generative web environment that runs entirely in a single HTML file. It uses your browser's local storage (IndexedDB) to save "files" and integrates with Large Language Models (LLMs) to generate and modify content. The entire interface is controlled through URL parameters, making it a highly portable and command-driven tool for creation.

## Core Concepts

*   **Single File System:** The entire application is a single `index.html` file.
*   **URL as API:** You control the environment by changing the URL's hash (`#`) and query parameters (`?`).
*   **Local Persistence:** All content is saved directly in your browser's IndexedDB, keyed by the page name in the URL hash.
*   **LLM Integration:** Use the `?p` parameter to send prompts to a configured LLM (e.g., OpenRouter, Ollama) to generate code, text, or entire pages.
*   **Live Preview:** A split-pane view shows the live rendered output in an `iframe` and the raw code in a `textarea`.

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
    ```    This will start a local server (usually at `http://localhost:5173`). Open this URL in your browser to start using `qrx`.

4.  **Build the project:**
    ```bash
    npm run build
    ```
    This command performs several actions:
    *   Builds and bundles the application using Vite.
    *   Aggressively minifies the output into a single `dist/index.html` file.
    *   Generates a QR code from the final HTML content and saves it as `dist/qrx.qrcode.png`.

5.  **Preview the QR Code in your terminal:**
    After a build, you can preview the final QR code directly in your terminal.
    ```bash
    npm run qrcode
    ```

## LLM Configuration

To use the generative features (`?p` parameter), you must configure an LLM provider. This is done in one of three ways, in order of priority:

1.  **URL Parameters (Temporary):** Add `k` (API key), `h` (API hostname), and `m` (API model name) to the URL for a single session.
    *   `http://localhost:5173/?k=YOUR_KEY&m=MODEL_NAME#home`

2.  **Local Storage (Persistent):** Set the values in your browser's developer console. They will be saved for future sessions.
    ```javascript
    localStorage.setItem('k', 'YOUR_API_KEY');
    localStorage.setItem('h', 'YOUR_API_ENDPOINT');
    localStorage.setItem('m', 'MODEL_NAME');
    ```

3.  **First-Use Prompts (Interactive):** If no configuration is found when you first use `?p`, the application will prompt you to enter the API Key and Model. These will then be saved to `localStorage`.

### Example Provider Setups

*   **Ollama (Local):**
    *   **h:** `http://localhost:11434/v1/chat/completions` (Note the `/v1/` path)
    *   **k:** `ollama`
    *   **m:** Any model you have pulled (e.g., `llama3`)

*   **OpenRouter (Cloud):**
    *   **h:** `https://openrouter.ai/api/v1/chat/completions`
    *   **k:** Your OpenRouter API key (e.g., `sk-or-...)`
    *   **m:** Any model from OpenRouter (e.g., `openai/gpt-4o`)

## URL API Reference

The primary way to interact with `qrx` is by manipulating the URL in your browser's address bar.

---

### `#<page_name>` (Hash)

This determines the current "file" you are working on. Content is loaded from and saved to IndexedDB under this key.

*   `#home` is the default page.
*   `#styles.css` refers to a page for CSS styles.
*   Changing the hash in the URL will load the corresponding page.

---

### `?p=<prompt>`

The **prompt** parameter instructs the configured LLM to generate or modify the content of the current page. The LLM's response will completely replace the existing content.

*   **Purpose:** The main engine for generative creation.
*   **Example:** Create a simple counter app on a new page named `counter`.
    ```
    http://localhost:5173/?p=create a simple javascript counter with a button#counter
    ```
*   **Workflow:** After the LLM responds, the content is saved, and the `?p` parameter is automatically removed from the URL as the page reloads.

---

### `?w=<url_encoded_content>`

The **write** parameter directly injects URL-decoded content onto the current page, completely overwriting its previous content. This bypasses the LLM.

*   **Purpose:** Precise, direct control for bootstrapping content or loading exact code.
*   **Example:** Write `<h1>Hello</h1>` to the `direct` page.
    ```
    http://localhost:5173/?w=%3Ch1%3EHello%3C%2Fh1%3E#direct
    ```
*   **Workflow:** The content is written to storage, and the `?w` parameter is removed from the URL as the page reloads.

---

### `?x=<url_encoded_script>`

The **execute** parameter directly executes URL-decoded JavaScript within the context of the preview `iframe`.

*   **Purpose:** The primary mechanism for creating "installer scripts" or "URL packs." It allows a single URL to run commands, write multiple files to storage, or dynamically manipulate the preview without saving any content itself.
*   **Key Behavior:** The script runs inside the sandboxed `iframe`. After execution, the `?x` parameter is removed from the URL *without* a page reload. This allows the script to complete its tasks, such as redirecting the user, without being interrupted.
*   **Example (Simple DOM manipulation):** Change the background of the preview pane to red.
    ```
    http://localhost:5173/?x=document.body.style.background%3D'red'#home
    ```
*   **Example (Installer Script):** Write two separate files (`styles` and `app`) to storage and then navigate to the new app. The `ui` object from the main window can be accessed via `parent.ui`.
    ```
    http://localhost:5173/?x=parent.ui.o('readwrite'%2C'styles'%2C'body%7Bcolor%3Ablue%7D')%3Bparent.ui.o('readwrite'%2C'app'%2C'%3Ch1%3EInstalled%21%3C%2Fh1%3E')%3Blocation.hash%3D'app'%3B#installer
    ```

---

### `?b=<key1>,<key2>,...`

The **boot dependencies** parameter loads content from other saved pages and prepends it to the preview `iframe` before the main page content. This enables modularity and code reuse.

*   **Purpose:** Inject shared CSS, JavaScript libraries, or HTML components without duplicating them in every file. The special key `boot` is always loaded first if it exists.
*   **Example:** Load content from `styles` and `utils` before rendering the `app` page.
    ```
    http://localhost:5173/?b=styles,utils#app
    ```*   **Workflow:** The content of `boot` (if present), `styles`, `utils`, and finally `app` are combined in the `iframe` preview. The editor pane will still only show the content for `app`. The `?b` parameter remains in the URL.

---

### `?k`, `?h`, `?m`

These **environment override** parameters temporarily override the LLM configuration settings for the current session, taking precedence over `localStorage`.

*   **Purpose:** Quickly test different models, endpoints, or API keys.
*   **Example:** Use Gemini Pro for a specific prompt.
    ```
    http://localhost:5173/?m=google/gemini-2.5-pro&p=summarize this page#summary
    ```

## Development Notes

*   **QR Code Size Limit:** The primary constraint of this project is the data capacity of a QR code. The theoretical maximum for a Version 40 QR code is 2,953 bytes with low error correction. The entire `dist/index.html` file must fit within this limit.
*   **Minification:** The `build` process uses `vite` and `html-minifier-terser` to aggressively shrink the final HTML file size. Every byte counts.
*   **Error Correction:** The QR code generation script is set to use the lowest error correction level (`L`) to maximize data capacity.

## License

This project is licensed under the ISC License.
