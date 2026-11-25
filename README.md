

<!DOCTYPE html>
<html lang="ro">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="theme-color" content="#153b35">
<title>Vând Rapid</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:#f2f5f3;color:#202a27}
header{background:#153b35;color:white;padding:20px;text-align:center}
header h1{margin:0;font-size:27px}
header p{margin:6px 0 0;font-size:13px}
main{max-width:700px;margin:auto;padding:15px}
.card{background:white;padding:16px;border-radius:14px;margin-bottom:16px;box-shadow:0 2px 8px #00000012}
h2{font-size:19px;margin:0 0 14px}
label{display:block;font-weight:bold;margin:12px 0 6px}
input,textarea{width:100%;padding:12px;border:1px solid #ccd5d0;border-radius:9px;font-size:16px}
textarea{min-height:75px;resize:vertical}
button{border:0;border-radius:9px;padding:12px 15px;font-size:15px;font-weight:bold;cursor:pointer}
.primary{background:#176b50;color:white}
.secondary{background:#e5eee9;color:#153b35}
.danger{background:#ffe5e5;color:#a32626}
.actions{display:flex;gap:8px;flex-wrap:wrap;margin-top:12px}
.actions button{flex:1}
.photo-area{display:flex;gap:10px;flex-wrap:wrap;margin-top:10px}
.photo-area button{flex:1;min-width:130px}
.preview{width:100%;height:190px;object-fit:contain;background:#f0f2f1;border-radius:10px;margin-top:12px}
.item{display:flex;gap:12px;padding:12px 0;border-bottom:1px solid #e5e9e6}
.item:last-child{border-bottom:0}
.item img{width:105px;height:105px;object-fit:cover;border-radius:9px;background:#eee}
.item-info{flex:1;min-width:0}
.item-info h3{margin:0 0 7px;font-size:17px;overflow-wrap:anywhere}
.price{font-weight:bold;color:#176b50;margin:5px 0}
.small{font-size:13px;color:#65716b}
.empty{text-align:center;color:#78827d;padding:20px}
.hidden{display:none}
footer{text-align:center;color:#7b8580;font-size:12px;padding:20px}
</style>
</head>
<body>

<header>
<h1>VÂND RAPID</h1>
<p>Catalogul tău personal de vânzări</p>
</header>

<main>
<section class="card">
<h2 id="formTitle">Adaugă un produs</h2>

<label for="name">Numele produsului</label>
<input id="name" type="text" placeholder="Ex: Telefon Samsung">

<label for="price">Preț (lei)</label>
<input id="price" type="number" min="0" placeholder="Ex: 500">

<label for="description">Descriere</label>
<textarea id="description" placeholder="Detalii despre produs..."></textarea>

<label>Fotografia produsului</label>
<div class="photo-area">
<button type="button" class="primary" id="cameraBtn">📷 Fă fotografie</button>
<button type="button" class="secondary" id="galleryBtn">🖼️ Alege din galerie</button>
</div>

<input class="hidden" id="cameraInput" type="file" accept="image/*" capture="environment">
<input class="hidden" id="galleryInput" type="file" accept="image/*">

<img id="preview" class="preview" alt="Previzualizare fotografie">

<div class="actions">
<button class="primary" id="saveBtn">Salvează produsul</button>
<button class="secondary" id="cancelBtn">Anulează</button>
</div>
</section>

<section class="card">
<h2>Produsele mele</h2>
<div id="products">
<div class="empty">Nu ai adăugat încă produse.</div>
</div>
</section>
</main>

<footer>VÂND RAPID · Catalog personal</footer>

<script>
const $ = id => document.getElementById(id);
const STORAGE_KEY = "vandrapid-products";
const PLACEHOLDER = "data:image/svg+xml;charset=UTF-8," +
encodeURIComponent('<svg xmlns="http://www.w3.org/2000/svg" width="400" height="300"><rect width="100%" height="100%" fill="#e9eeeb"/><text x="50%" y="50%" text-anchor="middle" dominant-baseline="middle" fill="#78827d" font-size="24">Fără fotografie</text></svg>');

let products = [];
let photoData = "";
let editingId = null;

function loadProducts(){
  try{
    products = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
    if(!Array.isArray(products)) products = [];
  }catch(e){
    products = [];
  }
}

function persist(){
  try{
    localStorage.setItem(STORAGE_KEY, JSON.stringify(products));
    return true;
  }catch(e){
    alert("Nu s-a putut salva. Memoria telefonului poate fi plină.");
    return false;
  }
}

function resetForm(){
  $("name").value = "";
  $("price").value = "";
  $("description").value = "";
  photoData = "";
  editingId = null;
  $("preview").src = PLACEHOLDER;
  $("cameraInput").value = "";
  $("galleryInput").value = "";
  $("formTitle").textContent = "Adaugă un produs";
  $("saveBtn").textContent = "Salvează produsul";
}

function readImage(file){
  return new Promise((resolve,reject)=>{
    const reader = new FileReader();
    reader.onload = ()=>{
      const img = new Image();
      img.onload = ()=>resolve(img);
      img.onerror = ()=>reject(new Error("Telefonul nu poate deschide această fotografie. Încearcă o fotografie JPG."));
      img.src = reader.result;
    };
    reader.onerror = ()=>reject(new Error("Fotografia nu a putut fi citită."));
    reader.readAsDataURL(file);
  });
}

async function resizePhoto(file){
  if(!file || !file.type.startsWith("image/")){
    throw new Error("Te rog alege o fotografie.");
  }

  let canvas = document.createElement("canvas");
  let width, height, source;

  if("createImageBitmap" in window){
    try{
      source = await createImageBitmap(file);
      width = source.width;
      height = source.height;
    }catch(e){
      source = null;
    }
  }

  if(!source){
    source = await readImage(file);
    width = source.naturalWidth;
    height = source.naturalHeight;
  }

  if(!width || !height) throw new Error("Fotografia nu este validă.");

  const max = 1200;
  const scale = Math.min(1, max / Math.max(width,height));
  canvas.width = Math.round(width * scale);
  canvas.height = Math.round(height * scale);

  const ctx = canvas.getContext("2d");
  if(!ctx) throw new Error("Nu pot procesa fotografia pe acest telefon.");

  ctx.drawImage(source,0,0,canvas.width,canvas.height);

  if(source.close) source.close();

  const result = canvas.toDataURL("image/jpeg",0.78);
  if(!result || result === "data:,"){
    throw new Error("Fotografia nu a putut fi prelucrată.");
  }
  return result;
}

async function handlePhoto(input){
  const file = input.files && input.files[0];
  if(!file) return;

  try{
    $("preview").style.opacity = "0.5";
    const result = await resizePhoto(file);
    photoData = result;
    $("preview").src = result;
  }catch(e){
    alert(e.message || "Fotografia nu a putut fi încărcată.");
  }finally{
    $("preview").style.opacity = "1";
    input.value = "";
  }
}

$("cameraBtn").addEventListener("click",()=>{
  $("cameraInput").click();
});

$("galleryBtn").addEventListener("click",()=>{
  $("galleryInput").click();
});

$("cameraInput").addEventListener("change",function(){
  handlePhoto(this);
});

$("galleryInput").addEventListener("change",function(){
  handlePhoto(this);
});

function render(){
  const container = $("products");
  container.innerHTML = "";

  if(products.length === 0){
    container.innerHTML = '<div class="empty">Nu ai adăugat încă produse.</div>';
    return;
  }

  products.forEach(product=>{
    const item = document.createElement("div");
    item.className = "item";

    const img = document.createElement("img");
    img.src = product.photo || PLACEHOLDER;
    img.alt = product.name || "Produs";
    img.onerror = function(){
      this.onerror = null;
      this.src = PLACEHOLDER;
    };

    const info = document.createElement("div");
    info.className = "item-info";

    const title = document.createElement("h3");
    title.textContent = product.name;

    const price = document.createElement("div");
    price.className = "price";
    price.textContent = product.price ? product.price + " lei" : "Preț negociabil";

    const description = document.createElement("div");
    description.className = "small";
    description.textContent = product.description || "";

    const buttons = document.createElement("div");
    buttons.className = "actions";

    const edit = document.createElement("button");
    edit.className = "secondary";
    edit.textContent = "Editează";
    edit.onclick = ()=>editProduct(product.id);

    const share = document.createElement("button");
    share.className = "primary";
    share.textContent = "Distribuie";
    share.onclick = ()=>shareProduct(product);

    const remove = document.createElement("button");
    remove.className = "danger";
    remove.textContent = "Șterge";
    remove.onclick = ()=>deleteProduct(product.id);

    buttons.append(edit,share,remove);
    info.append(title,price,description,buttons);
    item.append(img,info);
    container.appendChild(item);
  });
}

function saveProduct(){
  const name = $("name").value.trim();
  const price = $("price").value.trim();
  const description = $("description").value.trim();

  if(!name){
    alert("Scrie numele produsului.");
    $("name").focus();
    return;
  }

  let product;

  if(editingId !== null){
    const index = products.findIndex(p=>p.id === editingId);
    if(index === -1) return;
    product = {...products[index],name,price,description,photo:photoData};
    products[index] = product;
  }else{
    product = {
      id: Date.now(),
      name,
      price,
      description,
      photo: photoData
    };
    products.unshift(product);
  }

  if(persist()){
    render();
    resetForm();
    alert("Produs salvat cu succes!");
  }
}

function editProduct(id){
  const p = products.find(item=>item.id === id);
  if(!p) return;

  editingId = id;
  $("name").value = p.name || "";
  $("price").value = p.price || "";
  $("description").value = p.description || "";
  photoData = p.photo || "";
  $("preview").src = photoData || PLACEHOLDER;
  $("formTitle").textContent = "Editează produsul";
  $("saveBtn").textContent = "Salvează modificările";
  window.scrollTo({top:0,behavior:"smooth"});
}

function deleteProduct(id){
  if(!confirm("Sigur vrei să ștergi acest produs?")) return;
  const old = products;
  products = products.filter(p=>p.id !== id);
  if(persist()){
    render();
    if(editingId === id) resetForm();
  }else{
    products = old;
  }
}

function makeAd(p){
  return p.name + "\n" +
    (p.price ? "Preț: " + p.price + " lei\n" : "Preț negociabil\n") +
    (p.description ? p.description + "\n" : "") +
    "\nDisponibil. Pentru detalii, mesaj privat!";
}

async function shareProduct(p){
  const text = makeAd(p);

  if(navigator.share){
    try{
      await navigator.share({title:p.name,text});
      return;
    }catch(e){}
  }

  try{
    await navigator.clipboard.writeText(text);
    alert("Textul anunțului a fost copiat. Îl poți lipi pe Facebook sau WhatsApp.");
  }catch(e){
    prompt("Copiază textul anunțului:",text);
  }
}

$("saveBtn").addEventListener("click",saveProduct);
$("cancelBtn").addEventListener("click",resetForm);

loadProducts();
resetForm();
render();
</script>
</body>
</html>
