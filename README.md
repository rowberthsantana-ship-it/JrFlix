<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JrFlix - Streaming Professional</title>
    <style>
        :root {
            --primary-red: #E50914;
            --dark-bg: #141414;
            --card-width: 200px;
        }

        body {
            background-color: var(--dark-bg);
            color: white;
            font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
            margin: 0;
            padding: 0;
            overflow-x: hidden;
        }

        /* Header e Busca */
        header {
            padding: 15px 4%;
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: linear-gradient(to bottom, rgba(0,0,0,0.9), transparent);
            position: fixed;
            width: 100%;
            top: 0;
            z-index: 1000;
            box-sizing: border-box;
            transition: background 0.5s;
        }

        .logo {
            color: var(--primary-red);
            font-size: 2.2rem;
            font-weight: bold;
            letter-spacing: -1px;
        }

        .search-box input {
            background: rgba(0,0,0,0.5);
            border: 1px solid #fff;
            color: white;
            padding: 8px 12px;
            border-radius: 4px;
            outline: none;
        }

        /* Banner Principal */
        .banner {
            height: 85vh;
            background-size: cover;
            background-position: center;
            display: flex;
            align-items: center;
            padding: 0 4%;
            margin-bottom: 20px;
        }

        .banner-content { width: 45%; text-shadow: 2px 2px 4px rgba(0,0,0,0.5); }
        .banner-title { font-size: 3.5rem; margin-bottom: 15px; }
        .banner-desc { font-size: 1.2rem; line-height: 1.4; margin-bottom: 20px; }

        /* Fileiras de Filmes */
        .movie-row { margin: 30px 0; padding-left: 4%; }
        .movie-list {
            display: flex;
            overflow-x: auto;
            gap: 10px;
            padding: 20px 0;
            scrollbar-width: none;
        }
        .movie-list::-webkit-scrollbar { display: none; }

        .movie-card {
            min-width: var(--card-width);
            transition: transform 0.4s ease;
            cursor: pointer;
        }
        .movie-card:hover { transform: scale(1.15); z-index: 10; }
        .movie-card img { width: 100%; border-radius: 6px; }

        /* Login Screen */
        #loginScreen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: url('https://assets.nflxext.com/ffe/siteui/vlv3/f841d4c7-10e1-40af-bca1-07e3f8eb14af/614138e6-764f-409d-92d5-98544d673177/BR-pt-20220502-popsignuptwoweeks-perspective_alpha_website_medium.jpg');
            background-size: cover; z-index: 5000; display: flex; justify-content: center; align-items: center;
        }
        .login-card {
            background: rgba(0,0,0,0.75); padding: 60px; border-radius: 8px; width: 350px; text-align: center;
        }

        /* Player */
        #videoPlayer {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: black; z-index: 9000;
        }

        .btn-action {
            background: var(--primary-red); color: white; border: none;
            padding: 10px 20px; border-radius: 4px; cursor: pointer; font-weight: bold;
        }
    </style>
