<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0">
    <meta name="theme-color" content="#E50914">
    <meta name="description" content="JrFlix - Assista filmes e séries de Rowberth Santana.">
    <title>JrFlix - Original Santana</title>
    
    <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiSnJGbGl4IC0gU3RyZWFtIFNlbSBMaW1pdGVzIiwic2hvcnRfbmFtZSI6IkpyRmxpeCIsInN0YXJ0X3VybCI6Ii4vIiwiaWNvbnMiOlt7InNyYyI6Imh0dHBzOi8vY2RuLWljb25zLXBuZy5mbGF0aWNvbi5jb20vNTEyLzM2NTkvMzY1OTg5OC5wbmciLCJzaXplcyI6IjUxMng1MTIiLCJ0eXBlIjoiaW1hZ2UvcG5nIn1dfQ==">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <link rel="icon" type="image/png" href="https://cdn-icons-png.flaticon.com/512/3659/3659898.png">

    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;900&display=swap');
        * { font-family: 'Inter', sans-serif; box-sizing: border-box; }
        body { background-color: #141414; color: white; overflow-x: hidden; }
        
        /* Estilos Netflix/JrFlix */
        .glass-effect { background: rgba(20, 20, 20, 0.95); backdrop-filter: blur(20px); }
        .hero-gradient { background: linear-gradient(to top, #141414 0%, transparent 60%); }
        .card-hover { transition: transform 0.3s ease; cursor: pointer; position: relative; border-radius: 4px; overflow: hidden; background: #181818; }
        .card-hover:hover { transform: scale(1.08); z-index: 50; }
        .movie-info { position: absolute; bottom: 0; left: 0; right: 0; padding: 15px; background: linear-gradient(to top, black, transparent); opacity: 0; transition: 0.3s; }
        .card-hover:hover .movie-info { opacity: 1; }
        
        /* Selo Original Santana */
        .original-badge { position: absolute; top: 10px; left: 0; background: #E50914; color: white; font-size: 10px; font-weight: 900; padding: 4px 8px; text-transform: uppercase; z-index: 20; border-radius: 0 4px 4px 0; }
        
        /* Barra de Gêneros */
        .genre-btn { background: rgba(255,255,255,0.1); border: 1px solid rgba(255,255,255,0.2); padding: 8px 20px; border-radius: 20px; white-space: nowrap; transition: 0.3s; font-size: 14px; }
        .genre-btn.active { background: #E50914; border-color: #E50914; }
        .hide-scroll::-webkit-scrollbar { display: none; }

        /* Modais e UI */
        .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 10000; display: none; align-items: center; justify-content: center; }
        .modal-overlay.active { display: flex; }
        .modal-content { background: #181818; padding: 30px; border-radius: 12px; width: 100%; max-width: 500px; position: relative; }
        .input-field { width: 100%; background: #333; border: 1px solid #444; color: white; padding: 12px; border-radius: 6px; margin-bottom: 15px; outline: none; }
        .btn-primary { background: #E50914; width: 100%; padding: 12px; font-weight: bold; border-radius: 6px; }
        
        /* Video Player */
        .video-modal { position: fixed; inset: 0; background: black; z-index: 20000; display: none; }
        .video-modal.active { display: flex; }
    </style>
</head>
<body>

    <nav class="fixed w-full z-50 glass-effect p-4 flex items-center justify-between">
        <div class="flex items-center gap-8">
            <span class="text-2xl font-black text-[#E50914]">JRFLIX</span>
            <div class="hidden md:flex gap-6 text-sm text-gray-300">
                <a href="#" onclick="location.reload()">Início</a>
                <a href="#lancamentos">Minhas Obras</a>
            </div>
        </div>
        <div class="flex items-center gap-4">
            <div id="authSection">
                <button onclick="openAuth()" class="text-sm font-bold mr-4">Entrar</button>
                <button onclick="openPayment()" class="bg-[#E50914] px-4 py-1.5 rounded text-sm font-bold">Assinar</button>
            </div>
            <div id="userSection" class="hidden flex items-center gap-3">
                <div class="relative group">
                    <div id="userAvatar" class="w-8 h-8 rounded bg-[#E50914] flex items-center justify-center font-bold cursor-pointer">R</div>
                    <div class="absolute right-0 mt-2 w-48 bg-black border border-gray-800 rounded opacity-0 invisible group-hover:opacity-100 group-hover:visible transition-all">
                        <a href="#" onclick="openAuthorPanel()" class="block p-3 text-[#E50914] font-bold hover:bg-gray-900 border-b border-gray-800">Painel do Autor</a>
                        <a href="#" onclick="logout()" class="block p-3 text-sm hover:bg-gray-900">Sair</a>
                    </div>
                </div>
            </div>
        </div>
    </nav>

    <div class="relative h-[80vh] flex items-end pb-20 px-8">
        <img src="https://images.unsplash.com/photo-1626814026160-2237a95fc5a0?w=1920" class="absolute inset-0 w-full h-full object-cover -z-10" id="heroImg">
        <div class="absolute inset-0 hero-gradient -z-10"></div>
        <div class="max-w-2xl">
            <h1 class="text-6xl font-black mb-4" id="heroTitle">Vingadores:<br>Doutor Destino</h1>
            <p class="text-lg text-gray-300 mb-6" id="heroDesc">O aguardado desfecho da saga multiverso sob o olhar de Rowberth Santana.</p>
            <button onclick="playHero()" class="bg-white text-black px-8 py-3 rounded font-bold mr-4">Assistir</button>
        </div>
    </div>

    <div class="px-8 mt-8 overflow-x-auto hide-scroll flex gap-3">
        <button onclick="filterGenre('todos')" class="genre-btn active">Todos</button>
        <button onclick="filterGenre('ficcao')" class="genre-btn">Ficção Científica</button>
        <button onclick="filterGenre('terror')" class="genre-btn">Horror</button>
        <button onclick="filterGenre('drama')" class="genre-btn">Drama</button>
    </div>

    <main class="p-8">
        <section id="lancamentos">
            <h2 class="text-xl font-bold mb-6" id="sectionTitle">Lançamentos</h2>
            <div id="grid" class="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-5 gap-6"></div>
        </section>
    </main>

    <div id="authorModal" class="modal-overlay">
        <div class="modal-content">
            <h2 class="text-xl font-bold mb-4">Nova Obra Original</h2>
            <form onsubmit="handleUpload(event)">
                <input id="upTitle" class="input-field" placeholder="Título" required>
                <select id="upCat" class="input-field">
                    <option value="ficcao">Ficção Científica</option>
                    <option value="terror">Horror</option>
                    <option value="drama">Drama</option>
                </select>
                <input id="upImg" class="input-field" placeholder="Link da Capa (URL)" required>
                <input id="upVid" class="input-field" placeholder="Link do Vídeo (URL)" required>
                <button type="submit" class="btn-primary">Publicar na JrFlix</button>
                <button type="button" onclick="closeAuthorPanel()" class="w-full mt-2 text-gray-400">Cancelar</button>
            </form>
        </div>
    </div>

    <div id="videoModal" class="video-modal">
        <video id="player" class="w-full h-full" controls></video>
        <button onclick="closeVideo()" class="absolute top-8 right-8 text-3xl"><i class="fas fa-times"></i></button>
    </div>

    <script>
        const contentData = [
            { id: 1, title: "Doutor Destino", category: "ficcao", original: true, image: "https://images.unsplash.com/photo-1635805737707-575885ab0820?w=400", video: "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4" },
            { id: 2, title: "O Homem Macaco", category: "terror", original: true, image: "https://images.unsplash.com/photo-1594909122849-11daa2a0cf2b?w=400", video: "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4" }
        ];

        let currentUser = JSON.parse(localStorage.getItem('user'));

        function init() {
            const uploads = JSON.parse(localStorage.getItem('uploads') || '[]');
            const all = [...uploads, ...contentData];
            render(all);
            if(currentUser) updateUI();
        }

        function render(data) {
            const grid = document.getElementById('grid');
            grid.innerHTML = data.map(item => `
                <div class="card-hover group" onclick="playVideo('${item.video}')">
                    ${item.original ? `<div class="original-badge">Original Santana</div>` : ''}
                    <img src="${item.image}" class="w-full aspect-[2/3] object-cover">
                    <div class="movie-info">
                        <p class="font-bold text-sm">${item.title}</p>
                        <p class="text-xs text-red-500 uppercase">${item.category}</p>
                    </div>
                </div>
            `).join('');
        }

        function filterGenre(genre) {
            document.querySelectorAll('.genre-btn').forEach(b => b.classList.remove('active'));
            event.target.classList.add('active');
            const uploads = JSON.parse(localStorage.getItem('uploads') || '[]');
            const all = [...uploads, ...contentData];
            const filtered = genre === 'todos' ? all : all.filter(i => i.category === genre);
            render(filtered);
        }

        function handleUpload(e) {
            e.preventDefault();
            const newItem = {
                id: Date.now(),
                title: document.getElementById('upTitle').value,
                category: document.getElementById('upCat').value,
                image: document.getElementById('upImg').value,
                video: document.getElementById('upVid').value,
                original: true
            };
            const uploads = JSON.parse(localStorage.getItem('uploads') || '[]');
            uploads.unshift(newItem);
            localStorage.setItem('uploads', JSON.stringify(uploads));
            location.reload();
        }

        function playVideo(url) {
            if(!currentUser) return openAuth();
            const m = document.getElementById('videoModal');
            const v = document.getElementById('player');
            v.src = url;
            m.classList.add('active');
            v.play();
        }

        function closeVideo() {
            const m = document.getElementById('videoModal');
            const v = document.getElementById('player');
            v.pause();
            m.classList.remove('active');
        }

        function openAuth() { alert("Simulação de Login: Use qualquer email."); login(); }
        function login() { 
            currentUser = { name: "Rowberth" }; 
            localStorage.setItem('user', JSON.stringify(currentUser));
            updateUI();
        }
        function updateUI() {
            document.getElementById('authSection').classList.add('hidden');
            document.getElementById('userSection').classList.remove('hidden');
        }
        function logout() { localStorage.clear(); location.reload(); }
        function openAuthorPanel() { document.getElementById('authorModal').classList.add('active'); }
        function closeAuthorPanel() { document.getElementById('authorModal').classList.remove('active'); }

        init();
    </script>
</body>
</html>
