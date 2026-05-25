<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GitHub Neon Stats Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --bg-color: #0d020b;
            --card-bg: #160718;
            --neon-pink: #ff55b8;
            --text-color: #ffffff;
            --border-glow: 0 0 12px rgba(255, 85, 184, 0.4);
        }

   body {
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: 'Segoe UI', Arial, sans-serif;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
      /* Input de busca para testar qualquer usuário */
        .search-container {
            margin-bottom: 20px;
        }
        .search-container input {
            background: #250c29;
            border: 1px solid var(--neon-pink);
            color: white;
            padding: 8px 15px;
            border-radius: 4px;
            outline: none;
        }
        .search-container button {
            background: var(--neon-pink);
            border: none;
            color: white;
            padding: 8px 15px;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
        }
     .dashboard {
            width: 100%;
            max-width: 1100px;
            border: 2px solid var(--neon-pink);
            box-shadow: var(--border-glow);
            border-radius: 16px;
            padding: 30px;
            background: linear-gradient(145deg, #0d020b, #110310);
        }
     .header {
            text-align: center;
            margin-bottom: 30px;
            position: relative;
        }
      .logo-container {
            width: 70px;
            height: 70px;
            border: 2px solid var(--neon-pink);
            border-radius: 50%;
            line-height: 70px;
            font-size: 28px;
            margin: 0 auto 15px;
            box-shadow: var(--border-glow);
            font-weight: bold;
            color: var(--neon-pink);
            text-shadow: var(--border-glow);
        }
     .header h1 {
            margin: 0;
            font-size: 24px;
            letter-spacing: 2px;
        }
        .grid-container {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 20px;
        }
        .card {
            background: var(--card-bg);
            border: 1px solid rgba(255, 85, 184, 0.5);
            border-radius: 10px;
            padding: 20px;
            text-align: center;
            box-shadow: inset 0 0 8px rgba(255, 85, 184, 0.1);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }

  /* Modificadores de Grid baseados na imagem */
        .span-2 { grid-column: span 2; }
        .row-2 { grid-row: span 2; }
        h3 {
            margin: 0 0 10px 0;
            font-size: 12px;
            letter-spacing: 1.5px;
            color: #ff99dc;
            text-transform: uppercase;
        }
        .number {
            font-size: 36px;
            font-weight: bold;
            color: var(--neon-pink);
            text-shadow: 0 0 8px rgba(255, 85, 184, 0.5);
        }

  .number.large {
            font-size: 52px;
        }
        .chart-wrapper {
            width: 100%;
            height: 100%;
            min-height: 150px;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        canvas {
        max-width: 100%;
        }
    </style>
</head>
<body>
    <div class="search-container">
        <input type="text" id="usernameInput" value="hamarilys" placeholder="Usuário do GitHub">
        <button onclick="fetchGitHubStats()">Buscar</button>
    </div>
    <div class="dashboard">
        <header class="header">
            <div class="logo-container" id="userInitial">H</div>
            <h1 id="dashboardTitle">hamarilys's GitHub Stats</h1>
        </header>

   <main class="grid-container">
            <div class="card span-2">
                <h3>TOTAL STARS RECEIVED</h3>
                <div class="number large" id="totalStars">0</div>
            </div>
    <div class="card row-2">
                <h3>TOP LANGUAGES</h3>
                <div class="chart-wrapper">
                    <canvas id="languagesChart"></canvas>
                </div>
            </div>
    <div class="card span-2">
                <h3>COMMIT FREQUENCY (YEAR)</h3>
                <div class="chart-wrapper">
                    <canvas id="frequencyChart"></canvas>
                </div>
            </div>
            <div class="card">
                <h3>PUBLIC REPOS</h3>
                <div class="number" id="publicRepos">0</div>
            </div>
            <div class="card">
                <h3>FOLLOWERS</h3>
                <div class="number" id="followers">0</div>
            </div>
            <div class="card">
                <h3>FOLLOWING</h3>
                <div class="number" id="following">0</div>
            </div>
            <div class="card">
                <h3>GISTS</h3>
                <div class="number" id="publicGists">0</div>
            </div>
        </main>
    </div>

  <script>
        // Variáveis globais para destruir gráficos anteriores ao buscar novo usuário
        let langChartInstance = null;
        let freqChartInstance = null;

        async function fetchGitHubStats() {
            const username = document.getElementById('usernameInput').value.trim();
            if (!username) return;

            try {
                // 1. Buscar dados do perfil
                const userResponse = await fetch(`https://api.github.com/users/${username}`);
                if (!userResponse.ok) throw new Error('Usuário não encontrado');
                const userData = await userResponse.json();

                // Atualizar textos e cabeçalho
                document.getElementById('dashboardTitle').innerText = `${userData.login}'s GitHub Stats`;
                document.getElementById('userInitial').innerText = userData.login.charAt(0).toUpperCase();
                document.getElementById('publicRepos').innerText = userData.public_repos;
                document.getElementById('followers').innerText = userData.followers;
                document.getElementById('following').innerText = userData.following;
                document.getElementById('publicGists').innerText = userData.public_gists;

                // 2. Buscar dados dos repositórios para estrelas e linguagens
                const reposResponse = await fetch(`https://api.github.com/users/${username}/repos?per_page=100`);
                const reposData = await reposResponse.json();

                let totalStars = 0;
                let languagesMap = {};

                reposData.forEach(repo => {
                    totalStars += repo.stargazers_count;
                    if (repo.language) {
                        languagesMap[repo.language] = (languagesMap[repo.language] || 0) + 1;
                    }
                });

                document.getElementById('totalStars').innerText = totalStars;

                // Renderizar os gráficos com os dados processados
                renderLanguagesChart(languagesMap);
                renderFrequencyChart();

            } catch (error) {
                alert(error.message);
            }
        }

        function renderLanguagesChart(languagesMap) {
            const ctx = document.getElementById('languagesChart').getContext('2d');
            
            if (langChartInstance) langChartInstance.destroy();

            const labels = Object.keys(languagesMap);
            const data = Object.values(languagesMap);

            // Cores baseadas no estilo neon da imagem
            const neonColors = ['#ff55b8', '#3498db', '#f1c40f', '#2ecc71', '#9b59b6', '#e67e22'];

            langChartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: data,
                        backgroundColor: neonColors.slice(0, labels.length),
                        borderColor: '#160718',
                        borderWidth: 2
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'bottom',
                            labels: { color: '#ffffff', font: { size: 10 } }
                        }
                    }
                }
            });
        }

        function renderFrequencyChart() {
            const ctx = document.getElementById('frequencyChart').getContext('2d');
            
            if (freqChartInstance) freqChartInstance.destroy();

            // Mock de dados para simular a linha ondulada rosa da imagem original
            freqChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'],
                    datasets: [{
                        label: 'Activity',
                        data: [110, 260, 130, 90, 80, 140, 110, 160, 90, 150],
                        borderColor: '#ff55b8',
                        backgroundColor: 'rgba(255, 85, 184, 0.1)',
                        tension: 0.4,
                        fill: true,
                        pointBackgroundColor: '#ff55b8'
                    }]
                },
                options: {
              responsive: true,
                    plugins: { legend: { display: false } },
                    scales: {
                        x: { grid: { display: false }, ticks: { color: '#fff' } },
                        y: { grid: { color: 'rgba(255, 255, 255, 0.1)' }, ticks: { color: '#fff' } }
                    }
                }
            });
        }  
        // Executar automaticamente na primeira carga
        window.onload = fetchGitHubStats;
    </script>
</body>
</html>
