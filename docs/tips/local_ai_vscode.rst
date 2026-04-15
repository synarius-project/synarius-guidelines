..
   Tips and Tricks — Local AI with VS Code, Ollama, and Cline.

Free Local AI in VS Code with Ollama and Cline
===============================================

This guide shows how to set up the currently strongest free combination of
VS Code, Ollama, and the Cline agent for local AI-assisted development.

We use the **DeepSeek-R1** model because its built-in chain-of-thought reasoning
produces significantly fewer logical errors than a standard language model of the
same size. On reasoning-heavy tasks it is broadly competitive with much larger
cloud-based models.

.. note::

   Everything runs locally on your machine.
   No data leaves your system. No API key or subscription required.

--------------------------------------------------------------------------------

Step 1 — Install Ollama (the local model server)
-------------------------------------------------

Ollama runs AI models as a local HTTP server that other tools (like Cline) can
query.

1. Go to `<https://ollama.com>`_ and download the installer for your OS
   (Windows, macOS, or Linux).
2. Run the installer and start the application.
   A small Ollama icon should appear in your system tray.

--------------------------------------------------------------------------------

Step 2 — Download a model
--------------------------

Open a terminal (**Command Prompt** or **PowerShell** on Windows,
**Terminal** on macOS/Linux) and run one of the following commands.

Use ``ollama pull`` to download the model without starting an interactive chat
session. This is the right command for a one-time setup.

**For machines with 16 GB RAM or more (CPU) or 10 GB+ VRAM (GPU):**

.. code-block:: shell

   ollama pull deepseek-r1:14b

**For laptops with 8 GB RAM/VRAM:**

.. code-block:: shell

   ollama pull deepseek-r1:7b

.. note::

   VRAM requirements depend on quantization.
   The 14b model in 4-bit quantization (Q4, the Ollama default) requires
   approximately 8–10 GB VRAM. A GPU with 8 GB VRAM can therefore run the 14b
   model, but may be slow. 16 GB VRAM runs it comfortably.
   For CPU-only inference, 16 GB RAM is the practical minimum for 14b.

**Alternative — if DeepSeek-R1 is too slow on your hardware:**

.. code-block:: shell

   ollama pull qwen2.5-coder:7b

``qwen2.5-coder:7b`` is noticeably faster than the 14b DeepSeek model and is
optimised specifically for code generation tasks.

Wait for the download to complete before continuing.

--------------------------------------------------------------------------------

Step 3 — Install the Cline extension in VS Code
-------------------------------------------------

We use **Cline** (formerly known as *Claude Dev*) because it is fully compatible
with local, open-source model APIs and provides real agentic capabilities —
reading and writing files, running terminal commands, and making multi-step
decisions — rather than simple autocomplete.

1. Open Visual Studio Code.
2. Click the **Extensions** icon in the left sidebar (four squares).
3. Search for ``Cline``.
4. Click **Install**.

--------------------------------------------------------------------------------

Step 4 — Connect Cline to Ollama
---------------------------------

1. Click the new **Cline** icon in the VS Code sidebar (small robot head).
2. Click the **gear icon** (Settings) at the bottom of the Cline panel.
3. Set the following options:

   .. list-table::
      :header-rows: 1
      :widths: 30 70

      * - Setting
        - Value
      * - **API Provider**
        - ``Ollama``
      * - **Base URL**
        - ``http://localhost:11434`` (Ollama's default; leave as-is)
      * - **Model ID**
        - Select the model you downloaded, e.g. ``deepseek-r1:14b``

4. **Custom Instructions** (optional): in the Custom Instructions field you can
   add a persistent system prompt, for example::

      You are a senior software engineer.
      Write clean, well-structured code with concise inline comments.
      Prefer explicit error handling over silent fallbacks.

--------------------------------------------------------------------------------

Step 5 — Agent-based workflow
------------------------------

Unlike autocomplete tools, Cline operates as an **agent**: it plans a sequence
of actions, proposes each action to you for approval, and then executes it.
Actions include creating and editing files, running terminal commands, and
reading project context.

**Example task:**

   In the Cline chat, type:

   .. code-block:: text

      Create a React web app with a simple dashboard page.
      Save all files into a new folder called "my-app".
      Install the required dependencies via the terminal.

**What happens next:**

1. Cline analyses the task and proposes a plan.
2. Before creating any file or running any command, Cline asks for your approval.
3. Click **Approve** for each step you want to proceed with.
4. Watch the agent build the folder structure, write the files, and run
   ``npm install``.

You remain in control at every step.
Cline never executes an action without your explicit confirmation.

--------------------------------------------------------------------------------

Troubleshooting
---------------

**Cline cannot connect to Ollama**
   Verify that Ollama is running (check the system tray icon).
   Confirm the Base URL is ``http://localhost:11434`` and that the model name in
   the Model ID field exactly matches the name shown by ``ollama list`` in the
   terminal.

**The model is very slow**
   Reduce the model size (e.g. switch from ``14b`` to ``7b``) or try
   ``qwen2.5-coder:7b``. Performance depends heavily on whether inference runs
   on GPU or CPU. Check the Ollama logs to see which device is being used.

**Cline asks for an API key**
   Make sure you selected ``Ollama`` as the API Provider, not ``OpenAI`` or
   ``Anthropic``. Ollama does not require an API key.
