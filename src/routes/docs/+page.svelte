<script>
  import { marked } from "marked";
    document.getElementById('content').innerHTML =
      marked.parse('# Marked in the browser\n\nRendered by **marked**.');
</script>


<div class="container boot-screen">
    <h1>Documentation</h1>
    <div id="content"></div>
</div>