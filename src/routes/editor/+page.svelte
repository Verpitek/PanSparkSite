<script>
    import "../../global.css";
    import favicon from '$lib/assets/favicon.png';
    import { onMount } from 'svelte';
    import { PanSparkVM } from '../../lib/panspark';
    let editorContent = '';
    // Make outputContent reactive
    let outputContent = '';
    let outputStats = 'Compile Time: None Runtime: None';
    
    // Create PanSpark VM instance
    let vm = new PanSparkVM();
    import * as listModule from '../../lib/std/list';
    vm.loadModule('list', listModule);
    
    import CodeMirror from "svelte-codemirror-editor";
    
    let value = `// Welcome to PanSpark!
ECHO "=== PanSpark Demo Start ==="

// Basic math
SET 5 >> a
SET 10 >> b
SET 34 >> c
MATH a + b + c >> sum
ECHO "Sum of a + b + c:"
PRINT sum

// Procedure example
PROC square (x)
  MATH x * x >> result
  RETURN result
ENDPROC

CALL square (sum) >> squared
ECHO "Squared result:"
PRINT squared

// FOR loop example
ECHO "Counting from 0 to 5:"
FOR i 0 5
  PRINT i
ENDFOR

// List example
LIST_CREATE nums
LIST_PUSH 3 >> nums
LIST_PUSH 1 >> nums
LIST_PUSH 2 >> nums
LIST_SORT nums min
ECHO "Sorted list:"
PRINT nums

ECHO "=== Demo Complete ==="
END
`;
    
    import { EditorView } from "@codemirror/view";
    import { HighlightStyle, syntaxHighlighting, StreamLanguage } from "@codemirror/language";
    import { tags } from "@lezer/highlight";
    
    // ... (Your CodeMirror language and styling definitions remain unchanged)
    // --- START OF UNCHANGED CODE MIRROR SETUP ---
    const keywords = new Set([
        // Core Opcodes
        "SET", "MATH", "PRINT", "IF", "JUMP", "POINT",
        "END", "RETURN", "PROC", "CALL", "ECHO", "WAIT",
        "INC", "DEC", "FREE", "NOP", "MEMDUMP", "TICK", "IMPORT", "ENDPROC", "ENDFOR", "FOR", "BREAK", "CONTINUE",
        // List Operations
        "LIST_CREATE", "LIST_GET", "LIST_SET", "LIST_PUSH",
        "LIST_DUMP", "LIST_LEN", "LIST_SORT",
        // Math functions
        "sqrt", "log", "rand", "floor", "ceil", "sin", "cos",
        "tan", "abs", "round", "log10", "exp", "min", "max",
        // List sort params
        "desc", "asc"
    ]);
    
    // Define operators
    const operators = /^(>>|==|!=|>=|<=|\*\*|[+\-*/%><])/;
    
    export const pansparkLanguage = StreamLanguage.define({
        token: (stream) => {
            // Skip whitespace
            if (stream.eatSpace()) return null;
            
            // Line comments
            if (stream.match("//")) {
                stream.skipToEnd();
                return "lineComment";
            }
            
            // Numbers (including decimals and negatives)
            if (stream.match(/^-?[0-9]+(?:\.[0-9]+)?\b/)) {
                return "number";
            }
            
            // Operators
            if (stream.match(operators)) {
                return "operator";
            }
            
            // Strings
            if (stream.peek() == '"') {
                stream.next();
                while (stream.peek() != null && stream.peek() != '"') {
                    if (stream.peek() == '\\') {
                        stream.next(); // Skip escaped character
                    }
                    stream.next();
                }
                stream.next();
                return "string";
            }
            
            // List literals - brackets
            if (stream.match(/[\[\]]/)) {
                return "bracket";
            }
            
            // Commas in lists and function calls
            if (stream.match(/,/)) {
                return "punctuation";
            }
            
            // Parentheses
            if (stream.match(/[(){}]/)) {
                return "paren";
            }
            
            // Keywords, Variables, and Labels
            if (stream.match(/^[a-zA-Z_][a-zA-Z0-9_]*\b/)) {
                const word = stream.current();
                if (keywords.has(word.toUpperCase()) || keywords.has(word)) {
                    return "keyword";
                }
                
                // Check if it's a label (after POINT or before >>)
                return "variableName";
            }
            
            // If nothing else matches, just advance the stream
            stream.next();
            return null;
        },
        
        // This maps the token names above to the standard highlight style tags
        tokenTable: {
            keyword: tags.keyword,
            number: tags.number,
            variableName: tags.variableName,
            operator: tags.operator,
            lineComment: tags.lineComment,
            string: tags.string,
            bracket: tags.squareBracket,
            punctuation: tags.separator,
            paren: tags.paren,
        },
    });
    
    // Updated highlight style
    const amberHighlightStyle = HighlightStyle.define([
        { tag: tags.keyword, color: "#ffb000", fontWeight: "bold" },
        { tag: tags.comment, color: "#826d3e", fontStyle: "italic" },
        { tag: tags.lineComment, color: "#826d3e", fontStyle: "italic" },
        { tag: tags.string, color: "#a5ff95" },
        { tag: tags.number, color: "#a5ff95" },
        { tag: tags.variableName, color: "#c8b0ff" },
        { tag: tags.operator, color: "#ffd700" },
        { tag: tags.propertyName, color: "#ffd700" },
        { tag: tags.typeName, color: "#ffb000" },
        { tag: tags.className, color: "#ffb000" },
        { tag: tags.punctuation, color: "#ffd700" },
        { tag: tags.separator, color: "#ffd700" },
        { tag: tags.paren, color: "#ff8c42" },
        { tag: tags.squareBracket, color: "#ff8c42" },
        { tag: tags.definition(tags.variableName), color: "#c8b0ff", fontWeight: "bold" }
    ]);
    
    // Enhanced theme with better visibility
    const amberTheme = EditorView.theme({
        "&": {
            backgroundColor: "transparent",
            color: "#ffb000",
            fontSize: "16px",
        },
        ".cm-content": {
            caretColor: "#ffb000",
            padding: "10px 0",
        },
        ".cm-cursor, .cm-dropCursor": {
            borderLeftColor: "#ffb000",
            borderLeftWidth: "2px",
        },
        ".cm-selectionBackground, ::selection": {
            backgroundColor: "#ffb00033 !important",
        },
        ".cm-focused .cm-selectionBackground, :focus ::selection": {
            backgroundColor: "#ffb00044 !important",
        },
        ".cm-activeLine": {
            backgroundColor: "rgba(255, 176, 0, 0.05)",
        },
        ".cm-gutters": {
            backgroundColor: "transparent",
            color: "#826d3e",
            border: "none",
            borderRight: "1px solid #ffb00033",
        },
        ".cm-activeLineGutter": {
            backgroundColor: "rgba(255, 176, 0, 0.08)",
            color: "#ffb000",
        },
        ".cm-lineNumbers .cm-gutterElement": {
            padding: "0 10px 0 5px",
            minWidth: "40px",
        },
        ".cm-line": {
            padding: "0 8px",
        },
        // Matching brackets
        ".cm-matchingBracket, .cm-nonmatchingBracket": {
            backgroundColor: "rgba(255, 176, 0, 0.2)",
            outline: "1px solid #ffb000",
        },
    }, { dark: true });
    
    // Updated extensions array
    const customExtensions = [
        amberTheme,
        syntaxHighlighting(amberHighlightStyle),
        pansparkLanguage
    ];
    // --- END OF UNCHANGED CODE MIRROR SETUP ---
    
    function runCode() {
        // 1. Clear previous output for a fresh run
        outputContent = '';
        outputStats = 'Compile Time: None Runtime: None';
        vm.resetVM();

        const code = value;
        let compiledCode;
        let compileTime = 0;
        let runTime = 0;

        try {
            // --- Compilation Phase ---
            const compileStart = performance.now();
            // This is where compilation errors are most likely to occur
            compiledCode = vm.compile(code);
            compileTime = performance.now() - compileStart;

            // --- Runtime Phase ---
            const runStart = performance.now();
            let program = vm.run(compiledCode);

            // Iterate through the generator, error can be thrown here too
            while (true) {
                const result = program.next();
                if (result.done) break;
            }
            
            runTime = performance.now() - runStart;

            // --- Success Output ---
            outputStats = `Compile Time: ${compileTime.toFixed(2)}ms\nRuntime: ${runTime.toFixed(2)}ms`;
            
            const vmBuffer = vm.getBuffer();
            // Note: I changed the loop to use a reactive assignment to `outputContent`
            // which works better in Svelte than `+=` for immediate updates.
            // Using a single string join is also more performant.
            outputContent = vmBuffer.join('\n');
            
        } catch (error) {
            // --- Error Handling Phase ---
            outputContent = `--- ERROR ---\n${error.name || 'Error'}: ${error.message}`;
            outputStats = 'ERROR: See output log.';

            vm.resetVM();
            return; // Exit the function immediately after handling the error
        }

        // 5. Reset VM state after a successful run as originally intended (and after a failed run above)
        vm.resetVM();
    }
    
    function resetCode() {
        outputContent = '';
        outputStats = 'Compile Time: None Runtime: None'; // Also reset stats
    }
    
    onMount(() => {
        // ... (Your onMount boot screen logic remains unchanged)
        const bootScreen = document.getElementsByClassName('boot-screen')[0];
        const lines = [
            'PanSpark v1.0.1 loaded',
            'Developed by Verpitek',
            'A tiny scripting language for game scripting',
            'You can go to the bottom of the site to run the code',
            '',
            '>'
        ];
        let lineIndex = 0;
        let charIndex = 0;
        const delay = 1; // Milliseconds between characters

        function typeWriter() {
            if (lineIndex < lines.length) {
                if (charIndex < lines[lineIndex].length) {
                    bootScreen.innerHTML += lines[lineIndex].charAt(charIndex);
                    charIndex++;
                    setTimeout(typeWriter, delay);
                } else {
                    bootScreen.innerHTML += '<br>';
                    lineIndex++;
                    charIndex = 0;
                    setTimeout(typeWriter, delay * Math.random()* 50);
                }
            }
        }

        setTimeout(() => {
            document.querySelector('.container').style.opacity = 1;
        }, 200);

        typeWriter();
    
    });
