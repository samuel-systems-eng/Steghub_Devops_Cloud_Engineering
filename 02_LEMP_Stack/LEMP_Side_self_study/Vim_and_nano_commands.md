# Command-Line Text Editors (Vim and Nano)

DevOps engineers frequently configure files directly on remote servers using terminal-based text editors. Mastery of both Vim and Nano ensures flexibility across diverse Linux environments.  

## 1. Vim Editor
Vim is a powerful, modal text editor available on almost all Unix-like systems. It operates in different modes, primarily `Normal Mode` (for navigation/commands) and `Insert Mode` (for typing text).  

### Essential Vim Workflow
    • Open a file: vim filename.txt (Starts in Normal Mode).  

    • Switch to Insert Mode: Press i.  

    • Return to Normal Mode: Press Esc.  

### Commonly Used Vim Commands (From Normal Mode)  

    • :w: Saves (writes) the file. 

    • :q: Quits the editor.  

    • :wq or x: Saves and quits simultaneously.  

    • :q!: Quits without saving changes.  

    • u: Undo the last action.  

    • Ctrl + r: Redo the last undone action.  

    • dd: Deletes (cuts) the current line.  

    • yy: Copies (yanks) the current line.  

    • p: Pastes the copied/cut text below the cursor.  

    • /pattern: Searches for a specific word or pattern in the text.  


## 2. Nano Editor
Nano is a straightforward, modeless text editor. It is highly user-friendly for beginners because it displays available shortcuts at the bottom of the screen.  

### Essential Nano Workflow
    • Open a file: nano filename.txt  

    • Edit text: Type directly into the interface immediately.  

### Commonly Used Nano Commands
    • Ctrl + O: WriteOut (Saves the file).  

    • Ctrl + X: Exit the editor (prompts to save if changes exist).  

    • Ctrl + K: Cut the current line of text.  

    • Ctrl + U: Uncut (Paste) the text that was just cut.  

    • Ctrl + W: Where Is (Search for a specific keyword).  

    • Ctrl + \: Search and replace text.  

    • Ctrl + C: Display current cursor position (line and column status).  

    • Ctrl + _: Go to a specific line number.  

### Summary Comparison for DevOps
    • Nano is optimal for quick, simple configuration edits due to its immediate visual guides and lack of modes.  
    
    • Vim is superior for complex text manipulation, large file editing, and environments where resources are minimal, making it a foundational tool for infrastructure automation workflows.
