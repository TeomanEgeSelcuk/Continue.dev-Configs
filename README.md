# Continue Configuration Guide for VS Code

This guide outlines how to set up and configure Continue in VS Code, leveraging TypeScript, environment variables, and custom JSON configurations. Follow these steps to ensure your development environment is properly set up.

## Table of Contents

1. [Setting Up TypeScript Configuration (`tsconfig.json`)](#1-setting-up-typescript-configuration-tsconfigjson)
2. [Setting Up Environment Variables](#2-setting-up-environment-variables)
3. [Creating `global.d.ts`](#3-creating-globaldts)
4. [Creating or Updating `config.ts`](#4-creating-or-updating-configts)
5. [Configuring `config.json`](#5-configuring-configjson)

## 1. Setting Up TypeScript Configuration (`tsconfig.json`)

### Steps:

1. **Navigate to the Continue Project Directory**: Open your terminal or file explorer and navigate to the Continue configuration directory, usually located at `~/.continue/`.

2. **Create or Update `tsconfig.json`**:
   - If `tsconfig.json` doesn’t exist, create this file in the `.continue` directory.
   - Add the following configuration:

     ```json
     {
       "compilerOptions": {
         "target": "ESNext",
         "useDefineForClassFields": true,
         "lib": [
           "DOM",
           "DOM.Iterable",
           "ESNext"
         ],
         "allowJs": true,
         "skipLibCheck": true,
         "esModuleInterop": false,
         "allowSyntheticDefaultImports": true,
         "strict": true,
         "forceConsistentCasingInFileNames": true,
         "module": "System",
         "moduleResolution": "Node",
         "noEmit": false,
         "noEmitOnError": false,
         "outFile": "./out/config.js",
         "typeRoots": [
           "./node_modules/@types",
           "./types"
         ]
       },
       "include": [
         "./config.ts",
         "./global.d.ts"
       ],
       "types": ["node"]
     }
     ```

   - **Save** the file.

### Explanation:

- **Compiler Options**: These options dictate how TypeScript should compile your code.
- **Include**: Specifies the files to be included in the TypeScript compilation.
- **Types**: Ensures Node.js type definitions are included.

---

## 2. Setting Up Environment Variables

You need to define the `OPENROUTER_API_KEY` and `VOYAGE_AI_API_KEY` environment variables on your system to securely manage API keys.

### Windows:

1. **Access Environment Variables**:
   - Press `Win + R`, type `sysdm.cpl`, and press Enter.
   - Navigate to the **Advanced** tab and click on **Environment Variables**.

2. **Add the Variables**:
   - Under **User variables** or **System variables**, click **New**.
   - Enter `OPENROUTER_API_KEY` as the name and paste your OpenRouter API key as the value.
   - Repeat the process for `VOYAGE_AI_API_KEY`.

### macOS/Linux:

1. Open your terminal.
2. Add the following lines to your shell configuration file (e.g., `.bashrc` or `.zshrc`):

   ```bash
   export OPENROUTER_API_KEY="your-openrouter-api-key"
   export VOYAGE_AI_API_KEY="your-voyage-api-key"
   ```

3. Save the file and run `source ~/.bashrc` or `source ~/.zshrc` to apply the changes.

---

## 3. Creating `global.d.ts`

This file ensures TypeScript recognizes your environment variables.

### Steps:

1. **Create `global.d.ts`**:
   - In the `.continue` directory, create a file named `global.d.ts`.

2. **Add the Following Content**:

   ```ts
   declare var process: {
       env: {
         OPENROUTER_API_KEY: string;
         VOYAGE_AI_API_KEY: string;
       };
     };
   ```

3. **Save** the file.

---

## 4. Creating or Updating `config.ts`

This file handles model configuration and other settings using environment variables.

### Steps:

1. **Create `config.ts`**:
   - In the `.continue` directory, create a file named `config.ts`.

2. **Add the Following Configuration**:

   ```ts
   export function modifyConfig(config: Config): Config {
     // Retrieve the API keys from the environment variables
     const openRouterApiKey = process.env.OPENROUTER_API_KEY;
     const voyageApiKey = process.env.VOYAGE_AI_API_KEY;

     // Ensure API keys are set
     if (!openRouterApiKey) {
       throw new Error("OPENROUTER_API_KEY is not defined in environment variables");
     }
     if (!voyageApiKey) {
       throw new Error("VOYAGE_AI_API_KEY is not defined in environment variables");
     }

     // Configure models and other settings
     config.models = [
       {
         title: "OpenRouter deepseek-coder",
         provider: "openai",
         model: "deepseek/deepseek-coder",
         apiBase: "https://openrouter.ai/api/v1",
         contextLength: 128000,
         apiKey: openRouterApiKey
       }
     ];

     config.tabAutocompleteModel = {
       title: "Codestral Mamba",
       provider: "openai",
       apiBase: "https://openrouter.ai/api/v1",
       model: "mistralai/codestral-mamba",
       contextLength: 250000,
       apiKey: openRouterApiKey
     };

     config.embeddingsProvider = {
       provider: "openai",
       model: "voyage-code-2",
       apiBase: "https://api.voyageai.com/v1/",
       apiKey: voyageApiKey
     };

     config.reranker = {
       name: "voyage",
       params: {
         apiKey: voyageApiKey
       }
     };

     // Return the modified config
     return config;
   }
   ```

3. **Save the file**.

---

## 5. Configuring `config.json`

This file defines custom commands, context providers, slash commands, and more.

### Steps:

1. **Create `config.json`**:
   - Navigate to the `.continue` directory in your user profile (e.g., `~/.continue/` on macOS/Linux or `%userprofile%\.continue` on Windows).
   - If `config.json` doesn’t exist, create the file.

2. **Copy and Paste the Provided Content**:

   ```json
    {
    // link for the config.json files documentation: (https://docs.continue.dev/reference/config)
    "customCommands": [
        {
        "name": "test",
        "prompt": "{{{ input }}}\n\nWrite a comprehensive set of unit tests for the selected code. It should setup, run tests that check for correctness including important edge cases, and teardown. Ensure that the tests are complete and sophisticated. Give the tests just as chat output, don't edit any file.",
        "description": "Write unit tests for highlighted code"
        }
    ],
    "contextProviders": [
        {
        "name": "code",
        "params": {}
        },
        {
        "name": "docs",
        "params": {}
        },
        {
        "name": "diff",
        "params": {}
        },
        {
        "name": "terminal",
        "params": {}
        },
        {
        "name": "problems",
        "params": {}
        },
        { "name": "tree" },
        {
        "name": "folder",
        "params": {}
        },
        {
        "name": "codebase",
        "params": {}
        },
        /*
        Example Usage:
        This context provider connects to a PostgreSQL database to provide context to the model. 
        It allows the model to reference data within the database as part of its responses.

        Params Example:
        "name": "database",
        "params": {
            "connections": [
            {
                "name": "examplePostgres",
                "connection_type": "postgres",
                "connection": {
                "user": "username",
                "host": "localhost",
                "database": "exampleDB",
                "password": "yourPassword",
                "port": 5432
                }
            }
            ]
        }

        How to use:
        - Add this `database` context provider in your `config.json`.
        - When you type "@database", Continue will pull relevant data from the specified PostgreSQL database.

        Documentation Link:
        (https://docs.continue.dev/customization/context-providers)
        */
    ],
    "slashCommands": [
        {
        "name": "edit",
        "description": "Edit selected code"
        },
        {
        "name": "comment",
        "description": "Write comments for the selected code"
        },
        {
        "name": "share",
        "description": "Export the current chat session to markdown"
        /*
        Example Usage:
        The `/share` slash command exports your chat session to a markdown file.

        Params Example:
        "params": {
            "outputDir": "~/.continue/session-transcripts"
        }

        How to use:
        - Type "/share" in the chat to trigger this command.
        - The session transcript will be saved to the specified directory `~/.continue/session-transcripts`.
        - You can adjust the `outputDir` parameter to specify a different directory for saving the markdown files.

        Documentation Link:
        (https://docs.continue.dev/customization/slash-commands)
        */
        },
        {
        "name": "cmd",
        "description": "Generate a shell command"
        },
        {
        "name": "commit",
        "description": "Generate a git commit message"
        }
    ],
    "docs": []
    /*
    Example Usage:
    The `docs` section allows you to configure external documentation resources that the model can reference.
    Typically, this section is left empty, but you can add entries like the following if needed:

    "docs": [
        {
        "name": "projectDocs",
        "params": {
            "url": "https://docs.example.com",
            "authToken": "your_auth_token_here"
        }
        }
    ]

    How to use:
    - By configuring a documentation source here, you can use the `@docs` command in the chat to pull relevant information from the specified documentation.
    - This is useful for project-specific documentation or external API references.

    Documentation Link:
    (https://docs.continue.dev/customization/overview)
    */
    }
    ```        
---