</script>

<style>
    /* Styling for the output textarea to handle and display errors clearly */
    .output-textarea {
        white-space: pre-wrap; /* Ensures the multiline error message is respected */
        word-wrap: break-word;
        font-family: monospace;
        color: inherit; /* Inherit color, or set a specific color for errors */
        /* You might want to set a distinct color for errors, e.g., red or orange */
    }
</style>

<div><img src="{favicon}" style="max-height: 200px;" alt="panspark logo"></div>
<div class="boot-screen"></div>
<div class="container">
    <div>
    <CodeMirror 
            bind:value 
            extensions={customExtensions}
            lineNumbers={true}
            lineWrapping={false}
            styles={{
                "&": {
                    background: "transparent",
                    color: "#ffb000"
                },
                ".cm-gutters": {
                    background: "transparent",
                    borderRight: "1px solid #ffb00033"
                    
                },
                ".cm-activeLine": {
                    background: "transparent"
                },
                ".cm-activeLineGutter": {
                }
            }}
        />
    </div>
    <button on:click={runCode} class="editor-button">RUN</button>
    <button class="editor-button" on:click={resetCode}>RESET OUTPUT</button>
    <p>Output</p>
    <p class="output-stats">{outputStats}</p>
    <div class="output" style="border: solid 1px #ffd700; border-radius: 5px;">
        <textarea class="output-textarea" readonly bind:value={outputContent}></textarea>
    </div>
    <div><a href="https://github.com/Verpitek/PanSpark">PanSpark GitHub</a></div>
    <div><a href="https://github.com/Verpitek/PanSpark/wiki">PanSpark Documentation</a></div>
    <div><a href="https://discord.gg/fyNQNrr7dC">Verpitek Discord</a></div>
</div>
