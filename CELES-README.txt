================================================================================
                         CELES ARCHIVAL DOCUMENTATION
================================================================================
Version: 0.1.5
Project: https://github.com/TheServer-lab/Celes
License: SOCL 1.0 (Server-Lab Open-Control License)

--------------------------------------------------------------------------------
1. WHY CELES? (THE PHILOSOPHY)
--------------------------------------------------------------------------------
Celes is a "hardened" markup language built for 50-year digital longevity.
Most modern formats (HTML, PDF, Markdown) rely on complex browser engines or
fragmented "flavors" that may become unreadable as software evolves.

Key Archival Features:
* STRICTNESS: The parser never silently ignores errors[cite: 8]. If a file 
    is corrupted, you will know immediately[cite: 10].
* EXPLICITNESS: Every formatting decision is stated in the file[cite: 11]. 
    There are no hidden defaults[cite: 12].
* SOVEREIGNTY: Every file declares its version (e.g., <!Celes-0.1.5>) on 
    the first line so future systems know exactly how to read it[cite: 15, 16].
* SIMPLICITY: The logic is simple enough that a human can read the raw 
    text tags even without a computer[cite: 3, 7].

--------------------------------------------------------------------------------
2. HOW TO ACCESS THE FILES
--------------------------------------------------------------------------------
You can view .celes files in three ways, depending on your environment:

A. THE HUMAN WAY (Universal Access)
   Open any .celes file in a basic text editor (Notepad, Vim, TextEdit). 
   Because Celes uses named English words for tags instead of cryptic symbols, 
   the content is fully human-readable in its raw state[cite: 4, 7].

B. STANDALONE VIEWER (Portability)
   If a 'celes_view.exe' is included in this archive, you can run it to see 
   a rendered version of the documents without installing any dependencies.

C. BROWSER EXTENSION (Daily Use)
   Install the Celes Browser Extension to view .celes files directly in your 
   web browser with full formatting, images, and video support[cite: 125].

--------------------------------------------------------------------------------
3. HOW TO INSTALL THE CELES TOOLKIT
--------------------------------------------------------------------------------
To render, validate, or create Celes documents on a modern system:

1.  PYTHON PACKAGE (Reference Implementation):
    Ensure Python is installed, then run:
    > pip install celes

2.  VS CODE EXTENSION:
    Search for "Celes" in the VS Code Marketplace for syntax highlighting 
    and live previews[cite: 125].

3.  VALIDATION:
    Use the built-in validator to check for semantic errors (E01-E08) or 
    warnings (W01-W05) before archiving your data [cite: 119-124].

--------------------------------------------------------------------------------
4. FILE STRUCTURE GUIDELINES
--------------------------------------------------------------------------------
For maximum longevity, your archive should follow these rules:
- Keep all media (images, audio, video) in a local folder named /assets/[cite: 61, 93, 97].
- Do not use external URLs for critical information.
- Always include a copy of the 'celes_renderer.py' script in the root folder 
  so future users have the "DNA" to rebuild the viewer[cite: 14, 15].

"The story survives the software."
================================================================================