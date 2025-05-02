<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Painel Semanal – Esther Carvalho dos Santos</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #f0f2f5;
      margin: 0;
      padding: 20px;
    }

    h1 {
      text-align: center;
      color: #333;
    }

    .kanban-board {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
      gap: 10px;
      margin-top: 20px;
    }

    .day-column {
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      padding: 10px;
      width: 13%;
      min-width: 150px;
      display: flex;
      flex-direction: column;
    }

    .day-column h3 {
      text-align: center;
      margin-bottom: 10px;
    }

    .tasks {
      flex-grow: 1;
      min-height: 100px;
      padding: 5px;
      background: #f9f9f9;
      border-radius: 5px;
    }

    .task {
      background-color: #dff0d8;
      padding: 6px;
      margin: 5px 0;
      border-radius: 5px;
      cursor: grab;
    }

    .task:hover {
      background-color: #c8e6c9;
    }

    .controls {
      margin-top: 10px;
      display: flex;
      gap: 5px;
    }

    button {
      flex: 1;
      padding: 4px;
      font-size: 12px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    .add-btn {
      background-color: #2196f3;
      color: white;
    }

    .edit-btn {
      background-color: #ffc107;
      color: white;
    }

    .delete-btn {
      background-color: #f44336;
      color: white;
    }

    @media (max-width: 800px) {
      .kanban-board {
        flex-direction: column;
        align-items: center;
      }
      .day-column {
        width: 90%;
      }
    }
  </style>
</head>
<body>

<h1>Painel de Programação Semanal – Esther Carvalho dos Santos</h1>

<div class="kanban-board" id="kanbanBoard"></div>

<script>
  const dias = ['Segunda', 'Terça', 'Quarta', 'Quinta', 'Sexta', 'Sábado', 'Domingo'];

  function criarPainel() {
    const board = document.getElementById('kanbanBoard');
    board.innerHTML = '';
    dias.forEach(dia => {
      const col = document.createElement('div');
      col.className = 'day-column';
      col.innerHTML = `
        <h3>${dia}</h3>
        <div class="tasks" id="${dia}" ondrop="drop(event)" ondragover="allowDrop(event)"></div>
        <button class="add-btn" onclick="adicionarTarefa('${dia}')">+ Adicionar</button>
      `;
      board.appendChild(col);
      carregarTarefas(dia);
    });
  }

  function adicionarTarefa(dia) {
    const texto = prompt(`Nova tarefa para ${dia}:`);
    if (texto) {
      const id = 'task-' + Date.now();
      const tarefa = { id, texto };
      const tarefas = JSON.parse(localStorage.getItem(dia) || '[]');
      tarefas.push(tarefa);
      localStorage.setItem(dia, JSON.stringify(tarefas));
      carregarTarefas(dia);
    }
  }

  function carregarTarefas(dia) {
    const container = document.getElementById(dia);
    container.innerHTML = '';
    const tarefas = JSON.parse(localStorage.getItem(dia) || '[]');
    tarefas.forEach(tarefa => {
      const div = document.createElement('div');
      div.className = 'task';
      div.draggable = true;
      div.id = tarefa.id;
      div.ondragstart = drag;
      div.innerHTML = `
        ${tarefa.texto}
        <div class="controls">
          <button class="edit-btn" onclick="editarTarefa('${dia}', '${tarefa.id}')">✏</button>
          <button class="delete-btn" onclick="excluirTarefa('${dia}', '${tarefa.id}')">🗑</button>
        </div>
      `;
      container.appendChild(div);
    });
  }

  function editarTarefa(dia, id) {
    const tarefas = JSON.parse(localStorage.getItem(dia) || '[]');
    const tarefa = tarefas.find(t => t.id === id);
    const novoTexto = prompt('Editar tarefa:', tarefa.texto);
    if (novoTexto) {
      tarefa.texto = novoTexto;
      localStorage.setItem(dia, JSON.stringify(tarefas));
      carregarTarefas(dia);
    }
  }

  function excluirTarefa(dia, id) {
    let tarefas = JSON.parse(localStorage.getItem(dia) || '[]');
    tarefas = tarefas.filter(t => t.id !== id);
    localStorage.setItem(dia, JSON.stringify(tarefas));
    carregarTarefas(dia);
  }

  function allowDrop(ev) {
    ev.preventDefault();
  }

  function drag(ev) {
    ev.dataTransfer.setData("text", ev.target.id);
  }

  function drop(ev) {
    ev.preventDefault();
    const tarefaId = ev.dataTransfer.getData("text");
    const tarefaDiv = document.getElementById(tarefaId);
    const origem = encontrarDia(tarefaId);
    const destino = ev.currentTarget.id;

    if (origem && destino && origem !== destino) {
      let tarefa;
      let tarefasOrigem = JSON.parse(localStorage.getItem(origem));
      tarefasOrigem = tarefasOrigem.filter(t => {
        if (t.id === tarefaId) tarefa = t;
        return t.id !== tarefaId;
      });
      localStorage.setItem(origem, JSON.stringify(tarefasOrigem));

      const tarefasDestino = JSON.parse(localStorage.getItem(destino) || '[]');
      tarefasDestino.push(tarefa);
      localStorage.setItem(destino, JSON.stringify(tarefasDestino));

      carregarTarefas(origem);
      carregarTarefas(destino);
    }
  }

  function encontrarDia(tarefaId) {
    for (const dia of dias) {
      const tarefas = JSON.parse(localStorage.getItem(dia) || '[]');
      if (tarefas.find(t => t.id === tarefaId)) return dia;
    }
    return null;
  }

  criarPainel();
</script>

</body>
</html>
