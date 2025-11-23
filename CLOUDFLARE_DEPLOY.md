# Guia de Implantação: Cloudflare Pages

Este guia explica como transformar os arquivos do **AutoPost Pro** em um site real online usando o **Cloudflare Pages**.

Como o projeto utiliza React e TypeScript, precisamos usar uma ferramenta de construção (Build Tool) chamada **Vite** antes de enviar para a Cloudflare.

---

## Passo 1: Preparar o Projeto Localmente

Você precisará do [Node.js](https://nodejs.org/) instalado no seu computador.

1.  Abra seu terminal (Prompt de Comando ou PowerShell).
2.  Crie uma nova pasta para o projeto e inicialize com Vite:
    ```bash
    npm create vite@latest autopost-pro -- --template react-ts
    ```
3.  Entre na pasta:
    ```bash
    cd autopost-pro
    ```
4.  Instale as dependências necessárias do projeto:
    ```bash
    npm install lucide-react @google/genai
    ```
    *Para o estilo (Tailwind CSS), siga o guia oficial ou use CDN no index.html como feito no protótipo.*

---

## Passo 2: Transferir os Arquivos

Agora, copie os arquivos que geramos aqui para dentro da pasta do seu projeto local:

1.  Substitua o conteúdo de `src/App.tsx` pelo código do nosso App.
2.  Crie/Copie os arquivos da pasta `components` para `src/components/`.
3.  Crie/Copie os arquivos da pasta `services` para `src/services/`.
4.  Copie o arquivo `types.ts` para `src/types.ts`.

### ⚠️ Ajuste Importante para Produção

Ferramentas modernas como o Vite usam uma forma diferente de ler chaves de API.

1.  Abra o arquivo `src/services/geminiService.ts`.
2.  Encontre a linha:
    ```typescript
    const apiKey = process.env.API_KEY;
    ```
3.  Mude para:
    ```typescript
    // No Vite, usamos import.meta.env
    const apiKey = import.meta.env.VITE_API_KEY;
    ```

---

## Passo 3: Publicar na Cloudflare Pages

Existem duas formas principais: via **GitHub** (Recomendado) ou **Upload Direto**.

### Opção A: Via GitHub (Automático)
Ideal se você sabe usar Git. A cada alteração no código, o site atualiza sozinho.

1.  Suba seu código para um repositório no GitHub.
2.  Acesse o painel da [Cloudflare Dashboard](https://dash.cloudflare.com/).
3.  Vá em **Workers & Pages** -> **Create Application** -> **Pages** -> **Connect to Git**.
4.  Selecione seu repositório `autopost-pro`.
5.  Nas configurações de Build:
    *   **Framework preset:** Selecione `Vite`.
    *   **Build command:** `npm run build`
    *   **Build output directory:** `dist`
6.  **Configurar Chave de API (Segurança):**
    *   Ainda na tela de configuração, procure por **Environment Variables**.
    *   Adicione uma nova variável:
        *   **Variable name:** `VITE_API_KEY` (O prefixo VITE_ é obrigatório)
        *   **Value:** Cole sua chave da API do Google Gemini aqui.
7.  Clique em **Save and Deploy**.

### Opção B: Upload Direto (Manual)
Se você não usa Git.

1.  No seu terminal local, rode o comando para gerar o site:
    ```bash
    npm run build
    ```
    *Isso criará uma pasta chamada `dist` no seu projeto.*
2.  Acesse a [Cloudflare Dashboard](https://dash.cloudflare.com/).
3.  Vá em **Workers & Pages** -> **Create Application** -> **Pages** -> **Upload Assets**.
4.  Arraste a pasta `dist` que foi criada no seu computador para a Cloudflare.
5.  **Atenção:** Neste método, para que a chave de API funcione, você precisará criar um arquivo `.env` na raiz do projeto local **antes** de rodar o `npm run build`:
    ```env
    VITE_API_KEY=sua_chave_aqui_sem_aspas
    ```

---

## Resolução de Problemas Comuns

**1. O site carrega, mas dá erro ao gerar texto com IA.**
*   Verifique se você configurou a Variável de Ambiente `VITE_API_KEY` no painel da Cloudflare.
*   Verifique se alterou o código no `geminiService.ts` para usar `import.meta.env.VITE_API_KEY`.

**2. Erro de permissão no Facebook.**
*   Lembre-se que o domínio do seu site mudou (agora é algo como `autopost-pro.pages.dev`).
*   Vá no **Facebook Developers**, nas configurações do seu App, e adicione este novo domínio em "App Domains".

**3. Estilos quebrados.**
*   Se estiver usando Tailwind via CDN (como no código original), certifique-se de que o `index.html` na pasta `dist` manteve a tag `<script src="https://cdn.tailwindcss.com"></script>`. Se instalou o Tailwind via npm, certifique-se de ter importado o CSS no `main.tsx`.
