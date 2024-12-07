Nesse comite fiz a parte do banco de dados com essa estrutura:
Para a estrutura do banco de dados, você pode começar com as seguintes tabelas principais:

### 1. **Usuários**  
Armazena informações sobre os usuários que se cadastram no sistema.  
- **id_usuario** (PK, Auto Increment)
- **nome** (VARCHAR)
- **email** (VARCHAR, único)
- **senha** (VARCHAR, criptografada)
- **data_cadastro** (DATE)
- **status_conta** (ENUM: "ativo", "inativo")

### 2. **Vídeos**  
Armazena informações sobre os vídeos disponíveis no site.  
- **id_video** (PK, Auto Increment)
- **titulo** (VARCHAR)
- **descricao** (TEXT)
- **url_video** (VARCHAR) 
- **data_upload** (DATE)
- **id_playlist** (FK, referência para Playlists)

### 3. **Playlists**  
Armazena informações sobre as playlists criadas por você.  
- **id_playlist** (PK, Auto Increment)
- **nome_playlist** (VARCHAR)
- **descricao** (TEXT)
- **data_criacao** (DATE)

### 4. **Assinaturas**  
Registra os pagamentos e a relação do usuário com os serviços.  
- **id_assinatura** (PK, Auto Increment)
- **id_usuario** (FK, referência para Usuários)
- **data_inicio** (DATE)
- **data_fim** (DATE)
- **status_assinatura** (ENUM: "ativa", "expirada", "cancelada")

### 5. **Pagamentos**  
Armazena as informações das transações realizadas pelos usuários.  
- **id_pagamento** (PK, Auto Increment)
- **id_usuario** (FK, referência para Usuários)
- **valor** (DECIMAL)
- **data_pagamento** (DATE)
- **metodo_pagamento** (ENUM: "E-mola", "M-pesa", "Millenium")

tendo assim esse script do mysql:

-- Criar o banco de dados
CREATE DATABASE videos_app;

-- Usar o banco de dados
USE videos_app;

-- Tabela de Usuários
CREATE TABLE usuarios (
    id_usuario INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    senha VARCHAR(255) NOT NULL, -- Criptografada
    data_cadastro DATE NOT NULL DEFAULT CURRENT_DATE,
    status_conta ENUM('ativo', 'inativo') NOT NULL DEFAULT 'ativo'
);

-- Tabela de Playlists
CREATE TABLE playlists (
    id_playlist INT AUTO_INCREMENT PRIMARY KEY,
    nome_playlist VARCHAR(100) NOT NULL,
    descricao TEXT,
    data_criacao DATE NOT NULL DEFAULT CURRENT_DATE
);

-- Tabela de Vídeos
CREATE TABLE videos (
    id_video INT AUTO_INCREMENT PRIMARY KEY,
    titulo VARCHAR(150) NOT NULL,
    descricao TEXT,
    url_video VARCHAR(255) NOT NULL,
    data_upload DATE NOT NULL DEFAULT CURRENT_DATE,
    id_playlist INT,
    FOREIGN KEY (id_playlist) REFERENCES playlists(id_playlist) ON DELETE SET NULL
);

-- Tabela de Assinaturas
CREATE TABLE assinaturas (
    id_assinatura INT AUTO_INCREMENT PRIMARY KEY,
    id_usuario INT NOT NULL,
    data_inicio DATE NOT NULL DEFAULT CURRENT_DATE,
    data_fim DATE NOT NULL,
    status_assinatura ENUM('ativa', 'expirada', 'cancelada') NOT NULL DEFAULT 'ativa',
    FOREIGN KEY (id_usuario) REFERENCES usuarios(id_usuario) ON DELETE CASCADE
);

-- Tabela de Pagamentos
CREATE TABLE pagamentos (
    id_pagamento INT AUTO_INCREMENT PRIMARY KEY,
    id_usuario INT NOT NULL,
    valor DECIMAL(10, 2) NOT NULL,
    data_pagamento DATE NOT NULL DEFAULT CURRENT_DATE,
    metodo_pagamento ENUM('cartão', 'PIX', 'boleto') NOT NULL,
    FOREIGN KEY (id_usuario) REFERENCES usuarios(id_usuario) ON DELETE CASCADE
);

-- Tabela para controlar o acesso aos vídeos
CREATE TABLE acesso_videos (
    id_acesso INT AUTO_INCREMENT PRIMARY KEY,
    id_usuario INT NOT NULL,
    id_video INT NOT NULL,
    data_acesso DATE NOT NULL DEFAULT CURRENT_DATE,
    FOREIGN KEY (id_usuario) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (id_video) REFERENCES videos(id_video) ON DELETE CASCADE
);
Como usar essa tabela
Sempre que um pagamento for confirmado, adicione registros à tabela acesso_videos, associando o id_usuario aos vídeos comprados.
Antes de permitir o acesso a um vídeo, consulte esta tabela para verificar se o usuário tem permissão.
Validação no Backend
No momento de autenticação para assistir a um vídeo:
Solução 1: Verifique na tabela assinaturas se o usuário tem uma assinatura válida.
Solução 2: Consulte a tabela acesso_videos para verificar se há permissão explícita para o vídeo solicitado.
