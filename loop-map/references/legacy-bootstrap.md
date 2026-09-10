# Legacy Registry Bootstrap

Use this workflow only when mapping an existing subsystem and `.codex/MODULES.md` does not exist.

1. Treat the requested or user-confirmed module ID as provisional during research. Apply the same ID
   syntax as `$loop-commit`: lowercase ASCII letters and digits, hyphens between words, `/` for
   hierarchy, no leading, trailing, or repeated slash, and no `.` or `..` path segment.
2. Complete full research and adversarial verification before registering the module.
3. Draft a minimal registry containing only modules whose boundaries were verified in this run. Do
   not inventory or guess the rest of the repository. Use this minimum shape:

   ```markdown
   # Module Registry

   ## Modules

   - `<module-id>` — <verified durable boundary>
   ```

4. Show the complete registry draft together with the verified map and write both only after
   explicit approval.
5. Do not stage or commit either file. Tell the user to select the intended paths and use
   `$loop-commit`, which will also reconcile the Knowledge Loop Conventions.