</head>
<body>

    <div id="loginScreen">
        <div class="login-card">
            <h1 style="color: var(--primary-red); font-size: 2.5rem;">JrFlix</h1>
            <input type="text" id="userInput" placeholder="Usuário (admin)" style="width:100%; padding:12px; margin:10px 0; background:#333; color:white; border:none; border-radius:4px;">
            <input type="password" id="passInput" placeholder="Senha (123)" style="width:100%; padding:12px; margin:10px 0; background:#333; color:white; border:none; border-radius:4px;">
            <button class="btn-action" style="width:100%; margin-top:20px;" onclick="handleLogin()">Entrar</button>
        </div>
    </div>

    <header id="mainHeader">
        <div class="logo">JrFlix</div>
        <div class="search-box">
            <input type="text" id="searchInput" placeholder="Pesquisar filmes...">
            <button class="btn-action" onclick="handleSearch()">🔍</button>
        </div>
    </header>

    <section class="banner" id="mainBanner">
        <div class="banner-content">
            <h1 class="banner-title" id="bannerTitle">Destaque</h1>
            <p class="banner-desc" id="bannerDesc">Carregando as melhores histórias para você...</p>
            <button class="btn-action" onclick="openPlayer()">▶ Assistir</button>
        </div>
    </section>

    <main id="rows-container"></main>

    <div id="videoPlayer">
        <button onclick="closePlayer()" style="position:absolute; top:20px; right:20px; background:white; border:none; padding:10px; border-radius:50%; cursor:pointer; font-weight:bold; z-index:9001;">X</button>
        <video id="mainVideo" controls style="width:100%; height:100%;">
            <source src="https://www.w3schools.com/html/mov_bbb.mp4" type="video/mp4">
        </video>
    </div>

    <script>
        // CONFIGURAÇÕES DA API
        const API_KEY = 'SUA_CHAVE_AQUI'; // <--- INSIRA SUA CHAVE DO TMDB AQUI
        const BASE_URL = 'https://api.themoviedb.org/3';
        const IMG_URL = 'https://image.tmdb.org/t/p/original';
        const GENRES = [
            { id: 28, name: "Ação" },
            { id: 35, name: "Comédia" },
            { id: 27, name: "Terror" },
            { id: 878, name: "Ficção Científica" }
        ];

        // LOGIN
        function handleLogin() {
            const u = document.getElementById('userInput').value;
            const p = document.getElementById('passInput').value;
            if(u === "admin" && p === "123") {
                localStorage.setItem('jrflix_access', 'true');
                document.getElementById('loginScreen').style.display = 'none';
                loadApp();
            } else { alert("Acesso negado!"); }
        }

        // CARREGAR APP
        async function loadApp() {
            if(!API_KEY || API_KEY === 'SUA_CHAVE_AQUI') {
                alert("Por favor, insira sua API KEY do TMDB no código.");
                return;
            }

            // Banner
            const trending = await fetch(`${BASE_URL}/trending/all/week?api_key=${API_KEY}&language=pt-BR`).then(r => r.json());
            const feat = trending.results[0];
            document.getElementById('mainBanner').style.backgroundImage = `linear-gradient(to right, #141414 20%, transparent), url('${IMG_URL + feat.backdrop_path}')`;
            document.getElementById('bannerTitle').innerText = feat.title || feat.name;
            document.getElementById('bannerDesc').innerText = feat.overview.slice(0, 150) + "...";

            // Fileiras
            const container = document.getElementById('rows-container');
            container.innerHTML = "";
            for (const genre of GENRES) {
                const res = await fetch(`${BASE_URL}/discover/movie?api_key=${API_KEY}&with_genres=${genre.id}&language=pt-BR`).then(r => r.json());
                createRow(genre.name, res.results);
            }
        }

        function createRow(title, movies) {
            const row = document.createElement('div');
            row.className = 'movie-row';
            row.innerHTML = `<h2>${title}</h2><div class="movie-list"></div>`;
            const list = row.querySelector('.movie-list');
            
            movies.forEach(m => {
                if(m.poster_path) {
                    const card = document.createElement('div');
                    card.className = 'movie-card';
                    card.innerHTML = `<img src="${'https://image.tmdb.org/t/p/w500' + m.poster_path}">`;
                    card.onclick = () => openPlayer();
                    list.appendChild(card);
                }
            });
            document.getElementById('rows-container').appendChild(row);
        }

        // BUSCA
        async function handleSearch() {
            const q = document.getElementById('searchInput').value;
            if(q.length < 2) return loadApp();
            
            const res = await fetch(`${BASE_URL}/search/movie?api_key=${API_KEY}&query=${q}&language=pt-BR`).then(r => r.json());
            document.getElementById('mainBanner').style.display = 'none';
            const container = document.getElementById('rows-container');
            container.innerHTML = `<h2 style="padding-left:4%; margin-top:100px;">Resultados para: ${q}</h2>`;
            createRow("", res.results);
        }

        // PLAYER
        function openPlayer() {
            document.getElementById('videoPlayer').style.display = 'block';
            document.getElementById('mainVideo').play();
        }
        function closePlayer() {
            document.getElementById('videoPlayer').style.display = 'none';
            document.getElementById('mainVideo').pause();
        }

        // AUTO-LOGIN CHECK
        window.onload = () => {
            if(localStorage.getItem('jrflix_access') === 'true') {
                document.getElementById('loginScreen').style.display = 'none';
                loadApp();
            }
        }
    </script>
</body>
</html>
