<script>
    import "../../global.css";
    import { onMount } from 'svelte';
    import { run, compile, resetVM, buffer } from '../../lib/panspark';
    let editorContent = '';
    let outputContent = '';
    let outputStats = '';
    
    function runCode() {
        const code = editorContent;
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
    <div class="editor">
        <textarea class="editor-textarea" bind:value={editorContent}></textarea>
        <button on:click={runCode} class="editor-button">RUN</button>
        <button class="editor-button" on:click={resetCode}>RESET OUTPUT</button>
    </div>
    <div class="output">
        <p class="output-stats">{outputStats}</p>
        <textarea class="output-textarea" readonly bind:value={outputContent}></textarea>
    </div>
</div>