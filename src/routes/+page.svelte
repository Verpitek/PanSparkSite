<script>
    import "../global.css";
    import { onMount } from 'svelte';
    
    onMount(() => {
      setTimeout(() => {
        window.location.href = "/editor";
      }, 2000);
      const bootingElement = document.getElementsByClassName("booting")[0];
      const extraElement = document.getElementsByClassName("extra")[0];
      
      let dots = 1;
      const maxDots = 4;
      setTimeout(() => {
              document.querySelector('.container').style.opacity = 1;
          }, 200);
    
      function animateBoot() {
        let dotString = ".".repeat(dots);
        bootingElement.innerHTML = `Loading LunaTechnika NetSpace Kernel${dotString}`;
        
        dots = (dots % maxDots) + 1;
    
        // Call the function again after a delay
        setTimeout(animateBoot, 200); // 500ms delay for a faster animation
      }
      function animateExtra() {
        extraElement.innerHTML = "Active memory: ";
        
        const chineseChars = "愛友情希望幸せ夢평화未来美麗你好世界和平永遠幸福こんにちは愛友情幸せ夢平和希望미래夢想希望美麗愛友情希望幸せ夢평화미래あなたがた愛友情幸せ夢平和希望미래夢想希望美麗愛友情希望幸せ夢平和未来美麗0123456789!#$%&'()*+,-./:;<=>?@[\]^_";
        let matrixText = "";
        for (let i = 0; i < 16; i++) { // Loop to build a longer string
          matrixText += chineseChars[Math.floor(Math.random() * chineseChars.length)];
        }
        
        extraElement.innerHTML += matrixText;
        setTimeout(animateExtra, 100);
      }
      
      animateBoot();
      animateExtra();
    });
</script>

<div class="container boot-screen">
    <h1 class="booting"></h1>
    <h2 class="extra"></h2>
    <p>You will soon be redirected</p>
</div>