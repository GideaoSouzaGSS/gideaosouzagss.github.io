<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RAG vs Fine-Tuning | Calculadora de Decisão</title>
    <style>
        :root {
            --bg-color: #0f172a;
            --card-bg: #1e293b;
            --text-primary: #f8fafc;
            --text-secondary: #94a3b8;
            --accent-rag: #3b82f6;
            --accent-ft: #ec4899;
            --border: #334155;
        }

        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-primary);
            display: flex;
            justify-content: center;
            padding: 2rem;
            line-height: 1.6;
        }

        .container {
            max-width: 800px;
            width: 100%;
        }

        h1 { margin-bottom: 0.5rem; }
        p.subtitle { color: var(--text-secondary); margin-bottom: 2rem; }

        .question-card {
            background-color: var(--card-bg);
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 1.5rem;
            margin-bottom: 1.5rem;
        }

        .question-title {
            font-size: 1.1rem;
            font-weight: 600;
            margin-bottom: 1rem;
            display: block;
        }

        .options {
            display: flex;
            flex-direction: column;
            gap: 0.8rem;
        }

        label {
            display: flex;
            align-items: center;
            gap: 10px;
            cursor: pointer;
            padding: 0.5rem;
            border-radius: 4px;
            transition: background 0.2s;
        }

        label:hover { background-color: rgba(255,255,255,0.05); }

        input[type="radio"] { accent-color: var(--accent-rag); transform: scale(1.2); }

        .btn-calculate {
            background: linear-gradient(90deg, var(--accent-rag), var(--accent-ft));
            color: white;
            border: none;
            padding: 1rem 2rem;
            font-size: 1.1rem;
            font-weight: bold;
            border-radius: 6px;
            cursor: pointer;
            width: 100%;
            margin-top: 1rem;
            transition: opacity 0.2s;
        }

        .btn-calculate:hover { opacity: 0.9; }

        #result-area {
            margin-top: 2rem;
            padding: 2rem;
            border-radius: 8px;
            display: none; /* Hidden by default */
            border: 2px solid;
        }

        .result-rag { background-color: rgba(59, 130, 246, 0.1); border-color: var(--accent-rag); }
        .result-ft { background-color: rgba(236, 72, 153, 0.1); border-color: var(--accent-ft); }
        .result-hybrid { background-color: rgba(168, 85, 247, 0.1); border-color: #a855f7; }

        .score-bar {
            display: flex;
            height: 20px;
            background: #333;
            margin-top: 1rem;
            border-radius: 10px;
            overflow: hidden;
        }
        .bar-rag { background-color: var(--accent-rag); }
        .bar-ft { background-color: var(--accent-ft); }

    </style>
</head>
<body>

<div class="container">
    <h1>Agente (RAG) vs. Fine-Tuning</h1>
    <p class="subtitle">Ferramenta de apoio à decisão arquitetural para projetos de IA.</p>

    <form id="quizForm">
        <div class="question-card">
            <span class="question-title">1. Com que frequência os dados/conhecimento mudam?</span>
            <div class="options">
                <label><input type="radio" name="q1" value="RAG:10" required> Alta frequência (Estoque, Preços, Notícias, Leis novas)</label>
                <label><input type="radio" name="q1" value="FT:5"> Baixa frequência ou Estático (Gramática, Estilo, Código legado, Manuais antigos)</label>
            </div>
        </div>

        <div class="question-card">
            <span class="question-title">2. Qual é a principal dor do cliente hoje?</span>
            <div class="options">
                <label><input type="radio" name="q2" value="RAG:10"> Falta de Conhecimento (A IA não sabe ou inventa fatos)</label>
                <label><input type="radio" name="q2" value="FT:10"> Falta de Estilo/Padrão (A IA sabe, mas fala errado ou fora do formato)</label>
            </div>
        </div>

        <div class="question-card">
            <span class="question-title">3. Qual a tolerância a erros factuais (Alucinação)?</span>
            <div class="options">
                <label><input type="radio" name="q3" value="RAG:10"> Zero (Jurídico, Saúde, Engenharia)</label>
                <label><input type="radio" name="q3" value="FT:5"> Média/Alta (Brainstorming, Marketing Criativo, Ficção)</label>
            </div>
        </div>

        <div class="question-card">
            <span class="question-title">4. O que temos de dados disponíveis AGORA?</span>
            <div class="options">
                <label><input type="radio" name="q4" value="RAG:5"> Documentos brutos (PDF, Docx, SQL, APIs)</label>
                <label><input type="radio" name="q4" value="FT:10"> Dataset Curado (>1000 pares de Pergunta/Resposta revisados por humanos)</label>
            </div>
        </div>

        <div class="question-card">
            <span class="question-title">5. O objetivo é raciocínio ou formatação?</span>
            <div class="options">
                <label><input type="radio" name="q5" value="RAG:10"> Raciocínio (Consultar fontes diferentes e concluir algo)</label>
                <label><input type="radio" name="q5" value="FT:5"> Formatação/Imitação (Seguir um JSON estrito, imitar um dialeto)</label>
            </div>
        </div>

        <button type="button" class="btn-calculate" onclick="calculate()">Calcular Recomendação</button>
    </form>

    <div id="result-area">
        <h2 id="result-title"></h2>
        <p id="result-desc"></p>
        
        <div style="margin-top: 1rem; font-size: 0.9rem; color: var(--text-secondary);">Distribuição de Pontos:</div>
        <div class="score-bar">
            <div id="bar-rag" class="bar-rag" style="width: 50%"></div>
            <div id="bar-ft" class="bar-ft" style="width: 50%"></div>
        </div>
        <div style="display: flex; justify-content: space-between; margin-top: 5px;">
            <span>Agente/RAG (<span id="score-rag-display">0</span>)</span>
            <span>Fine-Tuning (<span id="score-ft-display">0</span>)</span>
        </div>
    </div>
</div>

<script>
    function calculate() {
        const form = document.getElementById('quizForm');
        const formData = new FormData(form);
        
        let scoreRAG = 0;
        let scoreFT = 0;
        let answered = 0;

        for (let pair of formData.entries()) {
            answered++;
            const [type, points] = pair[1].split(':');
            if (type === 'RAG') scoreRAG += parseInt(points);
            if (type === 'FT') scoreFT += parseInt(points);
        }

        if (answered < 5) {
            alert("Por favor, responda todas as perguntas para uma análise precisa.");
            return;
        }

        const total = scoreRAG + scoreFT;
        const ragPercent = (scoreRAG / total) * 100;
        const ftPercent = (scoreFT / total) * 100;

        const resultArea = document.getElementById('result-area');
        const title = document.getElementById('result-title');
        const desc = document.getElementById('result-desc');
        const barRag = document.getElementById('bar-rag');
        const barFt = document.getElementById('bar-ft');

        // Lógica de Decisão
        resultArea.style.display = 'block';
        barRag.style.width = `${ragPercent}%`;
        barFt.style.width = `${ftPercent}%`;

        document.getElementById('score-rag-display').innerText = scoreRAG;
        document.getElementById('score-ft-display').innerText = scoreFT;

        resultArea.className = ''; // Reset classes

        if (scoreRAG > scoreFT + 10) {
            // Vitória Clara do RAG
            resultArea.classList.add('result-rag');
            title.innerText = "Recomendação: Agente de IA (RAG)";
            desc.innerHTML = "<strong>O cenário pede acesso a informações dinâmicas e precisão factual.</strong><br>Não gaste tempo com treinamento agora. Foque em Engenharia de Prompt e busca vetorial eficiente.";
        } else if (scoreFT > scoreRAG + 10) {
            // Vitória Clara do Fine-Tuning
            resultArea.classList.add('result-ft');
            title.innerText = "Recomendação: Fine-Tuning";
            desc.innerHTML = "<strong>O cenário pede consistência de estilo, formato rígido ou imitação de comportamento.</strong><br>O RAG sozinho provavelmente vai falhar na 'personalidade' ou formato. Prepare o dataset!";
        } else {
            // Zona Híbrida
            resultArea.classList.add('result-hybrid');
            title.innerText = "Recomendação: Híbrido (Comece com RAG)";
            desc.innerHTML = "<strong>Resultado equilibrado.</strong><br>A melhor estratégia técnica é começar com um Agente (RAG) para garantir o funcionamento. Colete logs por 2-3 meses e use esses dados reais para fazer um Fine-Tuning posterior (Fase 2) para refinar o estilo.";
        }
        
        // Scroll suave para o resultado
        resultArea.scrollIntoView({ behavior: 'smooth' });
    }
</script>

</body>
</html>
