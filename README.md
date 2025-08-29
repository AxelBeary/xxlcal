<!DOCTYPE html>
<html lang="zh-Hans">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>XXLWOOFIA 角色技能升级券计算器（购物车版）</title>
  <style>
    :root{ --bg:#0b1220; --card:#121a2b; --muted:#8aa0c4; --text:#e7eefc; --accent:#7aa2ff; --ok:#2ccf75; --warn:#ffc658; }
    body{margin:0; font-family:ui-sans-serif,system-ui; background:var(--bg); color:var(--text)}
    .wrap{max-width:1300px; margin:28px auto; padding:0 16px}
    .card{background:var(--card); border:1px solid #1e2a44; border-radius:12px; padding:14px; margin-bottom:14px}
    .controls{display:flex; gap:12px; flex-wrap:wrap; align-items:center}
    label{font-size:14px}
    input[type=number]{width:72px; padding:6px; border-radius:8px; border:1px solid #24365a; background:#0f1626; color:var(--text)}
    select{padding:6px; border-radius:8px; border:1px solid #24365a; background:#0f1626; color:var(--text)}
    .btn{padding:8px 12px; border-radius:10px; border:none; cursor:pointer; font-weight:700; background:linear-gradient(135deg,#3a64ff,#7aa2ff); color:white; transition:0.2s}
    .btn:hover{opacity:0.85}
    table{width:100%; border-collapse:collapse; font-size:13px; margin-top:8px}
    th,td{padding:6px 8px; border-bottom:1px solid #1e2a44; text-align:right}
    th:first-child,td:first-child{text-align:left}
    tfoot td{font-weight:700}
    .note{font-size:13px; color:var(--muted); margin-top:8px}
    .cart-list{margin-top:8px; font-size:14px}
    .cart-item{padding:6px; border-bottom:1px solid #1e2a44}
    .cart-item span{color:var(--accent)}
  </style>
</head>
<body>
<div class="wrap">
  <h1>XXLWOOFIA 技能升级券计算器</h1>
  <div class="card">
    <div class="controls">
      <label>角色品质
        <select id="quality">
          <option value="xxl" selected>XXL</option>
          <option value="xl">XL</option>
          <option value="l">L</option>
          <option value="m">M</option>
        </select>
      </label>

      <div style="display:flex;gap:8px;align-items:center;flex-wrap:wrap">
        <div id="skills"></div>
      </div>

      <button class="btn" id="calc">计算</button>
      <button class="btn" id="reset" style="background:#1a2540">重置技能</button>
      <button class="btn" id="addCart" style="background:#2ccf75">加入购物车</button>
      <button class="btn" id="checkout" style="background:#ffc658;color:#000">结算</button>
    </div>
    <div class="note">输入当前技能等级（1–9），填 0 跳过该技能。填入无效数字会当0处理。<br>计算后可保存多个角色结果，点击结算按钮汇总，如需清空请刷新页面。<br><a href="https://space.bilibili.com/12370600" target="_blank" style="color:inherit;text-decoration:underline;"> AxelBeary 奚怡熊</a> 2025.8，Apache-2.0 license，建议手机端横屏使用。</div>
  </div>

  <div class="card">
    <h2>汇总（当前角色）</h2>
    <div id="summary"></div>
  </div>

  <div class="card">
    <h2>分技能明细</h2>
    <table id="detail">
      <thead>
        <tr>
          <th>技能</th>
          <th>入门战斗</th><th>初级战斗</th><th>中级战斗</th><th>进阶战斗</th><th>高级战斗</th>
          <th>入门必杀</th><th>初级必杀</th><th>中级必杀</th><th>进阶必殺</th><th>高级必杀</th>
          <th>入门修行</th><th>初级修行</th><th>中级修行</th><th>進階修行</th><th>高级修行</th>
          <th>兑换券（估算）</th>
        </tr>
      </thead>
      <tbody></tbody>
      <tfoot>
        <tr>
          <td>合计</td>
          <td id="sumCBeg">0</td><td id="sumCBas">0</td><td id="sumCMid">0</td><td id="sumCAdv">0</td><td id="sumCHigh">0</td>
          <td id="sumBBeg">0</td><td id="sumBBas">0</td><td id="sumBMid">0</td><td id="sumBAdv">0</td><td id="sumBHigh">0</td>
          <td id="sumTBeg">0</td><td id="sumTBas">0</td><td id="sumTMid">0</td><td id="sumTAdv">0</td><td id="sumTHigh">0</td>
          <td id="sumVoucher">0</td>
        </tr>
      </tfoot>
    </table>
  </div>

  <div class="card">
    <h2>购物车</h2>
    <div class="cart-list" id="cartList"></div>
  </div>
</div>

<script>
(function(){
  const ids=[1,2,3,4,5,6,7];
  const skillsContainer=document.getElementById('skills');
  ids.forEach(i=>{
    const wrapper=document.createElement('span');
    wrapper.style.display='inline-block'; wrapper.style.marginRight='6px';
    wrapper.innerHTML=`<label style="font-size:13px">技能${i} <input type="number" id="s${i}" min="0" max="9" value="0"/></label>`;
    skillsContainer.appendChild(wrapper);
  });

  const transToSeg=[0,0,1,1,2,2,3,3,4];
  const DATA={
    xxl:{normal:{combatTransitions:[30,35,35,40,40,45,45,50,50],burstTransitions:[0,0,0,0,0,0,0,0,0],trainTransitions:[0,0,0,0,0,0,0,0,0],voucherFull:3250},
         passive:{combatTransitions:[18,21,21,24,24,27,27,30,30],burstTransitions:[0,0,0,0,0,0,0,0,0],trainTransitions:[8,9,9,11,11,13,13,15,15],voucherFull:3810},
         ult:{combatTransitions:[54,63,63,72,72,81,81,90,90],burstTransitions:[24,27,27,33,33,39,39,45,45],trainTransitions:[0,0,0,0,0,0,0,0,0],voucherFull:25380}},
    xl:{normal:{combatTransitions:[10,15,15,20,20,25,25,30,30],burstTransitions:[0,0,0,0,0,0,0,0,0],trainTransitions:[0,0,0,0,0,0,0,0,0],voucherFull:1750},
        passive:{combatTransitions:[12,14,14,16,16,18,18,20,20],burstTransitions:[0,0,0,0,0,0,0,0,0],trainTransitions:[6,7,7,8,8,9,9,10,10],voucherFull:2600},
        ult:{combatTransitions:[36,42,42,48,48,54,54,60,60],burstTransitions:[18,21,21,24,24,27,27,30,30],trainTransitions:[0,0,0,0,0,0,0,0,0],voucherFull:17550}},
    l:{normal:{combatTransitions:Array(9).fill(10),burstTransitions:Array(9).fill(0),trainTransitions:Array(9).fill(0),voucherFull:750},
       ult:{combatTransitions:Array(9).fill(10),burstTransitions:Array(9).fill(5),trainTransitions:Array(9).fill(0),voucherFull:3375},
       passive:{combatTransitions:Array(9).fill(10),burstTransitions:Array(9).fill(0),trainTransitions:Array(9).fill(3),voucherFull:1200}},
    m:{normal:{combatTransitions:Array(9).fill(10),burstTransitions:Array(9).fill(0),trainTransitions:Array(9).fill(0),voucherFull:750},
       ult:{combatTransitions:Array(9).fill(10),burstTransitions:Array(9).fill(5),trainTransitions:Array(9).fill(0),voucherFull:3375},
       passive:{combatTransitions:Array(9).fill(10),burstTransitions:Array(9).fill(0),trainTransitions:Array(9).fill(3),voucherFull:1200}}
  };

  function calcForLevelTransitions(cur,spec){
    if(!Number.isFinite(cur)||cur<=0) return {combatSeg:[0,0,0,0,0],burstSeg:[0,0,0,0,0],trainSeg:[0,0,0,0,0],voucher:0};
    const startIndex=Math.max(0,cur-1);
    const combatSeg=[0,0,0,0,0],burstSeg=[0,0,0,0,0],trainSeg=[0,0,0,0,0];
    const cArr=spec.combatTransitions,bArr=spec.burstTransitions,tArr=spec.trainTransitions;
    let total=0,full=0;
    for(let i=0;i<cArr.length;i++) full+=(cArr[i]||0)+(bArr[i]||0)+(tArr[i]||0);
    for(let i=startIndex;i<9;i++){
      const seg=transToSeg[i]||0,cv=cArr[i]||0,bv=bArr[i]||0,tv=tArr[i]||0;
      combatSeg[seg]+=cv; burstSeg[seg]+=bv; trainSeg[seg]+=tv; total+=cv+bv+tv;
    }
    const voucher=Math.round(full?(total/full)*(spec.voucherFull||0):0);
    return {combatSeg,burstSeg,trainSeg,voucher};
  }

  function renderResults(){
    const quality=document.getElementById('quality').value;
    const kinds=['normal','ult','passive','passive','passive','passive','passive'];
    const tbody=document.querySelector('#detail tbody'); tbody.innerHTML='';
    const totals={combat:[0,0,0,0,0],burst:[0,0,0,0,0],train:[0,0,0,0,0],voucher:0};
    ids.forEach((skillIdx,idx)=>{
      const lv=parseInt(document.getElementById('s'+skillIdx).value,10)||0;
      const spec=DATA[quality][kinds[idx]];
      const res=calcForLevelTransitions(lv,spec);
      res.combatSeg.forEach((v,i)=>totals.combat[i]+=v);
      res.burstSeg.forEach((v,i)=>totals.burst[i]+=v);
      res.trainSeg.forEach((v,i)=>totals.train[i]+=v);
      totals.voucher+=res.voucher;
      const tr=document.createElement('tr');
      tr.innerHTML='<td>技能'+skillIdx+'</td>'+
        res.combatSeg.map(x=>`<td>${x}</td>`).join('')+
        res.burstSeg.map(x=>`<td>${x}</td>`).join('')+
        res.trainSeg.map(x=>`<td>${x}</td>`).join('')+
        `<td>${res.voucher}</td>`;
      tbody.appendChild(tr);
    });
    document.getElementById('sumCBeg').textContent=totals.combat[0];
    document.getElementById('sumCBas').textContent=totals.combat[1];
    document.getElementById('sumCMid').textContent=totals.combat[2];
    document.getElementById('sumCAdv').textContent=totals.combat[3];
    document.getElementById('sumCHigh').textContent=totals.combat[4];
    document.getElementById('sumBBeg').textContent=totals.burst[0];
    document.getElementById('sumBBas').textContent=totals.burst[1];
    document.getElementById('sumBMid').textContent=totals.burst[2];
    document.getElementById('sumBAdv').textContent=totals.burst[3];
    document.getElementById('sumBHigh').textContent=totals.burst[4];
    document.getElementById('sumTBeg').textContent=totals.train[0];
    document.getElementById('sumTBas').textContent=totals.train[1];
    document.getElementById('sumTMid').textContent=totals.train[2];
    document.getElementById('sumTAdv').textContent=totals.train[3];
    document.getElementById('sumTHigh').textContent=totals.train[4];
    document.getElementById('sumVoucher').textContent=totals.voucher;
    document.getElementById('summary').innerHTML=
      `战斗心得 — 入门:${totals.combat[0]}, 初级:${totals.combat[1]}, 中级:${totals.combat[2]}, 进階:${totals.combat[3]}, 高级:${totals.combat[4]}<br/>`+
      `必杀心得 — 入门:${totals.burst[0]}, 初级:${totals.burst[1]}, 中级:${totals.burst[2]}, 进階:${totals.burst[3]}, 高级:${totals.burst[4]}<br/>`+
      `修行心得 — 入门:${totals.train[0]}, 初级:${totals.train[1]}, 中级:${totals.train[2]}, 进階:${totals.train[3]}, 高级:${totals.train[4]}<br/>`+
      `预计兑换券:${totals.voucher}`;
    return totals;
  }

  document.getElementById('calc').addEventListener('click',renderResults);
  document.getElementById('reset').addEventListener('click',()=>{document.getElementById('quality').value='xxl';ids.forEach(i=>{document.getElementById('s'+i).value=1});renderResults()});

  // === 购物车 ===
  const cart=[];
  document.getElementById('addCart').addEventListener('click',()=>{
    const totals=renderResults();
    const quality=document.getElementById('quality').value.toUpperCase();
    cart.push({quality,totals});
    renderCart();
  });
  function renderCart(){
    const list=document.getElementById('cartList'); list.innerHTML='';
    cart.forEach((item,idx)=>{
      const div=document.createElement('div');
      div.className='cart-item';
      div.innerHTML=`<span>角色${idx+1} [${item.quality}]</span> — 战斗券:${item.totals.combat.reduce((a,b)=>a+b,0)}, 必杀券:${item.totals.burst.reduce((a,b)=>a+b,0)}, 修行券:${item.totals.train.reduce((a,b)=>a+b,0)}, 兑换券:${item.totals.voucher}`;
      list.appendChild(div);
    });
  }
  document.getElementById('checkout').addEventListener('click',()=>{
    if(!cart.length){alert('购物车为空');return}
    const final={combat:[0,0,0,0,0],burst:[0,0,0,0,0],train:[0,0,0,0,0],voucher:0};
    cart.forEach(item=>{
      item.totals.combat.forEach((v,i)=>final.combat[i]+=v);
      item.totals.burst.forEach((v,i)=>final.burst[i]+=v);
      item.totals.train.forEach((v,i)=>final.train[i]+=v);
      final.voucher+=item.totals.voucher;
    });
    alert(`结算结果：\n战斗心得: 入门${final.combat[0]}, 初级${final.combat[1]}, 中级${final.combat[2]}, 进阶${final.combat[3]}, 高级${final.combat[4]}\n`+
          `必杀心得: 入门${final.burst[0]}, 初级${final.burst[1]}, 中级${final.burst[2]}, 进阶${final.burst[3]}, 高级${final.burst[4]}\n`+
          `修行心得: 入门${final.train[0]}, 初级${final.train[1]}, 中级${final.train[2]}, 进阶${final.train[3]}, 高级${final.train[4]}\n`+
          `兑换券总数:${final.voucher}`);
  });

  renderResults();
})();
</script>
</body>
</html>
