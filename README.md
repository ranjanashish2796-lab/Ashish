# Ashish
My ashish from Complete Web Development Course
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Question AI App</title>
<style>
  body {
    margin:0;
    font-family: system-ui, sans-serif;
    background:#f4f7fb;
    color:#222;
  }
  header {
    padding:16px;
    background:#2c60ff;
    color:#fff;
    font-size:20px;
    font-weight:600;
    text-align:center;
  }
  main { padding:16px; max-width:600px; margin:auto; }
  .card {
    background:#fff;
    padding:16px;
    border-radius:16px;
    box-shadow:0 4px 10px rgba(0,0,0,0.08);
    margin-bottom:16px;
  }
  textarea, select, input[type=file] {
    width:100%; padding:12px; margin-top:8px;
    border:1px solid #ccc; border-radius:12px; font-size:15px;
  }
  button {
    background:#2c60ff; color:#fff; border:none;
    padding:12px 18px; font-size:16px;
    border-radius:12px; cursor:pointer; margin-top:12px;
    width:100%;
  }
  button:disabled { opacity:0.6; }
  #output {
    white-space:pre-wrap; padding:12px; font-size:15px;
  }
  #preview {
    max-width:100%; margin-top:10px; border-radius:12px;
  }
</style>
</head>
<body>
  <header>Question AI App</header>
  <main>
    <div class="card">
      <label>Ask your question</label>
      <textarea id="question" placeholder="Type your question here..."></textarea>
      
      <label style="margin-top:12px;display:block;">Or upload an image</label>
      <input type="file" id="imageInput" accept="image/*">
      <img id="preview" style="display:none;"/>

      <label style="margin-top:12px;display:block;">Subject</label>
      <select id="subject">
        <option>Math</option>
        <option>Physics</option>
        <option>Chemistry</option>
        <option>Biology</option>
        <option>English</option>
        <option>Hindi</option>
        <option>General Knowledge</option>
      </select>

      <button id="askBtn">Get Answer</button>
    </div>

    <div class="card">
      <div id="status">Ready.</div>
      <div id="output"></div>
    </div>
  </main>

<script>
const apiKey = "AIzaSyCalxgpFaSWKxn_o_HFSXigAlSxd2skQAs"; // your key

const askBtn = document.getElementById("askBtn");
const questionEl = document.getElementById("question");
const outputEl = document.getElementById("output");
const statusEl = document.getElementById("status");
const imageInput = document.getElementById("imageInput");
const preview = document.getElementById("preview");

let imageBase64 = null;

// Preview uploaded image
imageInput.addEventListener("change", ()=>{
  const file = imageInput.files[0];
  if(file){
    const reader = new FileReader();
    reader.onload = e=>{
      preview.src = e.target.result;
      preview.style.display="block";
      imageBase64 = e.target.result.split(",")[1];
    };
    reader.readAsDataURL(file);
  } else {
    preview.style.display="none";
    imageBase64=null;
  }
});

askBtn.addEventListener("click", async ()=>{
  const question = questionEl.value.trim();
  if(!question && !imageBase64){
    alert("Please enter a question or upload an image.");
    return;
  }

  askBtn.disabled = true;
  outputEl.textContent = "";
  statusEl.textContent = "Thinking...";

  try{
    const contents = [{
      role:"user",
      parts:[
        {text:`Subject: ${document.getElementById("subject").value}\nQuestion: ${question}`}
      ]
    }];
    if(imageBase64){
      contents[0].parts.push({
        inline_data:{ mime_type:"image/png", data:imageBase64 }
      });
    }

    const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash-latest:generateContent?key=${apiKey}`,{
      method:"POST",
      headers:{ "Content-Type":"application/json" },
      body: JSON.stringify({ contents })
    });

    if(!res.ok){ throw new Error("HTTP "+res.status); }
    const data = await res.json();
    const text = data?.candidates?.[0]?.content?.parts?.map(p=>p.text).join("\n") || "(No answer)";
    outputEl.textContent = text;
    statusEl.textContent = "Done.";
  }catch(err){
    console.error(err);
    statusEl.textContent = "Error: "+err.message;
  }finally{
    askBtn.disabled = false;
  }
});
</script>
</body>
</html>
