# Tree-sitter for Raku (for Helix)

This repository provides a [Tree-sitter](https://tree-sitter.github.io/tree-sitter/) parser for the [Raku](https://raku.org/) programming language, specifically adapted for use with the [Helix editor](https://helix-editor.com/).

This project is a fork of the excellent [tree-sitter-perl](https://github.com/tree-sitter-perl/tree-sitter-perl) parser. While the changes made in this fork are minor compared to the incredible effort invested in the original `tree-sitter-perl`, they provide a significantly improved syntax highlighting experience for Raku code in Helix.

The default Tree-sitter parser for Perl is often applied to Raku files in Helix, which can lead to incorrect highlighting. This parser addresses that by making initial Raku-specific syntax adjustments, such as correctly handling hyphens in variable names (e.g., `my $variable-name`).

## Quick Start: Using in Helix

Follow these steps to build the parser and configure Helix to use it for Raku files.

**Prerequisites:**
*   Node.js (v20 or higher) and npm are installed.
*   The `tree-sitter` CLI. If you don't have it, you can install it locally for the project with `npm install tree-sitter-cli`.

### 1. Clone and Build the Parser

First, clone this repository to your local machine and generate the parser files.

```bash
# Clone the repository
git clone https://github.com/acrion/tree-sitter-raku.git
cd tree-sitter-raku

# Generate the parser C code
npx tree-sitter generate
```

### 2. Configure Helix

Next, you need to tell Helix where to find the grammar and its associated query files.

1.  **Copy the Query Files:**
    The query files (`highlights.scm`, `indents.scm`, etc.) determine how Helix uses the parser for syntax highlighting, indentation, and more.

    ```bash
    # Create the necessary directory if it doesn't exist
    mkdir -p ~/.config/helix/runtime/queries/raku

    # Copy the queries from this repository
    cp -r queries/* ~/.config/helix/runtime/queries/raku/
    ```

2.  **Add the Grammar to `languages.toml`:**
    You need to register the new grammar in your Helix language configuration file (`~/.config/helix/languages.toml`). Add the following content to it. If the file doesn't exist, create it.

    ```toml
    [[language]]
    name = "raku"
    scope = "source.raku"
    file-types = ["raku", "rakumod", "rakutest", "rakudoc", "nqp", "p6", "pl6", "pm6"]
    shebangs = ["raku", "perl6"]
    comment-token = "#"
    indent = { tab-width = 4, unit = "    " }

    [[grammar]]
    name = "raku"
    source = { git = "https://github.com/acrion/tree-sitter-raku", rev = "main" }
    ```
### 3. Build and Verify in Helix

Finally, have Helix build the grammar and check that everything is configured correctly.

```bash
# Tell Helix to build any newly configured grammars
helix --grammar build

# Check the health of your Raku configuration
helix --health raku
```

You should see checkmarks (`✓`) for the Tree-sitter parser and its queries, indicating a successful setup.

## Key Changes from Upstream

*   **Language Name:** The grammar has been renamed from `perl` to `raku` throughout the project.
*   **Raku Syntax:** An initial adaptation was made in `lib/unicode_ranges.js` and `src/grammar.json` to allow hyphens within identifiers, a common feature in Raku.
*   **Helix Integration:**
    *   The `queries` for highlighting, indentation, folding, and text objects were copied from the [Helix repository](https://github.com/helix-editor/helix) (specifically the `perl` queries) and adapted for this parser.
    *   A sample `languages.toml` configuration is provided for easy integration.

## Credits

This project would not be possible without the work of others:

*   **[tree-sitter-perl](https://github.com/tree-sitter-perl/tree-sitter-perl):** The vast majority of the parsing logic comes from this project. Thank you to its maintainers and contributors.
*   **[Helix Editor](https://github.com/helix-editor/helix):** The language query files (`.scm`) are based on the ones provided by the Helix project for its built-in Perl support.

## For Developers

If you want to contribute to the parser itself, the process is straightforward.

#### Generating the Parser
After making any changes to the grammar definition in `grammar.js`, you must regenerate the C source code:
```bash
npx tree-sitter generate
```

#### Running Tests
Tests are located in the `/test/corpus` directory. You can run them using the Tree-sitter CLI:
```bash
npx tree-sitter test
```

---

## Bonus: Setting Up the Raku Language Server (RakuNavigator) in Helix

For a full IDE-like experience, you can complement the Tree-sitter parser with a Language Server. [RakuNavigator](https://github.com/bscan/RakuNavigator) is a great option for Raku. Here is how to set it up for Helix.

### 1. Install RakuNavigator

```bash
# Choose a directory for third-party projects
cd /path/to/your/projects

# Clone the RakuNavigator repository
git clone https://github.com/bscan/RakuNavigator.git
cd RakuNavigator/server

# Install dependencies
npm install

# Install TypeScript to compile the server
npm install -g typescript

# Compile the server
tsc
```

### 2. Configure Helix for the Language Server

Now, add the following configuration to your `~/.config/helix/languages.toml` file.

**Important:** You must replace `/path/to/your/projects` with the actual absolute path where you cloned `RakuNavigator`.

```toml
[language-server.raku-navigator]
command = "node"
args = ["/path/to/your/projects/RakuNavigator/server/out/server.js", "--stdio"]
```

### 3. Verify the Language Server Setup

Run the Helix health check again for Raku:
```bash
helix --health raku
```

You should now see the `raku-navigator` language server listed with a checkmark.

#### Understanding the `helix --health` Output

When you see the following output:
```
Configured language servers:
  ✓ raku-navigator: /usr/bin/node
```

This is what it means:
*   `✓ raku-navigator:` Helix has successfully read the `[language-server.raku-navigator]` section from your `languages.toml`.
*   `/usr/bin/node`: This is the resolved path to the `command` you specified. Helix confirms it can find the `node` executable. The actual server logic is in the JavaScript file (`out/server.js`) that you passed as an argument, which `node` will execute to start the Language Server Protocol (LSP) communication.
