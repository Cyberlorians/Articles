# 🖥 Azure Diagram MCP — Full Setup Guide

This guide explains how to set up and use **Azure Diagram MCP** in Visual Studio Code (VS Code).  
It is written for beginners — no prior MCP or VS Code knowledge required.  

---

<details><summary> <b><u><font size="<h3>">Prerequisites.</font></u></b></summary> 
<p>

### Tools Needed
- **Python** (latest version)  
- **Graphviz** (diagram rendering)  
- **Visual Studio Code (VS Code)**  
- **Git** (to download the project)  
- **GitHub Copilot + Copilot Chat extensions**  

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 1: Install Python.</font></u></b></summary> 
<p>

1. Download Python: [https://www.python.org/downloads/](https://www.python.org/downloads/)  
2. Run the installer.  
3. On the first screen, check the box:  
```

Add Python to PATH

````
4. Click **Install Now**.  

**Test:**  
```powershell
python --version
````

Expected output: `Python 3.x.x`

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 2: Install Graphviz.</font></u></b></summary> 
<p>

1. Download Graphviz: [https://graphviz.org/download/](https://graphviz.org/download/)
2. Select the **Windows MSI installer**.
3. Run the installer → Next → Install.

**Test:**

```powershell
dot -V
```

Expected output: `graphviz version 8.x`

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 3: Install Visual Studio Code.</font></u></b></summary> 
<p>

1. Download VS Code: [https://code.visualstudio.com/download](https://code.visualstudio.com/download)
2. Run the installer → accept defaults.

**Test:**

* Open VS Code from the Start Menu.

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 4: Install Git.</font></u></b></summary> 
<p>

1. Download Git for Windows: [https://git-scm.com/download/win](https://git-scm.com/download/win)
2. Run the installer and accept defaults.

**Test:**

```powershell
git --version
```

Expected output: `git version 2.x.x`

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 5: Download the Azure Diagram MCP Project.</font></u></b></summary> 
<p>

1. Open PowerShell.
2. Navigate to Documents:

   ```powershell
   cd C:\Users\<YourName>\Documents
   ```
3. Clone the repository:

   ```powershell
   git clone https://github.com/dminkovski/azure-diagram-mcp.git
   ```
4. Go into the folder:

   ```powershell
   cd azure-diagram-mcp
   ```

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 6: Install Project Requirements.</font></u></b></summary> 
<p>

Run this inside the project folder:

```powershell
pip install -r requirements.txt
```

This installs all Python libraries needed for the project.

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 7: Configure VS Code.</font></u></b></summary> 
<p>

### Open the Project

1. Start VS Code.
2. Go to **File → Open Folder…**
3. Open:

   ```
   C:\Users\<YourName>\Documents\azure-diagram-mcp
   ```

### Install Extensions

* Open the Extensions view (icon with 4 squares).
* Install:

  * **GitHub Copilot**
  * **GitHub Copilot Chat**

### Create MCP Config File

1. In Explorer, create a folder:

   ```
   .vscode
   ```
2. Inside, create a file:

   ```
   mcp.json
   ```
3. Paste the following:

   ```json
   {
     "servers": {
       "Azure Diagram MCP Server": {
         "command": "python",
         "args": [
           "-m",
           "azure_diagram_mcp_server.server"
         ]
       }
     }
   }
   ```
4. Save the file.

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 8: Run the MCP Server.</font></u></b></summary> 
<p>

1. In VS Code, open the terminal:

   ```
   View → Terminal
   ```
2. Ensure path is correct:

   ```
   PS C:\Users\<YourName>\Documents\azure-diagram-mcp>
   ```
3. Start the server:

   ```powershell
   python -m azure_diagram_mcp_server.server
   ```
4. Leave this terminal open.

   * If it “hangs” (does not return to prompt), that’s correct.

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 9: Use Copilot Chat.</font></u></b></summary> 
<p>

1. Open the Command Palette:

   ```
   Ctrl + Shift + P
   ```

2. Search:

   ```
   Copilot: Chat (Panel)
   ```

3. In the chat box, type:

   ```
   /mcp list
   ```

   Expected result:

   ```
   Azure Diagram MCP Server
   ```

4. To create a diagram:

   ```
   /mcp Azure Diagram MCP Server
   Make a diagram that shows:
   - Microsoft Sentinel exporting logs
   - Logs flow into Event Hub
   - Event Hub streams data into Azure Data Explorer (ADX)
   - ADX stores data for long-term retention
   ```

</p>
</details>

---

<details><summary> <b><u><font size="<h3>">Step 10: View Your Diagram.</font></u></b></summary> 
<p>

1. In VS Code Explorer, click Refresh.
2. Open the folder:

   ```
   diagrams
   ```
3. A PNG file will be created, e.g.:

   ```
   diagram_2025-10-02.png
   ```
4. Click the PNG to view it inside VS Code.

</p>
</details>

---

## 🚀 Quick Recap

* **Python + Graphviz** → tools to draw diagrams
* **VS Code + Copilot Chat** → where you request diagrams
* **Terminal** → runs the MCP server
* **Chat panel** → where you send `/mcp` commands
* **Diagrams folder** → where PNG images are saved

```


