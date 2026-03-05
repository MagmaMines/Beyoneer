Privacy Policy.
How Beyoneer v1.14 Pro handles your data and code.

Beyoneer v1.14 Pro is a Hybrid IDE. While we remain committed to Local Sovereignty, certain advanced features require the secure transmission of specific data to remote processing environments. This policy outlines exactly what stays and what goes.

1. Local-First Workspace
By default, 90% of your IDE functions entirely within your browser's IndexedDB database.

Source Code Storage: Your project files, folder structures, and assets are stored in a private browser partition. Magma Mines has no server-side access to this database.
Redirection Preview: The internal preview system uses virtual memory addresses. No files are uploaded to any server for the purpose of live-previewing your work.
2. BeyoAgent AI Transparency
The BeyoAgent utilizes Puter.js and Claude-3.7 models to provide intelligent coding assistance.

Data Sent: When you chat with the agent or request a bug fix, the relevant code snippets and your chat history are sent to third-party AI models for processing.
No Model Training: Per our agreement with API providers, your code is processed for immediate inference and is not used to train future public models.
Security: We recommend omitting API keys, passwords, or sensitive personal data from your source code comments before engaging the AI.
3. Remote Execution (Piston)
Running non-web languages like C, Rust, or Python involves the Piston API.

Transmission: Your source code is sent to a remote sandboxed container where it is compiled and executed.
Deletion: Execution environments are ephemeral. Your code is discarded immediately after the program output is returned to your IDE console.
 Privacy-First Engineering
If you choose to work strictly with HTML, CSS, and JS, you can use Beyoneer 100% Offline. In this state, zero bytes of data leave your device.

4. Firebase Hosting & Auth
Using the optional Hosting and Auth features involves Google Firebase.

Authentication: If you sign in via Google or Email, your profile information (UID, Email, Avatar) is stored securely by Firebase.
Public Sites: Any site you "Publish" to the Beyoneer Network is stored in our database and made publicly accessible. You retain ownership but grant us a license to host the content.
5. Your Right to Erasure
You own your data. You can delete it at any time by:

Local Clear: Clearing your browser's "Site Data" for the Beyoneer domain.
Hosting Deletion: Deleting published sites directly from the "Host" management panel.
Account Deletion: Contacting support to request a full wipe of your Firebase Auth profile.
6. Security Contact
For vulnerability reports or privacy inquiries:
help.magmamine@gmail.com
