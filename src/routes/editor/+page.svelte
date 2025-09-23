<script>
    import "../../global.css";
    import favicon from '$lib/assets/favicon.png';
    import { onMount } from 'svelte';
    import { PanSparkVM } from '../../lib/panspark';
    let editorContent = '';
    let outputContent = '';
    let outputStats = 'Compile Time: None Runtime: None';
    
    // Create PanSpark VM instance
    let vm = new PanSparkVM();
    import * as listModule from '../../lib/std/list';
    vm.loadModule('list', listModule);
    
    import CodeMirror from "svelte-codemirror-editor";
      
    let value = `
// ===== MODULE IMPORTS =====
IMPORT "list"
ECHO "Modules loaded successfully"

// ===== VARIABLE OPERATIONS =====
SET 42 >> answer
SET 100 >> initialValue
SET initialValue >> copiedValue

// ===== MATH OPERATIONS =====
MATH initialValue + 8 >> addResult
MATH initialValue - 15 >> subResult
MATH initialValue * 2 >> mulResult
MATH initialValue / 4 >> divResult
MATH initialValue % 3 >> modResult
MATH initialValue min 59 >> minResult
MATH initialValue max 59 >> maxResult
MATH 2 ** 8 >> expResult
MATH 25 sqrt >> sqrtResult
MATH 100 log >> logResult
MATH 10 rand >> randResult
MATH 7.8 floor >> floorResult
MATH 7.2 ceil >> ceilResult
MATH 3.14159 sin >> sinResult
MATH 3.14159 cos >> cosResult
MATH 3.14159 tan >> tanResult
MATH -3443 abs >> absResult
MATH 23.43 round >> roundResult
MATH 3.9 log10 >> log10Result
MATH 32.21 exp >> expXResult

// ===== PRINT OPERATIONS =====
ECHO "=== Math Operation Results ==="
ECHO "Basic Operations:"
PRINT addResult
PRINT subResult
PRINT mulResult
PRINT divResult
PRINT modResult

ECHO "Advanced Math:"
PRINT expResult
PRINT sqrtResult
PRINT logResult
PRINT randResult
PRINT floorResult
PRINT ceilResult
PRINT sinResult
PRINT cosResult
PRINT tanResult
PRINT absResult
PRINT roundResult
PRINT log10Result
PRINT expXResult

// ===== MEMORY OPERATIONS =====
ECHO "=== Memory Operations ==="
SET 999 >> tempVar
PRINT tempVar
FREE tempVar
ECHO "Variable freed from memory"

// ===== INCREMENT/DECREMENT OPERATIONS =====
ECHO "=== Increment/Decrement Operations ==="
SET 10 >> counter
PRINT counter
INC counter
PRINT counter
DEC counter
DEC counter
PRINT counter

// ===== LIST OPERATIONS =====
ECHO "=== List Operations ==="
SET 1 >> first
SET 2 >> second
SET 3 >> third

// Create list with variables and literals
LIST_CREATE [first, second, third, 4, 5] >> numbers
LIST_DUMP numbers

// Modify list elements
LIST_SET 99 2 >> numbers
LIST_DUMP numbers

// Get element from list
LIST_GET numbers 2 >> extracted
ECHO "Extracted element:"
PRINT extracted

// Add elements to list
LIST_PUSH 6 >> numbers
LIST_PUSH 7 >> numbers
LIST_DUMP numbers

// Get list length
LIST_LEN numbers >> listSize
ECHO "List length:"
PRINT listSize

// Sort list
ECHO "Sorting list ascending:"
LIST_SORT numbers
LIST_DUMP numbers

ECHO "Sorting list descending:"
LIST_SORT numbers desc
LIST_DUMP numbers

// ===== MEMORY DEBUGGING =====
ECHO "=== Memory Dump ==="
MEMDUMP

// ===== TICK COUNTER =====
ECHO "=== Execution Counter ==="
TICK currentTick
PRINT currentTick

// ===== CONTROL FLOW =====
ECHO "=== Control Flow - Loops ==="
SET 0 >> loopCounter
POINT loop_start
MATH loopCounter + 1 >> loopCounter
ECHO "Loop iteration:"
PRINT loopCounter
IF loopCounter < 5 >> loop_start

// ===== CONDITIONAL JUMPS =====
ECHO "=== Conditional Jumps ==="
SET 10 >> x
SET 15 >> y

IF x > y >> greater
IF x < y >> less
IF x == y >> equal
JUMP comparisons_done

POINT greater
ECHO "X is greater than Y"
PRINT 9001
JUMP comparisons_done

POINT less
ECHO "X is less than Y"
PRINT 9000
JUMP comparisons_done

POINT equal
ECHO "X equals Y"
PRINT 9002

POINT comparisons_done
ECHO "Comparisons completed"

// ===== ADVANCED COMPARISONS =====
ECHO "=== Advanced Comparisons ==="
SET 20 >> a
SET 20 >> b

IF a >= b >> greater_equal
ECHO "A is not >= B"
JUMP not_greater_equal

POINT greater_equal
ECHO "A is >= B"

POINT not_greater_equal

IF a <= b >> less_equal
ECHO "A is not <= B"
JUMP not_less_equal

POINT less_equal
ECHO "A is <= B"

POINT not_less_equal

IF a != b >> not_equal
ECHO "A equals B"
JUMP are_equal

POINT not_equal
ECHO "A does not equal B"

POINT are_equal

// ===== PROCEDURES (PROCs) =====
ECHO "=== Procedures ==="

// Define a PROC to calculate factorial
PROC factorial (n) {
    SET 1 >> result
    POINT factorial_loop
    IF n <= 1 >> end_factorial_loop
    MATH result * n >> result
    MATH n - 1 >> n
    JUMP factorial_loop
    POINT end_factorial_loop
    RETURN result
}

// Define a PROC to calculate Fibonacci (iterative to avoid stack overflow)
PROC fibonacci (n) {
    IF n <= 1 >> fib_base_case
    
    SET 0 >> a
    SET 1 >> b
    SET 2 >> i
    
    POINT fib_loop
    IF i > n >> fib_done
    MATH a + b >> temp
    SET b >> a
    SET temp >> b
    MATH i + 1 >> i
    JUMP fib_loop
    
    POINT fib_done
    RETURN b
    
    POINT fib_base_case
    RETURN n
}

// Define a PROC to find maximum of two numbers
PROC max_two (a, b) {
    IF a > b >> a_greater
    RETURN b
    POINT a_greater
    RETURN a
}

// Call procedures and demonstrate results
ECHO "Calculating factorial of 5:"
CALL factorial (5) >> factorial_result
PRINT factorial_result

ECHO "Calculating Fibonacci of 5:"
CALL fibonacci (5) >> fib_result
PRINT fib_result

ECHO "Finding max of 25 and 17:"
CALL max_two (25, 17) >> max_result
PRINT max_result


// ===== NO OPERATION =====
ECHO "=== No Operation Test ==="
NOP
ECHO "NOP executed successfully"

// ===== FINAL MEMORY STATE =====
ECHO "=== Final Memory State ==="
MEMDUMP

ECHO "=== Program Execution Complete! ==="

// ===== END PROGRAM EXECUTION =====
END`;
    
      import { EditorView } from "@codemirror/view";
      import { HighlightStyle, syntaxHighlighting, StreamLanguage } from "@codemirror/language";
      import { tags } from "@lezer/highlight";
      
      const keywords = new Set([
          // Opcodes
          "SET", "MATH", "PRINT", "IF", "JUMP", "POINT",
          "END", "RETURN", "PROC", "CALL", "ECHO", "WAIT",
          "INC", "DEC", "FREE", "NOP", "MEMDUMP", "TICK", "IMPORT", "LIST_CREATE", "LIST_GET", "LIST_SET", "LIST_PUSH", "LIST_DUMP", "LIST_LEN", "LIST_SORT",
          // Math functions
          "sqrt", "log", "rand", "floor", "ceil", "sin", "cos", "tan", "abs", "round", "log10", "exp"
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
          
          if (stream.peek() == '"') {
            stream.next();
            while (stream.peek() != null && stream.peek() != '"') {
              stream.next();
            }
            stream.next();
            return "string";
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
          string: tags.string,
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
        const compiledCode = vm.compile(code);
        const compileTime = performance.now() - compileStart;
        const runStart = performance.now();
        let program = vm.run(compiledCode);
        while (program.next().done === false) {
        }
        const runTime = performance.now() - runStart;
        outputStats = `Compile Time: ${compileTime}ms\nRuntime: ${runTime}ms`;
        const vmBuffer = vm.getBuffer();
        for (let i = 0; i < vmBuffer.length; i++) {
          outputContent += vmBuffer[i] + '\n';
        }
        vm.resetVM();
    }
    
    function resetCode() {
        outputContent = '';
    }
    
    onMount(() => {
                const bootScreen = document.getElementsByClassName('boot-screen')[0];
                const lines = [
                  'PanSpark v1.2 loaded',
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
        <textarea class="output-textarea">{outputContent}</textarea>
    </div>
    <div><a href="https://github.com/Verpitek/PanSpark">PanSpark GitHub</a></div>
    <div><a href="https://discord.gg/fyNQNrr7dC">Verpitek Discord</a></div>
</div>