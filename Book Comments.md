```dataviewjs
// 1. Configuration
const bookFolder = "Book"; 
const calloutType = "[!comment]";

// 2. Get all markdown files in the folder
const files = app.vault.getMarkdownFiles().filter(f => f.path.startsWith(bookFolder));

const tableRows = [];

for (let file of files) {
    const content = await app.vault.read(file);
    const lines = content.split('\n');
    
    let inComment = false;
    let currentComment = "";
    let blockId = "";

    lines.forEach((line) => {
        if (line.includes(`> ${calloutType}`)) {
            inComment = true;
            return;
        }
        
        if (inComment) {
            if (line.trim().startsWith(">")) {
                let cleanLine = line.replace(/^>\s?/, "").trim();
                
                // Check if this line contains a Block ID (e.g., ^ref1)
                const idMatch = cleanLine.match(/\^([a-zA-Z0-9-]+)$/);
                if (idMatch) {
                    blockId = idMatch[1];
                    cleanLine = cleanLine.replace(idMatch[0], "").trim();
                }
                
                currentComment += cleanLine + " ";
            } else {
                // End of callout - Process Row
                if (currentComment.trim()) {
                    // Create the link: Use Block ID if found, otherwise fallback to search
                    const linkPath = blockId 
                        ? `${file.path}#^${blockId}` 
                        : `${file.path}`;
                    
                    const displayLink = blockId ? "🎯 Jump" : "📂 Open";

                    tableRows.push([
                        file.basename, 
                        `[[${linkPath}|${displayLink}]]`, 
                        currentComment.trim()
                    ]);
                }
                currentComment = "";
                blockId = "";
                inComment = false;
            }
        }
    });
}

dv.table(["Chapter", "Precision Link", "Editorial Note"], tableRows);
```







