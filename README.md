[index.html](https://github.com/user-attachments/files/24911300/index.html)
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<title>Time Planner</title>

<style>
body{
  font-family:Arial, sans-serif;
  background:#f4f6fa;
  margin:0;
  padding:20px;
  display:flex;
  gap:20px;
}
#grid{
  display:grid;
  grid-template-columns:90px repeat(12,20px);
  border:1px solid #ccc;
  background:white;
  border-radius:10px;
  overflow:hidden;
}
#side{
  width:280px;
  background:white;
  border:1px solid #ccc;
  border-radius:8px;
  padding:10px;
  overflow-y:auto;
  height: calc(20 * 40px);
}
#side h3{text-align:center;}

.side-item{
  display:flex;
  align-items:flex-start;
  gap:8px;
  border:1px solid #ddd;
  border-radius:6px;
  padding:6px;
  padding-right:80px;
  margin-bottom:6px;
  font-size:12px;
  position:relative;
  overflow:hidden;
}

.side-item .text-box{
  flex:1;
  min-width:0;
  max-width:100%;
  word-break:break-word;
  overflow-wrap:break-word;
  white-space:normal;
}

.delete-btn{
  position:absolute;
  top:6px;
  border:none;
  color:white;
  border-radius:4px;
  font-size:10px;
  cursor:pointer;
  padding:2px 6px;
}

.color-box{width:16px;height:16px;border-radius:4px;}

.time{
  height:40px;width:90px;background:#f1f3f6;
  display:flex;align-items:center;justify-content:center;
  font-size:11px;
  box-shadow: inset -1px 0 0 #ddd, inset 0 -1px 0 #ddd;
}

.cell{
  height:40px;
  width:20px;
  cursor:pointer;
  box-shadow: inset -1px 0 0 #eee, inset 0 -1px 0 #eee;
}

.cell.v-strong{
  box-shadow: inset -1px 0 0 #cfcfcf, inset 0 -1px 0 #eee;
}

.modal{position:fixed;inset:0;background:rgba(0,0,0,.5);display:none;align-items:center;justify-content:center;z-index:10;}
.modal-box{background:white;padding:20px;border-radius:10px;width:300px;}
#addFromSide{width:100%;margin-bottom:10px;padding:6px;border:none;border-radius:6px;background:#3f8cff;color:white;cursor:pointer;}

.drag-preview{background: rgba(63,140,255,0.3);}
</style>
</head>

<body>
<div><div id="grid"></div></div>
<div id="side">
  <h3>📌 일정 목록</h3>
  <button id="addFromSide">📋 일정 추가</button>
  <div id="list"></div>
</div>

<div class="modal" id="modal">
  <div class="modal-box">
    <h3 id="modalTitle">일정 입력</h3>
    <input id="titleInput" placeholder="제목" style="width:100%"><br><br>
    <textarea id="detailInput" placeholder="세부내용" style="width:100%"></textarea><br><br>
    <label>시작</label><select id="editStart"></select><br><br>
    <label>종료</label><select id="editEnd"></select><br><br>
    <input type="color" id="colorInput"><br><br>
    <button id="save">저장</button>
    <button id="cancel">취소</button>
  </div>
</div>

<script>
const grid=document.getElementById("grid");
const listBox=document.getElementById("list");
const modal=document.getElementById("modal");
const modalTitle=document.getElementById("modalTitle");
const saveBtn=document.getElementById("save");
const cancelBtn=document.getElementById("cancel");

const titleInput=document.getElementById("titleInput");
const detailInput=document.getElementById("detailInput");
const colorInput=document.getElementById("colorInput");
const editStart=document.getElementById("editStart");
const editEnd=document.getElementById("editEnd");

const addFromSideBtn=document.getElementById("addFromSide");

let editingItem=null;
let isEditMode=false;
let backupData=null;   // 🔥 백업 데이터

let isDragging = false;
let dragStartIndex = null;
let dragEndIndex = null;

const times=[];
for(let i=7;i<27;i++){
  let h=i%24;
  for(let j=0;j<12;j++){
    times.push(String(h).padStart(2,"0")+":"+String(j*5).padStart(2,"0"));
  }
}

[times].forEach(arr=>{
  arr.forEach(t=>{
    [editStart,editEnd].forEach(sel=>{
      const o=document.createElement("option");
      o.value=t;o.innerText=t;
      sel.appendChild(o.cloneNode(true));
    });
  });
});

function clearDragPreview(){
  document.querySelectorAll(".cell.drag-preview").forEach(c=>c.classList.remove("drag-preview"));
}

function paintDragPreview(s,e){
  clearDragPreview();
  for(let i=s;i<=e;i++){
    const c=document.querySelector(`.cell[data-index='${i}']`);
    if(c) c.classList.add("drag-preview");
  }
}

