<script>
    import "../../global.css";
    import { onMount } from 'svelte';
    import { run, compile, resetVM, buffer } from '../../lib/panspark';
    let editorContent = '';
    let outputContent = '';
    let outputStats = '';
    
    import CodeMirror from "svelte-codemirror-editor";
      
    let value = `SET 10 >> num1 // the SET operator allows setting a value to a variable
SET 0 >> counter

POINT loop // this defines a jump point to a specific location
  MATH counter + 1 >> counter
  IF counter < num1 >> loop // ifs allow a logical operation that if succeeds, jumps to a designated POINT
  PRINT counter // PRINT allows the printing of variables or values`
    
      import { EditorView } from "@codemirror/view";
      import { HighlightStyle, syntaxHighlighting, StreamLanguage } from "@codemirror/language";
      import { tags } from "@lezer/highlight";
      
      const keywords = new Set([
          // Opcodes
          "SET", "MATH", "PRINT", "IF", "JUMP", "POINT",
          "END", "MEMVIPE", "MEMDUMP", "RETURN",
          // Math functions
          "sqrt", "log", "rand", "floor", "ceil", "sin"
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
      
          // Numbers
          if (stream.match(/^[0-9]+(?:\.[0-9]+)?\b/)) {
            return "number";
          }
          
          // Operators
          if (stream.match(operators)) {
            return "operator";
          }
      
          // Keywords, Variables, and Labels
          if (stream.match(/^[a-zA-Z_][a-zA-Z0-9_]*\b/)) {
            const word = stream.current();
            if (keywords.has(word.toUpperCase()) || keywords.has(word)) {
              return "keyword";
            }
            
            // For this simple parser, we'll treat labels and variables the same.
            // Your highlight style colors them similarly, so it will look good.
            // A more advanced parser could differentiate them based on context (e.g., after a POINT or JUMP).
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
          variableName: tags.variableName, // Also used for labels
          operator: tags.operator,
          lineComment: tags.lineComment,
        },
      });

      const amberTheme = EditorView.theme({
              // ... your theme styles
          }, { dark: true });
      
          
      const amberHighlightStyle = HighlightStyle.define([
          { tag: tags.keyword, color: "#ffb000", fontWeight: "bold" },
          { tag: tags.comment, color: "#826d3e", fontStyle: "italic" },
          { tag: tags.string, color: "#a5ff95" },
          { tag: tags.number, color: "#a5ff95" },
          { tag: tags.variableName, color: "#c8b0ff" },
          { tag: tags.operator, color: "#ffd700" },
          { tag: tags.propertyName, color: "#ffd700" },
          { tag: tags.typeName, color: "#ffb000" },
          { tag: tags.className, color: "#ffb000" },
          { tag: tags.punctuation, color: "#ffd700" },
          { tag: tags.paren, color: "#ffd700" },
          { tag: tags.definition(tags.variableName), color: "#c8b0ff", fontWeight: "bold" }
      ]);
      
          // 3. UPDATE your extensions array to use panspark()
          const customExtensions = [
              amberTheme,
              syntaxHighlighting(amberHighlightStyle),
              pansparkLanguage // Use your custom language here!
          ];
    
    function runCode() {
        const code = value;
        const compileStart = performance.now();
        const compiledCode = compile(code);
        const compileTime = performance.now() - compileStart;
        const runStart = performance.now();
        run(compiledCode);
        const runTime = performance.now() - runStart;
        outputStats = `Compile Time: ${compileTime}ms\nRuntime: ${runTime}ms`;
        for (let i = 0; i < buffer.length; i++) {
          outputContent += buffer[i] + '\n';
        }
        resetVM();
    }
    
    function resetCode() {
        outputContent = '';
    }
    
    onMount(() => {
                const bootScreen = document.getElementsByClassName('boot-screen')[0];
                const lines = [
                    'NetSpace Operating System - v1.34',
                    'Developed by Verpitek',
                    '',
                    '',
                    'Loading PanSpark...',
                    'PanSpark v1.0 loaded',
                    '',
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
    
                typeWriter();
              
    });
</script>

<div class="container">
    <div class="boot-screen"></div>
    <button on:click={runCode} class="editor-button">RUN</button>
    <button class="editor-button" on:click={resetCode}>RESET OUTPUT</button>
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
    <div class="output">
        <p class="output-stats">{outputStats}</p>
        <textarea class="output-textarea" readonly bind:value={outputContent}></textarea>
    </div>
</div>