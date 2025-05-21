<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>🛒 Estoque Animado</title>
  <style>
    * {
      box-sizing: border-box;
    }
    body, html {
      margin: 0; padding: 0; height: 100%;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      color: white;
      overflow-x: hidden;
    }
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      min-height: 100vh;
      padding: 20px;
      background-size: 300% 300%;
      animation: bgMove 15s ease infinite;
    }

    @keyframes bgMove {
      0% {background-position:0% 50%;}
      50% {background-position:100% 50%;}
      100% {background-position:0% 50%;}
    }

    body.tema-neon {
      background-image: linear-gradient(270deg, #ff00cc, #3333ff, #00ffff, #ff00cc);
    }
    body.tema-pastel {
      background-image: linear-gradient(270deg, #fbc2eb, #a6c1ee, #fbc2eb);
    }
    body.tema-neutro {
      background-image: linear-gradient(270deg, #000000, #555555, #bbbbbb, #eeeeee);
    }

    .container {
      width: 100%;
      max-width: 700px;
      background: rgba(0,0,0,0.35);
      border-radius: 12px;
      padding: 20px;
      box-shadow: 0 0 15px rgba(255,255,255,0.15);
    }

    h1 {
      text-align: center;
      margin-bottom: 20px;
    }

    label {
      font-weight: bold;
      margin-top: 15px;
      display: block;
    }

    input[type=text], input[type=number], select {
      width: 100%;
      padding: 10px;
      margin-top: 5px;
      border-radius: 6px;
      border: none;
      font-size: 1em;
      background: rgba(255,255,255,0.2);
      color: rgb(255, 255, 255);
    }

    input::placeholder {
      color: #ddd;
    }

    button {
      margin-top: 15px;
      padding: 12px;
      width: 100%;
      border: none;
      border-radius: 8px;
      font-weight: bold;
      cursor: pointer;
      background: #fff;
      color: #222;
      transition: background 0.3s ease;
    }
    button:hover {
      background: #eee;
    }

    .produtos-lista {
      margin-top: 20px;
      max-height: 250px;
      overflow-y: auto;
    }

    .produto-item {
      background: rgba(255, 255, 255, 0.1);
      margin: 8px 0;
      padding: 12px;
      border-radius: 8px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      font-size: 1em;
      box-shadow: 0 2px 5px rgba(0,0,0,0.2);
      transition: transform 0.2s ease;
    }
    .produto-item:hover {
      transform: scale(1.02);
    }
    .produto-item span {
      font-weight: bold;
      font-size: 1.1em;
    }
    .produto-baixo {
      border-left: 5px solid #ff5050;
      background-color: rgba(255, 80, 80, 0.15);
    }
    .produto-ok {
      border-left: 5px solid #00ff99;
      background-color: rgba(0, 255, 153, 0.08);
    }

    #tema {
      width: 200px;
      margin-bottom: 20px;
    }
  </style>
</head>
<body class="tema-neon">
  <div class="container">
    <h1>🛒 Sistema de Estoque Animado</h1>

    <label for="tema">🎨 Selecione Tema:</label>
    <select id="tema" onchange="trocarTema()">
      <option value="neon" selected>Neon</option>
      <option value="pastel">Pastel</option>
      <option value="neutro">Neutro</option>
    </select>

    <label for="nomeProduto">Nome do Produto:</label>
    <input type="text" id="nomeProduto" placeholder="Ex: Camiseta" />

    <label for="quantidadeProduto">Quantidade Inicial:</label>
    <input type="number" id="quantidadeProduto" placeholder="Ex: 10" min="0" />

    <button onclick="cadastrarProduto()">📦 Cadastrar Produto</button>

    <hr style="margin: 25px 0; border-color: rgba(255,255,255,0.2)" />

    <label for="produtoSelecionado">Produto:</label>
    <select id="produtoSelecionado">
      <option value="">-- Selecione um produto --</option>
    </select>

    <label for="quantidadeAcao">Quantidade:</label>
    <input type="number" id="quantidadeAcao" placeholder="Ex: 5" min="1" />

    <button onclick="adicionarUnidades()">➕ Adicionar unidades</button>
    <button onclick="retirarUnidades()">➖ Retirar unidades</button>

    <hr style="margin: 25px 0; border-color: rgba(255,255,255,0.2)" />

    <button onclick="listarProdutos()">📋 Listar Todos os Produtos</button>
    <button onclick="listarBaixoEstoque()">⚠️ Mostrar Estoque Baixo (menos de 5)</button>

    <div id="estoque" class="produtos-lista"></div>
  </div>

  <script>
    let produtos = [];

    function atualizarProdutosSelect() {
      const select = document.getElementById('produtoSelecionado');
      select.innerHTML = '<option value="">-- Selecione um produto --</option>';
      produtos.forEach((p, i) => {
        const option = document.createElement('option');
        option.value = i;
        option.textContent = `${p.nome} (${p.quantidade})`;
        select.appendChild(option);
      });
    }

    function cadastrarProduto() {
      const nome = document.getElementById('nomeProduto').value.trim();
      const qtd = parseInt(document.getElementById('quantidadeProduto').value);
      if (!nome) return alert('Digite o nome do produto.');
      if (isNaN(qtd) || qtd < 0) return alert('Quantidade inicial inválida.');

      produtos.push({ nome, quantidade: qtd });
      atualizarProdutosSelect();

      document.getElementById('nomeProduto').value = '';
      document.getElementById('quantidadeProduto').value = '';
      listarProdutos();
    }

    function adicionarUnidades() {
      const select = document.getElementById('produtoSelecionado');
      const idx = select.value;
      const qtd = parseInt(document.getElementById('quantidadeAcao').value);
      if (idx === '') return alert('Selecione um produto.');
      if (isNaN(qtd) || qtd <= 0) return alert('Digite uma quantidade válida.');

      produtos[idx].quantidade += qtd;
      atualizarProdutosSelect();
      listarProdutos();
      alert('Unidades adicionadas!');
    }

    function retirarUnidades() {
      const select = document.getElementById('produtoSelecionado');
      const idx = select.value;
      const qtd = parseInt(document.getElementById('quantidadeAcao').value);
      if (idx === '') return alert('Selecione um produto.');
      if (isNaN(qtd) || qtd <= 0) return alert('Digite uma quantidade válida.');

      if (produtos[idx].quantidade < qtd) {
        alert('Estoque insuficiente!');
        return;
      }

      produtos[idx].quantidade -= qtd;
      atualizarProdutosSelect();
      listarProdutos();
      alert('Unidades retiradas!');
    }

    function listarProdutos() {
      const div = document.getElementById('estoque');
      div.innerHTML = '<h3>📦 Produtos em Estoque:</h3>';
      
      if (produtos.length === 0) {
        div.innerHTML += '<p>Nenhum produto cadastrado.</p>';
        return;
      }

      produtos.forEach(p => {
        const classe = p.quantidade < 5 ? 'produto-baixo' : 'produto-ok';
        const icone = p.quantidade < 5 ? '⚠️' : '✅';
        div.innerHTML += `
          <div class="produto-item ${classe}">
            <span>${icone} ${p.nome}</span>
            <span>${p.quantidade} un.</span>
          </div>
        `;
      });
    }

    function listarBaixoEstoque() {
      const div = document.getElementById('estoque');
      div.innerHTML = '<h3>⚠️ Produtos com Estoque Baixo (menos de 5):</h3>';
      const baixos = produtos.filter(p => p.quantidade < 5);
      if (baixos.length === 0) {
        div.innerHTML += '<p>Todos os produtos estão com estoque suficiente.</p>';
        return;
      }
      baixos.forEach(p => {
        div.innerHTML += `
          <div class="produto-item produto-baixo">
            <span>⚠️ ${p.nome}</span>
            <span>${p.quantidade} un.</span>
          </div>
        `;
      });
    }

    function trocarTema() {
      const tema = document.getElementById('tema').value;
      document.body.className = '';
      document.body.classList.add(`tema-${tema}`);
    }

    window.onload = () => {
      trocarTema();
    }
  </script>
</body>
</html>