let cellIndex=0;
for(let r=0;r<20;r++){
  const timeCell=document.createElement("div");
  timeCell.className="time";
  timeCell.innerText=times[r*12];
  grid.appendChild(timeCell);

  for(let c=0;c<12;c++){
    const cell=document.createElement("div");
    cell.className="cell";
    cell.dataset.index=cellIndex;

    if((cellIndex+1)%2===0 && (cellIndex+1)%12!==0){
      cell.classList.add("v-strong");
    }

    cell.onmousedown = ()=>{
      isDragging = true;
      dragStartIndex = Number(cell.dataset.index);
      dragEndIndex = dragStartIndex;
      paintDragPreview(dragStartIndex, dragEndIndex);
    };

    cell.onmouseenter = ()=>{
      if(isDragging){
        dragEndIndex = Number(cell.dataset.index);
        let s = Math.min(dragStartIndex, dragEndIndex);
        let e = Math.max(dragStartIndex, dragEndIndex);
        paintDragPreview(s, e);
      }
    };

    cell.onmouseup = ()=>{
      if(!isDragging) return;

      isDragging = false;
      clearDragPreview();

      let s = Math.min(dragStartIndex, dragEndIndex);
      let e = Math.max(dragStartIndex, dragEndIndex) + 1;

      modalTitle.innerText="일정 추가";
      isEditMode=false;

      titleInput.value="";
      detailInput.value="";
      colorInput.value="#3f8cff";

      editStart.value=times[s];
      editEnd.value=times[e] || times[times.length-1];

      modal.style.display="flex";
    };

    grid.appendChild(cell);
    cellIndex++;
  }
}

document.body.onmouseup = ()=>{
  isDragging = false;
  clearDragPreview();
};

function clearCells(s,e){
  for(let i=s;i<e;i++){
    const c=document.querySelector(`.cell[data-index='${i}']`);
    if(c) c.style.background="";
  }
}

function paintCells(s,e,color){
  for(let i=s;i<e;i++){
    const c=document.querySelector(`.cell[data-index='${i}']`);
    if(c) c.style.background=color;
  }
}

function sortSideList(){
  const items = Array.from(document.querySelectorAll(".side-item"));
  items.sort((a,b)=> Number(a.dataset.start) - Number(b.dataset.start));
  items.forEach(item => listBox.appendChild(item));
}

function createSideItem(title,detail,color,sIdx,eIdx){
  const item=document.createElement("div");
  item.className="side-item";
  item.dataset.start=sIdx;
  item.dataset.end=eIdx;
  item.dataset.title=title;
  item.dataset.detail=detail;
  item.dataset.color=color;

  const colorBox=document.createElement("div");
  colorBox.className="color-box";
  colorBox.style.background=color;

  const text=document.createElement("div");
  text.className="text-box";
  text.innerHTML=`<b>${title}</b><br><small>${detail}</small><br><small>⏰ ${times[sIdx]} ~ ${times[eIdx]}</small>`;

  const editBtn=document.createElement("button");
  editBtn.className="delete-btn";
  editBtn.style.right="42px";
  editBtn.style.background="#4caf50";
  editBtn.innerText="수정";

  editBtn.onclick=()=>{
    isEditMode=true;
    editingItem=item;

    backupData = {
      s: Number(item.dataset.start),
      e: Number(item.dataset.end),
      color: item.dataset.color
    };

    modalTitle.innerText="일정 수정";

    titleInput.value=item.dataset.title;
    detailInput.value=item.dataset.detail;
    colorInput.value=item.dataset.color;
    editStart.value=times[item.dataset.start];
    editEnd.value=times[item.dataset.end];

    clearCells(item.dataset.start,item.dataset.end);
    modal.style.display="flex";
  };

  const del=document.createElement("button");
  del.className="delete-btn";
  del.style.right="6px";
  del.style.background="#ff5c5c";
  del.innerText="삭제";

  del.onclick=()=>{
    clearCells(Number(item.dataset.start), Number(item.dataset.end));
    item.remove();
  };

  item.append(colorBox,text,editBtn,del);
  listBox.appendChild(item);
  sortSideList();
}

saveBtn.onclick=()=>{
  const title=titleInput.value.trim();
  const detail=detailInput.value.trim();
  const color=colorInput.value;
  const sIdx=times.indexOf(editStart.value);
  const eIdx=times.indexOf(editEnd.value);

  if(eIdx <= sIdx){ alert("⚠ 종료 시간은 시작 시간보다 늦어야 합니다!"); return; }
  if(title === ""){ alert("⚠ 제목을 입력해주세요!"); return; }

  if(isEditMode && editingItem){ editingItem.remove(); }

  paintCells(sIdx,eIdx,color);
  createSideItem(title,detail,color,sIdx,eIdx);

  modal.style.display="none";
  isEditMode=false;
  editingItem=null;
  backupData=null;
};

cancelBtn.onclick=()=>{
  if(isEditMode && backupData){
    paintCells(backupData.s, backupData.e, backupData.color);
  }

  modal.style.display="none";
  isEditMode=false;
  editingItem=null;
  backupData=null;
};

addFromSideBtn.onclick=()=>{
  isEditMode=false;
  modalTitle.innerText="일정 추가";
  titleInput.value="";
  detailInput.value="";
  colorInput.value="#3f8cff";
  editStart.value=times[0];
  editEnd.value=times[1];
  modal.style.display="flex";
};
</script>
</body>
</html>
